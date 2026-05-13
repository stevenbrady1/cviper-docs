# CViper Observability

**Current state (since 2026-05-13)**: Azure Log Analytics + Azure Monitor only. The Grafana/Loki/Prometheus stack is scaled to zero (CV-272). All resources, configs, dashboards, and code paths are preserved for future re-enablement.

> If you are about to re-enable the stack: read CV-273 first. Loki has been crashlooping since 2026-04-02 (likely AzureFile mount permissions). Do not flip `observabilityEnabled=true` without addressing this.

---

## What changed

| Concern | Before | After |
|---|---|---|
| Application logs | stdout → Container Apps Environment → both Log Analytics AND `cviper-loki` (push API) | stdout → Container Apps Environment → Log Analytics only |
| Metrics dashboards | Grafana panels backed by Prometheus | Azure Monitor in the Azure portal |
| Alerting | None active on Loki/Prom; Container Apps health visible only via `az containerapp show` | Same gap — see Future work below |
| Cost | Grafana running 24/7 (one 0.25 vCPU / 0.5 GiB replica) | Grafana scaled to 0/0, saving roughly £5–10/month |

The Container Apps Environment was simultaneously sending logs to Log Analytics throughout the entire Loki window, so disabling Loki produced **zero data loss**. The Loki path was redundant.

---

## Where to look now

### Logs — Log Analytics

Workspace `cviper-logs` in resource group `cviper-rg`.

**Query the Azure portal**: Log Analytics workspaces → `cviper-logs` → Logs.

**Query from CLI** (requires the `log-analytics` extension):

```bash
az monitor log-analytics query \
  --workspace 4b71fba4-46b5-4551-956e-a1a391fce6d9 \
  --analytics-query "ContainerAppConsoleLogs_CL | where ContainerAppName_s == 'cviper-backend' | take 20" \
  --timespan PT1H \
  -o table
```

The backend emits structured JSON (one record per line, `backend/monitoring.py:86-114`), so you can pivot on any field.

### Useful KQL snippets

**Errors in the last hour, grouped by path:**

```kql
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(1h)
| where ContainerAppName_s == 'cviper-backend'
| extend payload = parse_json(Log_s)
| where payload.level in ('ERROR', 'WARNING')
| summarize count() by tostring(payload.path), tostring(payload.level)
| order by count_ desc
```

**Slow requests (>2s) in the last 4 hours:**

```kql
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(4h)
| where ContainerAppName_s == 'cviper-backend'
| extend payload = parse_json(Log_s)
| where toreal(payload.duration_ms) > 2000
| project TimeGenerated, payload.method, payload.path, payload.status_code, payload.duration_ms, payload.request_id
| order by toreal(payload.duration_ms) desc
| take 50
```

**Trace a single request across services by correlation ID:**

```kql
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(2h)
| extend payload = parse_json(Log_s)
| where payload.request_id == 'abc123def4567890'
| project TimeGenerated, ContainerAppName_s, payload.level, payload.message, payload.path
| order by TimeGenerated asc
```

**AI gateway call rate by provider (last 24h):**

```kql
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(24h)
| where ContainerAppName_s == 'cviper-backend'
| extend payload = parse_json(Log_s)
| where payload.component == 'ai_gateway' or payload.event_type startswith 'ai_'
| summarize count() by tostring(payload.provider), bin(TimeGenerated, 1h)
| render timechart
```

**5xx rate (replacement for the old Grafana SLO panel):**

```kql
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(1h)
| where ContainerAppName_s == 'cviper-backend'
| extend payload = parse_json(Log_s)
| where isnotempty(payload.status_code)
| summarize total = count(), errors = countif(toint(payload.status_code) >= 500) by bin(TimeGenerated, 5m)
| extend error_rate_pct = round(100.0 * errors / total, 2)
| project TimeGenerated, total, errors, error_rate_pct
| order by TimeGenerated desc
```

### Metrics — Azure Monitor

Container Apps natively exposes these in the Azure portal at zero extra cost (Container App → Monitoring → Metrics):

