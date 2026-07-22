# Sub-Processors

**Version:** 1.1
**Last updated:** 2026-07-22
**Owner:** DPO (Data Protection Officer role — currently the service operator)
**Companion documents:** [PRIVACY_POLICY.md](./PRIVACY_POLICY.md) · [DATA_RETENTION_SCHEDULE.md](./DATA_RETENTION_SCHEDULE.md) · [DPIA-AI-Processing.md](./DPIA-AI-Processing.md)

> This document lists every third-party processor (GDPR Art. 28) that
> processes personal data on behalf of CViper. Each entry names the
> legal mechanism for international transfers where applicable and
> links to the public Data Processing Agreement (DPA).
>
> Service integrations that do **not** receive personal data — outbound
> job-board API calls where only search keywords are sent, source-code
> hosting on GitHub — are listed in §3 for completeness but are not
> sub-processors under Art. 28.

---

## 1. Active sub-processors

### 1.1 Infrastructure

| Processor | Role | Data categories | Location | Transfer mechanism | DPA |
|---|---|---|---|---|---|
| **Microsoft Azure** | Application hosting, PostgreSQL database, Key Vault, Blob Storage | All account, CV, application, search, and audit-log data (encrypted at rest with Fernet AES-128; PostgreSQL TDE) | UK South region (Azure data residency UK) | UK adequacy + Microsoft DPA | [Microsoft Online Services DPA](https://www.microsoft.com/licensing/docs/view/Microsoft-Products-and-Services-Data-Protection-Addendum-DPA) |
| **Cloudflare** | DNS resolution, CDN (cached static assets), DDoS protection, WAF | IP addresses, request headers, geographic country, user-agent strings, public assets (no personal data in transit through CDN) | Global edge network (UK + EU PoPs serve UK/EU users) | Cloudflare DPA + SCCs (controller-to-processor) | [Cloudflare DPA](https://www.cloudflare.com/cloudflare-customer-dpa/) |

### 1.2 AI processors (engaged only when the user has configured them in Settings → AI)

The user explicitly selects which AI provider receives their CV / job text. Sandbox users are restricted to the `pluribus` and `sandbox_*` provider routes; their data is ephemeral.

| Processor | Role | Data categories | Location | Transfer mechanism | DPA | API training opt-out |
|---|---|---|---|---|---|---|
| **OpenAI** | AI text generation (GPT family) | CV text + job description (truncated to ~12k tokens), model output | USA | UK→US: SCCs + OpenAI DPA | [OpenAI DPA](https://openai.com/policies/data-processing-addendum/) | Yes — API data not used for training (default since 2023-03-01) |
| **Anthropic** | AI text generation (Claude family) | As above | USA | UK→US: SCCs + Anthropic DPA | [Anthropic DPA](https://www.anthropic.com/legal/dpa) | Yes — API data not used for training (default) |
| **Google (Gemini)** | AI text generation + OAuth login | As above + (OAuth) Google profile email & display name | Global (Google Cloud regions); Gemini API processing typically US | UK→US: Google Cloud DPA + SCCs | [Google Cloud DPA](https://cloud.google.com/terms/data-processing-addendum) | Yes — paid API tier excludes training; free tier may differ — confirm before use |
| **Mistral AI** | AI text generation (open + commercial models) | CV text + job description, model output | France / EU | UK→EU: adequacy (no SCC required) + Mistral DPA | [Mistral DPA](https://mistral.ai/terms#data-processing-agreement) | Yes — La Plateforme API excludes training |
| **xAI (Grok)** | AI text generation | As above | USA | UK→US: SCCs + xAI DPA | xAI DPA — **to verify before enabling for non-sandbox users** | To verify |
| **OpenRouter** | AI gateway routing to multiple upstream model providers | As above + the upstream provider receives the same data | USA (OpenRouter); upstream varies | UK→US: SCCs + OpenRouter DPA; chain-of-DPA via upstream provider | [OpenRouter terms](https://openrouter.ai/terms) — formal DPA on enterprise tier | Upstream-dependent — disclosed per model |
| **Pluribus** | Sandbox / demo AI provider — operator-managed, ephemeral session use only | CV text + job description from demo sessions only; never persisted | UK | UK domestic | Internal service — no cross-border transfer | Not used for training |
| **Ollama (self-hosted)** | AI text generation on user infrastructure | None leaves the user's network | User-controlled | N/A — no transfer | N/A — not a sub-processor under Art. 28 | N/A |

### 1.3 Communication & identity

| Processor | Role | Data categories | Location | Transfer mechanism | DPA |
|---|---|---|---|---|---|
| **Resend** | Transactional email delivery (verification, password reset, job alerts) | Recipient email address, email body (which may include CV-derived references such as job titles) | USA (primary); EU region available | UK→US: SCCs + Resend DPA | [Resend DPA](https://resend.com/legal/dpa) |
| **Apple (APNs + Sign in with Apple)** | iOS push notification delivery; native sign-in (when iOS app launches in Phase H) | APNs device token, notification payload metadata (title + small body); Apple-provided user identifier for SIWA | USA + Apple global edge | UK→US: Apple SDK terms + SCCs | [Apple Developer Program Agreement](https://developer.apple.com/support/terms/) |
| **Google OAuth** | Optional social sign-in | Google profile email + name only | USA + global | UK→US: SCCs + Google Cloud DPA | [Google Cloud DPA](https://cloud.google.com/terms/data-processing-addendum) |
| **Microsoft OAuth** | Optional social sign-in | Microsoft profile email + name only | Microsoft global | UK→US: Microsoft DPA + SCCs | [Microsoft Online Services DPA](https://www.microsoft.com/licensing/docs/view/Microsoft-Products-and-Services-Data-Protection-Addendum-DPA) |
| **LinkedIn OAuth** | Optional social sign-in | LinkedIn profile email + name only | USA | UK→US: SCCs + LinkedIn DPA | [LinkedIn Subscription Agreement (Developer)](https://legal.linkedin.com/api-terms-of-use) |

### 1.4 Payments — scaffolded, **not active in private beta**

| Processor | Role | Data categories | Location | Transfer mechanism | DPA |
|---|---|---|---|---|---|
| **Stripe** | Payment processing (Pro tier checkout + webhooks) — code present, endpoints return 503 until `STRIPE_SECRET_KEY` is set | Payment card data (Stripe-tokenised; never reaches CViper servers), customer email, billing country | Ireland (EU) + USA | UK→EU: adequacy; UK→US for some operations: SCCs + Stripe DPA | [Stripe DPA](https://stripe.com/legal/dpa) |

**Status during private beta**: Pro tier is granted manually by an administrator. No payment data is collected. Stripe will move to §1.4 active before public launch.

---

## 2. International transfer summary

All UK→US transfers rely on a combination of:

1. The processor's **own Data Processing Agreement** (linked above), which incorporates Standard Contractual Clauses (Module 2 — controller-to-processor), and
2. The recipient's compliance with the EU–US **Data Privacy Framework** (DPF) where they have self-certified. (Note: the DPF is a US-specific mechanism; UK transfers may rely on the UK Extension to the DPF where the recipient has elected to extend coverage to UK data.)

UK→EU transfers (Mistral) rely on the **UK adequacy decision** for EEA states.

CViper does not engage any sub-processor located in a country without an adequate transfer mechanism in place.

---

## 3. Non-processor third parties (listed for transparency)

These services either (a) do not receive personal data, or (b) act as independent controllers (not processors) under Art. 28.

| Service | Why it's not a sub-processor |
|---|---|
| **GitHub** (source-code hosting + Actions CI) | Receives application source code only; no production user data. Microsoft owns; covered by Microsoft DPA for any incidental processing during CI. |
| **Job board search APIs** (LinkedIn, Indeed, Reed, Adzuna, Jooble, eFinancialCareers, Jobserve, Remotive, Findwork) | Outbound search-query API calls only. We send keyword strings (e.g. "python developer london") — no user PII, no CV content. Returned job listings are public data. |
| **PyPI / npm registries** | Build-time dependency resolution. No user data leaves the build container. |

---

## 4. Change log

| Date | Change | Author |
|---|---|---|
| 2026-05-19 | Initial publication. | DPO |

---

## 5. Review cadence

This document is reviewed:

- **Every quarter** (routine).
- **Whenever a new processor is added** — adding a processor to the AI router (`backend/ai/providers.py`), wiring a new auth or email vendor, enabling Stripe in production, or shipping a new region of Azure must be paired with a row addition here in the same commit.
- **Whenever a processor changes its own terms** in a way that affects the transfer mechanism (e.g. a new sub-processor region).
- **After any data-protection incident** that involves a processor.

---

## 6. How to report a processor concern

If you believe a sub-processor is mishandling your data, contact us at
[privacy@cviper.ai](mailto:privacy@cviper.ai). You also have the right
to lodge a complaint with the Information Commissioner's Office (ICO) at
[ico.org.uk](https://ico.org.uk).
