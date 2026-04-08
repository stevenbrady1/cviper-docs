# CViper Service Level Objectives (SLOs)

**Last updated**: 2026-03-28
**Review cadence**: Monthly

---

## SLO Definitions

### SLO-1: HTTP Availability

| Attribute | Value |
|-----------|-------|
| **Target** | 99.5% over 30-day rolling window |
| **Metric** | Ratio of non-5xx responses to total responses |
| **Recording rule** | `cviper:http_availability:ratio_30m` |
| **Error budget** | 0.5% = ~3.6 hours/month of downtime equivalent |
| **Alert** | Fast burn (14.4x, 5m, critical), Slow burn (6x, 30m, warning) |

**Calculation**:
```
1 - (sum(rate(http_request_duration_seconds_count{status=~"5.."}[30m]))
    / sum(rate(http_request_duration_seconds_count[30m])))
```

### SLO-2: HTTP Latency

| Attribute | Value |
|-----------|-------|
| **Target** | 95% of requests complete in < 3 seconds (p95 < 3s) |
| **Metric** | Ratio of requests completing within 3s threshold |
| **Recording rule** | `cviper:http_latency_compliance:ratio_30m` |
| **Error budget** | 5% of requests may exceed 3s |
| **Alert** | Latency SLO breach (< 90% compliance, 10m, warning) |

**Calculation**:
```
sum(rate(http_request_duration_seconds_bucket{le="3.0"}[30m]))
/ sum(rate(http_request_duration_seconds_count[30m]))
```

### SLO-3: AI Provider Success Rate

| Attribute | Value |
|-----------|-------|
| **Target** | 95% of AI calls succeed (not fall back to keywords) |
| **Metric** | Ratio of successful AI calls to total AI calls |
| **Recording rule** | `cviper:ai_success:ratio_30m` |
| **Error budget** | 5% = keyword fallbacks are expected during provider issues |
| **Alert** | AI SLO breach (< 90% success, 10m, warning) |

**Calculation**:
```
sum(rate(cviper_ai_call_total{status="success"}[30m]))
/ sum(rate(cviper_ai_call_total[30m]))
```

---

## Error Budget Policy

| Budget consumed | Action |
|-----------------|--------|
| < 50% | Normal development velocity. Ship features freely. |
| 50-75% | Caution. Review recent deploys. Increase monitoring scrutiny. |
| 75-90% | Slow down. Prioritise reliability work. No risky deploys. |
| > 90% | Freeze non-critical changes. Focus entirely on stability. |
| 100% (exhausted) | Incident response mode. All hands on reliability until budget resets. |

---

## Burn Rate Alerting

Error budget burn rate alerts catch both sudden failures (fast burn) and gradual degradation (slow burn) without noise from transient spikes.

### Fast Burn (Critical)
- **Rate**: 14.4x normal budget consumption
- **Window**: 5 minutes
- **Meaning**: At this rate, the entire monthly error budget will be consumed in ~2 days
- **Action**: Immediate investigation required

### Slow Burn (Warning)
- **Rate**: 6x normal budget consumption
- **Window**: 30 minutes
- **Meaning**: At this rate, the entire monthly error budget will be consumed in ~5 days
- **Action**: Investigate within 1 hour

### AI SLO Breach (Warning)
- **Threshold**: AI success rate drops below 90% (vs 95% target)
- **Window**: 10 minutes
- **Meaning**: Significant AI degradation, keyword fallbacks increasing
- **Action**: Check circuit breaker state, provider status pages

---

## Dashboard

SLO compliance is visible on the `cviper-slo` Grafana dashboard:
- Current availability and latency compliance gauges
- Error budget remaining (% and hours)
- Burn rate timeline
- AI success rate timeline
