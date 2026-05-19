# Data Protection Impact Assessment — AI Processing

**Version:** 1.0
**Last updated:** 2026-05-19
**Owner:** DPO (Data Protection Officer role — currently the service operator)
**Status:** Draft for private-beta launch; to be reviewed alongside ToS legal review

**Legal basis:** GDPR Article 35 — required where processing is likely to result in a high risk to the rights and freedoms of natural persons, particularly when using new technology and processing special-category data.

**Companion documents:** [PRIVACY_POLICY.md](./PRIVACY_POLICY.md) · [SUB_PROCESSORS.md](./SUB_PROCESSORS.md) · [DATA_RETENTION_SCHEDULE.md](./DATA_RETENTION_SCHEDULE.md) · [TERMS_OF_SERVICE.md](./TERMS_OF_SERVICE.md)

This DPIA follows the seven-step structure recommended by the UK Information Commissioner's Office (ICO).

---

## Step 1 — Identify the need for a DPIA

Processing scope: CViper's AI features (CV analysis, fit scoring, salary estimation, document generation, interview preparation, career insights) involve transmitting user-supplied CV text and job descriptions to third-party large-language-model (LLM) providers, who generate text output that is returned to the user.

A DPIA is required because the processing:

- **Uses new technologies** (third-party LLMs), Art. 35(1)
- May process **special-category data** by inadvertence — CVs can contain or imply: health status (career gaps for medical reasons), trade-union membership, political opinions (volunteer work), religious beliefs (affiliations), and racial / ethnic origin (names, education locations), Art. 9
- Involves **systematic and extensive evaluation of personal aspects** based on automated processing — fit scoring assigns a numerical match between candidate and job, Art. 35(3)(a)
- Includes **large-scale cross-border transfers** to processors in the United States, Art. 44–49

---

## Step 2 — Describe the processing

### 2.1 Nature of the processing

| Step | Description |
|---|---|
| 1. Data ingestion | User uploads a CV (DOCX/PDF/TXT). Backend parses to text. |
| 2. Local pre-processing | Skills + job title profile is extracted (kept indefinitely). Full CV text is encrypted at rest (Fernet AES-128) with a per-user 7-day TTL. |
| 3. User opt-in | User explicitly selects which AI provider(s) to enable in Settings → AI. No AI call fires without an enabled provider. |
| 4. Prompt construction | Backend constructs a structured prompt (system message + JSON schema + truncated CV text + truncated job description). User-supplied input is sanitised to mitigate prompt injection. |
| 5. Transmission | Prompt is sent over TLS 1.2+ to the selected provider's API endpoint. |
| 6. Provider processing | Provider runs the prompt through its model and returns JSON output. Output is logged (encrypted) for debugging and quality. |
| 7. Output presentation | Backend validates the output schema, persists the structured result (encrypted), and returns to the frontend for display. |
| 8. Retention | Full CV text auto-deletes after 7 days of user inactivity. AI prompt logs auto-delete after 180 days. Both delete immediately on account deletion. Demo sessions never persist prompts. |

### 2.2 Scope

| Dimension | Detail |
|---|---|
| Data subjects | Adult job seekers (16+) who have created an account and explicitly enabled AI processing. Private beta size: 50–200 invited users initially. |
| Volume | One CV per user. Per-user daily quotas: Free tier 3 CV analyses + 50 AI calls; Pro tier 5× that; Sandbox 0.5× that. |
| Data categories | CV text (which may include special-category data — see Step 5); job description text (rarely contains personal data of the user). |
| Geographical scope | UK users primarily; processors in UK, EU, and USA. |
| Frequency | Per user-initiated action (no background processing of CVs). |

### 2.3 Context

| Aspect | Detail |
|---|---|
| Source of data | Direct from the data subject (the user uploads their own CV). |
| Relationship | Contract — the user is the data controller for their CV content; CViper is the controller for the service-operation data. |
| Expectations | Users expect AI processing because the service is marketed as AI-powered; the provider is named in the UI for every generated artefact (`scored_by`, `ai_provider`, `generated_by` chips). |
| Vulnerable groups | None targeted. The service is restricted to 16+. |
| Public concerns | Industry-wide concern about LLMs reproducing training data, biased scoring, and fabricated content. Addressed in mitigations (Step 6). |

### 2.4 Purpose

To produce decision-support output (fit scores, tailored documents, interview answers, salary estimates) that helps the user manage their job search. The legitimate-interest test (Step 4) considers whether this purpose can be achieved with less data or less processing.

---

## Step 3 — Consultation

| Stakeholder | Consultation method | Outcome / status |
|---|---|---|
| Data subjects (users) | Privacy Policy v1.2 §5 publishes the AI disclaimer + cross-border transfer notice. Users opt-in per provider in Settings → AI. | Published 2026-05; private-beta users will see it during registration. |
| DPO | This DPIA is authored by the DPO. | Approved subject to private-beta scope. |
| Processors | DPAs from each AI provider linked in [SUB_PROCESSORS.md](./SUB_PROCESSORS.md). | Reviewed. Each provider attests to non-training of API data (default), encryption in transit, deletion on request. |
| Legal counsel | Solicitor briefing for ToS review (Phase D of public-rollout roadmap) includes this DPIA as an attachment for parallel review. | In flight. |
| ICO | Not consulted (consultation only required if residual risk remains high after mitigations — see Step 7). | N/A at this revision. |