| Metric | What the old Grafana panel showed |
|---|---|
| `CpuUsageNanoCores` | CPU utilisation per replica |
| `MemoryWorkingSetBytes` | Memory pressure |
| `Requests` | Request count (split on `statusCodeCategory` dimension for 2xx/4xx/5xx) |
| `ReplicaCount` | Current running replicas (scaling visibility) |
| `RestartCount` | Container restart frequency (crashloop detector) |

For p95/p99 latency, the previous Prometheus histogram was synthesised from the `duration_ms` field — query that directly in Log Analytics using `percentile()` on the KQL above.

---

## Re-enabling the stack

**Prerequisites (do these first):**

1. **Resolve [CV-273](https://github.com/stevenbrady1/CViper/issues/539)** — Loki has been crashlooping since 2026-04-02 (exit code 1 at boot, 965+ container restarts). Most likely cause is AzureFile mount permissions (Loki UID 10001 needs explicit `uid=10001,gid=10001` mount options on the SMB share). Without fixing this, re-enabling just gets you back to a silently-broken Loki.
2. **Add Container Apps health alert** in Azure Monitor: `ContainerAppName_s in ('cviper-loki','cviper-prometheus','cviper-grafana') and runningState != 'Running'`. Without it, the next regression will go unnoticed for another six weeks.
3. **Add a synthetic log test** — backend emits a sentinel line, a test queries it back from Loki within 60s, fails CI if missing.

**To enable:**

1. Flip the parameter in both Bicep files:
   ```bicep
   // azure/container-apps.bicep (line 36) and azure/container-apps-staging.bicep
   param observabilityEnabled bool = true
   ```
2. Trigger the `Deploy to Azure` workflow (workflow_dispatch → environment=`production`). The deploy will:
   - Add `LOKI_URL` and `GRAFANA_URL` back to the `cviper-backend` env (new revision)
   - Set `OBSERVABILITY_ENABLED=true` so `backend/monitoring.py:attach_loki_handler()` re-engages
   - Scale `cviper-loki`, `cviper-prometheus`, `cviper-grafana` back to `min=1, max=1`
3. Repeat for staging by deploying `container-apps-staging.bicep` with the same flip.
4. Verify:
   ```bash
   # Loki shipping log on the first new backend revision
   az containerapp logs show -n cviper-backend -g cviper-rg --tail 100 | grep -i 'Loki log shipping enabled'

   # All three observability apps Running
   for APP in cviper-loki cviper-prometheus cviper-grafana; do
     az containerapp show -n "$APP" -g cviper-rg --query "properties.runningStatus" -o tsv
   done
   ```

**Alternative — runtime scale without redeploy** (skip Bicep, useful for short-lived re-enables):

Use the existing [`.github/workflows/manage-services.yml`](../.github/workflows/manage-services.yml) — Actions → `Manage Services` → action=`start`, service=`all`. This bypasses Bicep entirely and will be reverted by the next infrastructure deploy. Only use this if you don't want the change to persist.

---

## Disabling (if it ever needs doing again)

Already done as of 2026-05-13. The PR for reference: see commits on branch `infra/CV-272-disable-observability-stack`.

If a future incident requires re-disabling:

1. Flip `observabilityEnabled` back to `false` in both Bicep files and deploy.
2. Or trigger `manage-services.yml` → action=`stop`, service=`grafana` (then `loki`, then `prometheus`) for a runtime-only stop that the next Bicep deploy will re-enforce.

---

## What was preserved (do NOT delete in cleanup sweeps)

The following files exist solely so the stack can be re-enabled without rebuilding from scratch:

- `azure/container-apps.bicep` — Loki/Prometheus/Grafana resource definitions remain. Only the `scale` blocks are parameterised.
- `azure/container-apps-staging.bicep` — staging backend gating only (staging has no own observability resources).
- `monitoring/loki-config.yml`, `monitoring/prometheus*.yml`, `monitoring/grafana/**` — all configs, dashboards, alerts, provisioning.
- `docker-compose.yml`, `docker-compose.dev.yml`, `docker-compose.e2e.yml` — local-dev stack is unchanged; `docker compose up` still spins up Loki/Prometheus/Grafana for development.
- `backend/monitoring.py` LokiHandler, `backend/metrics.py` prometheus_client integration, `backend/routes/monitoring.py` Grafana URL endpoint — all code paths intact and conditional, no behavioural change when env vars are empty.
- `frontend/src/components/MonitoringPanel.jsx` — already renders a "not configured" fallback when GRAFANA_URL is empty (no change).
- `workers/status-page/src/worker.js` — `GRAFANA_DASHBOARDS` array and `renderGrafanaSection()` are dormant when the worker's `GRAFANA_BASE` env var is unset. To fully hide the Grafana section on the public status page, unset `GRAFANA_BASE` in the Cloudflare Worker's environment.
- `tests/test_deploy_config.sh` — Grafana hostname check is preserved, gated by `OBSERVABILITY_ENABLED`.

---

## Alerting — `azure/monitor-alerts.bicep`

Three Azure Monitor alert rules are defined in [`azure/monitor-alerts.bicep`](../azure/monitor-alerts.bicep) and route to an action group (`cviper-alerts`) with one email receiver. **The file is not deployed automatically** — it sits separate from `azure/container-apps.bicep` so tuning the thresholds doesn't risk the main infrastructure deploy.

### Alerts created (when deployed)

| # | Name | Severity | Type | Condition (defaults) | Replaces |
|---|---|---|---|---|---|
| 1 | `cviper-backend-5xx-rate` | 1 (error) | metric | >10 HTTP 5xx responses in 5 min | Grafana 5xx panel |
| 2 | `cviper-backend-cpu-sustained` | 3 (info) | metric | CPU >200M nanocores (~80% of 0.25 vCPU) sustained 15 min | Prometheus CPU panel |
| 3 | `cviper-ai-provider-errors` | 2 (warning) | KQL (Log Analytics) | >20 AI gateway ERROR events in 15 min | Grafana `cviper_ai_call_total{status="error"}` |

p95 latency was descoped from the first pass — KQL with `percentile()` is straightforward but the threshold needs operational data to calibrate. Add as a follow-up alert when there's a known baseline.

DB connection saturation also descoped — it lives on a different resource (PostgreSQL Flexible Server) with a different metric provider. Add separately when needed.

### Deploying

```bash
az deployment group create \
  --resource-group cviper-rg \
  --template-file azure/monitor-alerts.bicep \
  --parameters notificationEmail='steven.brady1@gmail.com'
```

Parameters worth knowing:
- `notificationEmail` (required) — recipient for the action group's email receiver
- `alertsEnabled` (bool, default `true`) — set to `false` to deploy alerts in disabled state for soak testing
- `threshold5xxPer5Min` / `thresholdCpuNanocores` / `thresholdAiErrorsPer15Min` — tune without code edits

To extend the action group with SMS, Teams, or webhook receivers: edit the `emailReceivers` block in `monitor-alerts.bicep` or add `smsReceivers` / `webhookReceivers` arrays alongside it. Re-run the deploy command; Azure Monitor handles the action group update idempotently.

### Maintenance

- The `cviper-ai-provider-errors` KQL query depends on backend log shape (`payload.event_type startswith "ai_"` or `payload.component == "ai_gateway"`). If `backend/monitoring.py` JSON log keys change, update the KQL accordingly.
- Metric alerts query Container Apps' built-in `Requests` / `CpuUsageNanoCores` metrics — these are stable Azure-side, no code dependency.

---

## Future work

Tracked under CV-272 follow-ups:

- Calibrate alert thresholds once a few weeks of operational data are in Log Analytics
- Add p95 latency alert and DB connection saturation alert (deferred from first pass)
- Decide whether to keep the `/metrics` Prometheus endpoint exposed (currently kept — zero cost when unscraped, useful if someone runs a one-off scrape)
- Once the alerts are in place and tuned, consider removing the `monitoring/grafana/dashboards/*.json` files — they are not referenced by any runtime path and represent ~5000 lines of duplicate "look at metrics" intent now covered by KQL
