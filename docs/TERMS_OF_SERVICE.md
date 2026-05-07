# Terms of Service

**Version:** 1.0
**Effective date:** April 2026
**Service:** CViper — https://cviper.ai

> **⚠️ This document has not yet been reviewed by a qualified solicitor.**
> It is a good-faith template written by the service operator to describe
> how the service works and the expectations placed on users. It must be
> reviewed by appropriate legal counsel before being treated as binding
> or before any material change to the service (new processor, price
> model, etc.).

> The in-app copy at [`frontend/src/components/TermsOfService.jsx`](../frontend/src/components/TermsOfService.jsx)
> must be kept in sync with this file. If the two drift, this markdown
> version takes precedence; the component should be updated to match.

---

## 1. Agreement

By creating an account, signing in via an OAuth provider (Google, Microsoft, LinkedIn), or using the demo mode, you agree to these Terms of Service (the **"Terms"**) and the [Privacy Policy](./PRIVACY_POLICY.md). If you do not agree, do not use the service.

These Terms apply to **CViper** (the **"Service"**), operated from the United Kingdom and accessed via https://cviper.ai.

## 2. Who Can Use the Service

You must be at least 16 years old to create an account. If you are using the Service on behalf of an organisation, you represent that you have authority to bind that organisation to these Terms.

The Service is intended for lawful personal and professional job-search use. Commercial resale, automated scraping of the Service, or reuse of AI-generated output in contexts that violate a third-party AI provider's own terms is not permitted.

## 3. Accounts and Security

You are responsible for:

- Keeping your password and API keys confidential.
- All activity that occurs under your account.
- Notifying us immediately if you suspect unauthorised access: [security@cviper.ai](mailto:security@cviper.ai).

We may suspend or terminate accounts that violate these Terms, attempt to bypass rate limits, or put other users at risk.

## 4. Demo Mode

Demo accounts are provided for evaluation. Demo sessions:

- Expire automatically (typically 30 minutes).
- Are rate-limited to prevent abuse.
- Use shared AI provider quotas — features may be restricted when quota is exhausted.
- Do not persist data beyond the session.

Using automated tools to create multiple demo sessions, bypass the IP/fingerprint limits, or exhaust shared quotas is prohibited and may result in an IP block.

## 5. Your Content

You retain all rights to the content you upload (CVs, cover letters, job descriptions, notes). By using the Service, you grant CViper a limited, non-exclusive licence to:

1. Store and process your content to provide the Service.
2. Transmit your content to the AI provider you select, for the purpose of producing the analysis you requested.
3. Generate derivative AI output (fit scores, tailored CVs, interview answers, etc.) that is returned to you.

CViper does not claim ownership of your content and does not use it to train AI models. You are responsible for ensuring you have the right to upload any content (for example, if your CV contains content written by a third party).

## 6. AI-Generated Content

Features like CV analysis, fit scoring, salary estimates, interview preparation, and document generation are produced by third-party AI models selected by you or your administrator. The [Privacy Policy, Section 5](./PRIVACY_POLICY.md#5-ai-generated-content-disclaimer) describes this in full. Key points:

- **No guarantee of accuracy.** AI output may be inaccurate, incomplete, or misleading. Review everything before relying on it.
- **Not professional advice.** AI-generated salary, career, or legal guidance is informational only.
- **Model selection is yours.** Different providers produce different results; the provider you choose is disclosed in the UI at the time of generation.
- **Modified prompts.** If you customise prompts in the Prompt Lab, you assume responsibility for the output. Default prompts include fabrication-prevention safeguards that user edits may weaken or remove.

You are responsible for reviewing AI-generated content before using it in any application, interview, or communication with a third party.

## 7. Acceptable Use

You agree **not** to:

- Use the Service to harass, defame, or impersonate any person or organisation.
- Upload content that is unlawful, infringing, or that you do not have the right to share.
- Attempt to reverse-engineer, decompile, or gain unauthorised access to the Service or its infrastructure.
- Circumvent rate limits, authentication, RBAC controls, or demo-mode restrictions.
- Use the Service to generate content intended to mislead employers about your qualifications, experience, or identity. CViper's default prompts include fabrication-prevention rules — these exist for your protection and the integrity of the hiring process.
- Use the Service to send unsolicited messages (spam) to recruiters or employers.
- Use the Service for any activity that violates a third-party AI provider's terms (for example, generating content that would violate OpenAI's, Anthropic's, Google's, Mistral's, or Microsoft's usage policies).