---

## Step 4 — Assess necessity and proportionality

### 4.1 Lawful basis

| Activity | Lawful basis | Notes |
|---|---|---|
| Account creation, service provision (search, application tracking) | Art. 6(1)(b) Contract | Necessary to provide the requested service. |
| Sending CV text to AI provider | Art. 6(1)(a) Consent | Explicit opt-in per provider in Settings → AI. Withdrawal stops future calls without affecting past results. |
| Inference of special-category data from CV | Art. 9(2)(a) Explicit consent | The Privacy Policy and registration flow inform users that CVs may contain special-category data and that they consent to the AI processor receiving that data. |
| Error logging, security monitoring | Art. 6(1)(f) Legitimate interest | Balancing test favours the operator: anonymised IPs only, retention 90 days. |

### 4.2 Data minimisation

| Measure | Implementation |
|---|---|
| Truncate CV text before sending | `cv_text[:12000]` cap; sanitisation strips obvious PII not relevant to the prompt (telephone numbers in some flows). |
| Truncate job description | Strip boilerplate, cap at ~6k characters. |
| Strip prompt-injection vectors | Patterns like "ignore previous instructions" are sanitised. |
| Do not send sandbox data to non-sandbox providers | Hard isolation: sandbox users route only to `sandbox_google` / `sandbox_openrouter` / `pluribus`. |
| Do not include user identifiers in prompts | No email, username, ID, or session token is included in the prompt body. |

### 4.3 Storage limitation

| Data | Retention |
|---|---|
| Full CV text | 7-day idle TTL (`cv_text_expires_at`), re-uploadable any time |
| Tailored CV drafts | 30 days from creation or job archive/reject (whichever first) |
| AI prompt + response logs | 180 days, encrypted at rest |
| Demo / sandbox prompts | Never persisted |
| All of the above | Deleted immediately on account deletion |

### 4.4 Accuracy

- AI output is non-deterministic and may be inaccurate; the Privacy Policy §5 and the Terms of Service §6 disclaim accuracy explicitly.
- Users can edit any AI-generated artefact (CV, cover letter, interview answer) before using it externally.
- Fit scores are flagged in-UI with the provider that produced them; users can re-score with a different provider if they disagree.

### 4.5 Less intrusive alternatives considered

| Alternative | Outcome |
|---|---|
| Local-only AI (Ollama) | Offered as an opt-in route. Many users lack hardware to run Ollama; relegated to self-hosted users. |
| EU-only provider (Mistral) | Offered as an opt-in route. Users can avoid cross-border transfer entirely by selecting Mistral exclusively. |
| Keyword-based scoring (no LLM) | Already implemented as the fallback when AI fails. Less accurate than AI but still useful — hybrid scoring uses 80% AI + 20% keyword. |
| No AI features at all | Would undermine the service's value proposition; users have explicitly chosen an AI-powered tool. |

The conclusion is that AI processing is proportionate, with multiple opt-out pathways (Ollama, Mistral-only, keyword-only fallback) preserved for risk-averse users.

---

## Step 5 — Identify and assess risks

| # | Risk | Likelihood | Severity | Inherent score | Notes |
|---|---|---|---|---|---|
| R1 | Special-category data (Art. 9) inadvertently sent to provider as part of CV text | Medium | High | High | CVs routinely contain birthdays, photos, names from which racial/ethnic origin may be inferred, religious affiliations via voluntary roles, health information via career gaps. |
| R2 | Provider uses CV text for model training despite stated opt-out | Low | High | Medium | All listed providers' API tiers attest non-training (default). Risk is provider policy change. |
| R3 | Provider data breach exposes CV text | Low | High | Medium | Encryption in transit (TLS), provider's own at-rest controls, contractual breach-notification (SCCs Art. 9). |
| R4 | Cross-border transfer to USA without adequate safeguards | Low | High | Medium | Mitigated by per-processor DPAs incorporating SCCs Module 2 + UK Extension to DPF where elected. |
| R5 | AI output reproduces verbatim text from training data, leading to plagiarism or copyright issues for the user | Low | Medium | Low | Provider terms shift responsibility to user; CViper's prompt design favours structured JSON output over free-form generation. |
| R6 | AI output is biased against protected characteristics (e.g. age inferred from dates, gender from name) | Medium | Medium | Medium | Active concern. Mitigated by: (a) fairness disclaimers in UI, (b) "show me why" score breakdowns, (c) Ethics review queued (AIE persona). |
| R7 | AI hallucinates content (fabricated employers, achievements) | Medium | Medium | Medium | Prompt templates include explicit "integrity constraint" — every company, title, date must come from the base CV. Fabrication check runs on every generated CV. |
| R8 | Prompt injection from user-supplied job description or CV alters the model's behaviour | Medium | Low | Low | Sanitiser strips known injection patterns. Output is validated against schema before persistence. Worst case: a malformed JSON gets rejected and the user is asked to retry. |
| R9 | User cannot identify which provider produced a given output → cannot exercise informed consent | Low | Low | Low | Every result chip in the UI shows the provider (`scored_by`, `ai_provider`, `generated_by`). |
| R10 | Provider quota / circuit breaker causes silent fallback to a non-consented provider | Low | Medium | Low | Routing only uses providers in the user's configured set. Logged event (`scored_by` shows the actual provider). |
| R11 | Customised prompts (Prompt Lab) remove integrity safeguards and produce fabricated content | Medium | Medium | Medium | ToS §6 and Privacy Policy §5 assign responsibility to the user; in-UI warning before custom prompt activation. |

