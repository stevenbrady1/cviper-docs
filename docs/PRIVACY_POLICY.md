# Privacy Policy

**Version:** 1.3
**Last updated:** July 2026
**Service:** CViper — https://cviper.ai

> This document is the canonical Privacy Policy for CViper. The public
> copy at [`frontend/public/legal/privacy-policy.md`](../frontend/public/legal/privacy-policy.md)
> is rendered to users and must stay identical to this file. If the
> two drift, this markdown version takes precedence for legal review.

---

## At a glance

- **Your CV text is deleted automatically after 7 days of inactivity.** Your skills + job title profile is kept so job search keeps working — you can re-upload your CV any time to bring it back.
- **Demo / Try CViper sessions leave nothing behind.** Demo accounts auto-expire within an hour; we never persist their prompts or AI responses.
- **You can delete your account in two clicks** from Settings → Privacy & Data. Everything — including AI prompts derived from your CV — is removed.
- **We never sell your data** and don't run analytics, tracking pixels, or third-party ads.

The rest of this document explains what we collect, what we send to AI providers, and the rights you have over your data.

---

## 1. Data Controller

CViper is a job search management application. When you use CViper, you are the data controller for the personal data you enter. The application operator provides the infrastructure and processing capabilities.

**Contact:** [support@cviper.ai](mailto:support@cviper.ai)

## 2. Data We Collect

We collect and process the following categories of personal data:

- **Account data:** Username, email address, display name, and encrypted password hash.
- **CV and career data:** Full CV text (deleted automatically after 7 days of inactivity — see §6), structured profile (skills, job title, summary — kept indefinitely), and AI-generated analyses.
- **Job application data:** Job titles, companies, locations, salaries, application status, notes, and fit scores.
- **Search data:** Search keywords, location preferences, and salary expectations.
- **AI interaction logs:** Prompts sent to and responses received from AI providers (kept for debugging and quality; auto-deleted with your account and never stored for demo/sandbox sessions).
- **Technical data:** Anonymised IP addresses, browser type, and error reports (with URLs redacted).
- **Waitlist data:** If you join the waitlist we store your email address, solely to contact you about access to CViper.

## 3. Lawful Basis for Processing

We process your data under the following lawful bases (GDPR Article 6):

- **Contract (Art. 6(1)(b)):** Processing necessary to provide the job search management service you requested.
- **Consent (Art. 6(1)(a)):** For sending your CV data to third-party AI providers for analysis. You may withdraw consent at any time.
- **Legitimate interest (Art. 6(1)(f)):** For error logging, security monitoring, and service improvement.

## 4. Third-Party AI Providers

When you use AI-powered features (CV analysis, job scoring, document generation), your CV text and job descriptions are sent to one or more AI providers — only the providers **you** have configured in Settings → AI:

- OpenAI (USA)
- Anthropic (USA)
- Google Gemini (Global)
- Mistral (France/EU)
- Grok / xAI (USA)
- OpenRouter (USA — gateway routing to multiple model providers)
- Pluribus (sandbox/demo provider, used only for hosted demo sessions)
- Ollama (self-hosted, no data leaves your infrastructure)

These providers act as data processors. Data sent to US-based providers is protected by Standard Contractual Clauses (SCCs) and each provider's Data Processing Agreement. You can avoid cross-border transfers entirely by using Ollama (self-hosted) or Mistral (EU-based). The full list of providers active on your account is always visible in Settings → AI.

For the complete index of every sub-processor (including infrastructure, OAuth, and transactional-email providers) and the legal basis for each cross-border transfer, contact [privacy@cviper.ai](mailto:privacy@cviper.ai) or see [`docs/SUB_PROCESSORS.md`](./SUB_PROCESSORS.md) in our public documentation.

## 5. AI-Generated Content Disclaimer

CViper uses third-party AI models (selected by you or your administrator) to generate content including CV analysis, job fit scores, salary estimates, interview preparation, and tailored documents. This content is produced by the AI model, not by CViper.