## 8. Third-Party Services

CViper integrates with third-party services including:

- **AI providers:** OpenAI, Anthropic, Google Gemini, Mistral, Ollama, Microsoft, and others you may configure.
- **Job boards:** LinkedIn, Indeed, Reed, Adzuna, Jooble, and others.
- **Authentication providers:** Google, Microsoft, LinkedIn (OAuth).
- **Infrastructure:** Microsoft Azure (hosting), Cloudflare (DNS + DDoS protection).

Your use of these third-party services is governed by their own terms. CViper is not responsible for outages, rate limits, policy changes, or data handling by third-party providers. Where possible, we offer fallbacks (e.g. keyword-based scoring when an AI provider is unavailable) but we cannot guarantee continuous availability of any third-party feature.

## 9. Service Availability

We aim for high availability but do not guarantee uninterrupted service. We may:

- Schedule maintenance with reasonable notice.
- Throttle or disable features in response to abuse, quota exhaustion, or security incidents.
- Modify or discontinue features with reasonable notice.

The Service is provided **"as is"** without warranty of any kind, either express or implied, including but not limited to implied warranties of merchantability, fitness for a particular purpose, or non-infringement.

## 10. Fees

As of the effective date, CViper is offered free of charge to individual users. If paid tiers are introduced, users will be notified in advance and these Terms will be updated. Any third-party AI provider fees incurred by using your own API keys are your responsibility.

## 11. Limitation of Liability

To the fullest extent permitted by law, CViper and its operators will not be liable for:

- Loss of job opportunities, offers, or employment arising from reliance on AI-generated content.
- Loss of data due to third-party provider outages, account deletion, or service discontinuation.
- Indirect, incidental, special, consequential, or punitive damages.

Our total aggregate liability arising out of or relating to the Service is limited to £100 or, if greater, the amount you paid to CViper in the twelve months preceding the event giving rise to the claim.

Nothing in these Terms limits or excludes liability for fraud, death or personal injury caused by negligence, or any other liability that cannot lawfully be limited.

## 12. Termination

You may delete your account at any time from **Settings → Privacy & Data**. All associated data will be removed in accordance with the [Privacy Policy](./PRIVACY_POLICY.md).

We may suspend or terminate your account at any time if:

- You materially breach these Terms.
- Your account is used for abuse, spam, or security violations.
- We are required to do so by law.

Sections 5 (Your Content), 6 (AI-Generated Content), 11 (Limitation of Liability), and 14 (Governing Law) survive termination.

## 13. Changes to These Terms

We may update these Terms when the Service changes materially. The version number and effective date at the top of this document indicate the latest revision. Material changes will be communicated via the application and, where appropriate, by email. Continued use of the Service after a change constitutes acceptance of the new Terms.

## 14. Governing Law and Jurisdiction

These Terms are governed by the laws of **England and Wales**. Any dispute arising out of or in connection with these Terms is subject to the exclusive jurisdiction of the courts of England and Wales, except that we retain the right to seek injunctive or equitable relief in any competent jurisdiction to protect the Service.

## 15. Contact

For questions about these Terms, contact us at [support@cviper.ai](mailto:support@cviper.ai).

Complaints about data handling should be directed to the Information Commissioner's Office (ICO) at [ico.org.uk](https://ico.org.uk) — see the [Privacy Policy](./PRIVACY_POLICY.md) for details.