---

## Step 6 — Identify measures to mitigate risk

| Risk | Mitigation | Residual likelihood × severity |
|---|---|---|
| R1 (special-category inadvertence) | (a) Privacy Policy & registration flow notify users that CVs may contain special-category data, obtain explicit consent (Art. 9(2)(a)); (b) data minimisation: 12k-char truncation; (c) user-controlled provider selection. | Low × High = Medium |
| R2 (training despite opt-out) | (a) Use only API tiers with attested non-training; (b) annual contract review; (c) SUB_PROCESSORS.md tracks each provider's training-opt-out clause; (d) prompt logging detects if a model regurgitates training data. | Low × High = Medium |
| R3 (provider breach) | (a) Provider's own security controls (SOC 2 / ISO 27001 for major providers); (b) breach-notification clauses in DPAs; (c) CViper's local CV-text TTL means most user data is no longer on our side at the time of any breach. | Low × Medium = Low |
| R4 (cross-border) | (a) SCCs Module 2 in every US-processor DPA; (b) Mistral (EU) and Ollama (self-hosted) provided as opt-out routes; (c) Sub-processor index maintained per change. | Low × Medium = Low |
| R5 (verbatim reproduction) | (a) Structured JSON output reduces free-form risk; (b) ToS §6 places responsibility on the user; (c) AI-generated content disclaimer in UI. | Low × Low = Low |
| R6 (bias) | (a) Fairness disclaimer + "show why" panel; (b) periodic AI ethics review (AIE persona); (c) hybrid AI + keyword scoring reduces single-source bias; (d) future: synthetic-CV fairness tests across protected characteristics. | Medium × Medium = Medium |
| R7 (hallucination) | (a) Integrity constraint in every generation prompt; (b) automated fabrication check on generated CVs (`fabrication_check` JSON field); (c) UI shows diff vs base CV. | Low × Medium = Low |
| R8 (prompt injection) | (a) Input sanitisation; (b) schema-validated output; (c) sandbox isolation. | Low × Low = Low |
| R9 (provider attribution) | (a) `scored_by` / `ai_provider` / `generated_by` chips on every result; (b) `<ActiveModelIndicator>` in nav; (c) `ai_meta` derived from gateway truth, not pre-call selection (LESSON-038, Auto-correction rule #39). | Low × Low = Low |
| R10 (silent fallback to non-consented provider) | (a) Routing restricted to user-configured providers; (b) every fallback logged with the actual provider; (c) UI chip shows the actual provider, not the requested one. | Low × Medium = Low |
| R11 (custom prompts removing safeguards) | (a) In-UI warning before Prompt Lab activation; (b) ToS §6 + Privacy Policy §5 assign responsibility; (c) default prompt restore button. | Medium × Medium = Medium (user-assumed) |

---

## Step 7 — Sign off and record outcomes

| Item | Outcome |
|---|---|
| Residual risk level | **Medium** — no individual risk remains High after mitigation. The two Medium residual risks (R1, R6) are intrinsic to AI processing of CVs and are addressed through transparency + user control rather than technical removal. |
| ICO consultation required? | **No** — residual risk is not "high" within the meaning of Art. 36(1). Decision documented here; revisit if a future incident or risk reassessment changes this. |
| Risk acceptance | Accepted by the DPO (service operator) for the **private beta** of 50–200 invited users on web. |
| Conditions for public-launch revision | Before opening registration to the general public, this DPIA is to be re-reviewed alongside: (a) any new AI provider added, (b) ToS solicitor sign-off, (c) production observability re-enabled (currently degraded — see CV-273), (d) any change to the Stripe / paid-tier path that affects what data is sent to billing processors. |
| Approval | DPO — 2026-05-19 (initial publication). |

---

## Appendix A — Mapping to Article 35(7) requirements

| GDPR Art. 35(7) requirement | DPIA section |
|---|---|
| (a) Systematic description of the envisaged processing and purposes | Step 2.1, Step 2.4 |
| (b) Assessment of necessity and proportionality | Step 4 |
| (c) Assessment of risks to rights and freedoms | Step 5 |
| (d) Measures to address risks, demonstrate compliance | Step 6 |

---

## Appendix B — Change log

| Date | Change | Author |
|---|---|---|
| 2026-05-19 | Initial publication — private-beta scope. | DPO |