- **No guarantee of accuracy:** AI models can produce inaccurate, incomplete, or misleading information. All AI-generated output should be treated as a starting point, not a definitive assessment.
- **User responsibility:** You are responsible for reviewing and validating all AI-generated content before relying on it for career decisions, applications, or communications with employers.
- **Model selection:** The quality and characteristics of AI output depend on the model and provider you have chosen. Different providers may produce different results for the same input.
- **Not professional advice:** AI-generated salary estimates, career strategy suggestions, and interview guidance are informational only and do not constitute professional career, legal, or financial advice.
- **User-modified prompts:** If you customise the AI prompt templates via the Prompt Lab, you assume full responsibility for the output produced by your modified prompts. CViper's default templates include integrity safeguards (e.g. fabrication prevention rules) that may be weakened or removed by user edits. We cannot guarantee the quality, accuracy, or safety of AI output generated from customised templates.

## 6. Data Retention

We keep your data only as long as we need it.

**Auto-deleted on a schedule:**

| Data | When it's deleted |
|---|---|
| **Full CV text** (the raw document content) | **7 days after you last open CViper.** Your skills + job title profile is kept so job search keeps working; re-uploading your CV at any time brings the full text back. |
| **Tailored CV drafts** (per-job versions you generated) | 30 days after creation, OR immediately when you archive/reject the matching job (whichever comes first). |
| **Saved-search CV snapshots** | 7 days after you last open the saved search. |
| **Demo / Try CViper data** | Demo accounts and all their data are deleted within ~1 hour of session start. AI prompts from demo sessions are never persisted to the database. |
| Search history | 180 days |
| AI prompt logs (paid accounts) | 180 days, or immediately when you delete your account |
| Score history | 180 days |
| Failed login attempts | 90 days |
| Sandbox event analytics | 90 days |
| Database backups | 7 days |
| Session cookies | 24 hours (or 30 days with "remember me") |
| **Waitlist email** (if you joined the waitlist) | Deleted once your invitation is actioned, or on request. |

**Kept until you delete your account:**

- Account data (username, email, hashed password)
- Saved jobs, companies, applications, notes
- Skills and job-title profile (small structured data)
- Salary estimates and insights you've generated

**When you delete your account**, every row referencing you — including all AI prompts and responses derived from your CV — is removed at the same time. This is enforced at the database level, not by a cleanup job.

## 7. Your Rights

Under GDPR, you have the following rights:

- **Right of access (Art. 15):** Export all your data from Settings → Privacy & Data.
- **Right to erasure (Art. 17):** Delete your account and all associated data from Settings → Privacy & Data.
- **Right to data portability (Art. 20):** Download your data in JSON format.
- **Right to rectification (Art. 16):** Edit your profile, jobs, and CV data at any time.
- **Right to withdraw consent (Art. 7):** Withdraw AI processing consent from Settings → Privacy & Data.
- **Right to object (Art. 21):** Contact us to object to specific processing activities.

## 8. Cookies and Local Storage

- **Session cookie** (`session_token`): Strictly necessary for authentication. HttpOnly, SameSite=Lax. Expires after 24 hours or 30 days with "remember me".
- **Local storage:** UI preferences (detail level, advanced mode) and last search criteria. Cleared on logout.

We do not use analytics cookies, tracking pixels, or third-party advertising.

## 9. Security

- Passwords are hashed with bcrypt and per-user salt.
- Sensitive data (CV analyses, AI prompts, API keys) is encrypted at rest using AES-128 (Fernet). The encryption key is validated at production startup — the service refuses to boot with a missing or malformed key, so data-at-rest protection cannot silently fall back to plaintext.
- All connections use HTTPS with TLS 1.2+.
- IP addresses are anonymised in application logs. Email addresses are redacted to domain-only (`<redacted>@example.com`) in operational logs.
- Rate limiting protects against abuse.
- Regular security scanning (OWASP ZAP, Snyk, dependency audits).

## 10. Infrastructure

CViper is hosted on Microsoft Azure (UK region) with Cloudflare providing DNS and DDoS protection. Both act as data processors under their respective Data Processing Agreements.

## 11. Changes to This Policy

We will update this policy when our data practices change. The version number and date at the top of this page indicate the latest revision. Material changes will be communicated via the application.

## 12. Contact & Complaints

For privacy questions or to exercise your rights, contact us at [privacy@cviper.ai](mailto:privacy@cviper.ai).

You also have the right to lodge a complaint with the Information Commissioner's Office (ICO) at [ico.org.uk](https://ico.org.uk).
