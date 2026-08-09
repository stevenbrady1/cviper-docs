# Solicitor Briefing — Terms of Service & Privacy Policy Review

**Date:** 2026-08-09 (supersedes the 2026-05-19 draft, never sent)
**Prepared by:** Steven Brady, Operator — CViper
**Audience:** External UK solicitor (commercial / technology / data-protection practice)
**Status:** Send-ready — fill in the firm name, then send
**Tracked as:** CV-1162 — *Legal: commission solicitor review of Terms of Service and Privacy Policy*

> **What changed from the May draft.** Scope now covers **both** the Terms and the
> Privacy Policy (previously the Privacy Policy was explicitly out of scope). The
> documents under review are the **v2.0 drafts**, not the live v1.x versions. The
> private-beta framing is replaced with go-live framing. Budget and turnaround are
> filled in. Questions are tiered so the core scope can be quoted as a fixed fee.

---

## Before you send — 3 things

1. **Pick a firm** and replace `[FIRM NAME]` in the email below. See *Where to find
   a solicitor* at the bottom.
2. **Attach the 5 documents** listed in the manifest. They are pre-built as Word
   files in `ClaudeReports/legal-pack/` — Word so the solicitor can use tracked changes.
3. **BCC yourself** so you have a dated record of what was sent.

---

## The email — paste and send

**Subject:** Terms of Service & Privacy Policy review — UK AI SaaS pre-launch — fixed-fee enquiry

Dear **`[FIRM NAME]`**,

I am looking to instruct a solicitor to review the Terms of Service and Privacy
Policy for a UK software-as-a-service product before it opens to the public. I
would prefer a **fixed-fee engagement** and have tried to scope this tightly so
that is possible.

### The product

**CViper** ([cviper.ai](https://cviper.ai)) is an AI-powered job-search and
CV-analysis tool for UK job seekers, initially the London IT and
financial-services contracting market. A user uploads their CV, and the service
uses third-party AI models to score how well they fit a given job advert, suggest
CV improvements, estimate salary ranges, and prepare interview answers.

It is built, deployed and functional, but **has not yet been opened to public
registration**. This review is the last item before it does.

### What I need reviewed

Two documents, both drafted in good faith by me. I am not a lawyer:

| # | Document | Length | Status |
|---|----------|--------|--------|
| 1 | **Terms of Service v2.0** | ~9 pages | Draft, not yet published |
| 2 | **Privacy Policy v2.0** | ~14 pages | Draft, not yet published |

Thinner v1.x versions of both are currently on the site. The intention is that
your reviewed versions replace them at launch.

I have flagged every point where I made a legal judgement call I am not qualified
to make. There are **27 such flags** in the two documents, marked
`[SOLICITOR CONFIRM]` inline. To keep this affordable I have sorted them into
what I believe must be settled before launch, and what can follow.

### Tier 1 — the eight I believe are gating

| # | Question |
|---|----------|
| Q1 | **Trading entity.** I currently operate as a sole trader and the drafts name me personally as the contracting party. Should I incorporate before binding users to these Terms, and must a postal address be disclosed for distance contracts? |
| Q2 | **Special-category data.** CVs routinely reveal ethnicity, health, religion and other Article 9 data, even though I never ask for it. I rely on **explicit consent** as the Article 9(2) condition for sending CV text to AI providers. Is that sufficient and correctly worded? |
| Q3 | **Lawful basis.** Is the AI processing better characterised as *consent* or *contractual necessity*? And do the "legitimate interests" bases in the Privacy Policy need a formal Legitimate Interests Assessment, which I have not produced? |
| Q4 | **International transfers.** CV text goes to US-based AI providers (OpenAI, Anthropic, Google, xAI, OpenRouter). I rely on each provider's DPA plus SCCs. Post-Schrems II and post UK-US Data Bridge, is that the right mechanism, and do I need a Transfer Impact Assessment? |
| Q5 | **Limitation of liability.** The cap is the greater of £100 or 12 months' fees. Is that enforceable against UK consumers under the Consumer Rights Act 2015, particularly for users on the free tier? |
| Q6 | **Automated decision-making.** Fit-scoring is profiling. My position is that it is *not* "solely automated decision-making producing legal or similarly significant effects" under Article 22, because output is advisory and the user decides whether to apply. Is that position defensible? |
| Q7 | **Age threshold.** The Terms set 16+. Is that the right line for a UK service of this kind? |
| Q8 | **Anything missing.** Is there anything you would expect to see in the Terms or Privacy Policy of a UK SaaS handling career data and third-party AI, that is simply absent? |

### Tier 2 — important, but I can live with these landing later

Affiliate-link disclosure against CMA and ASA guidance; whether GitHub is an
adequately disclosed sub-processor for user feedback; whether my proposed data
retention periods are proportionate; whether an EU-consumer carve-out is needed
in the jurisdiction clause; whether CCPA applies at my scale; whether the right
to restriction must be self-service rather than by email; and whether Article 37
requires me to appoint a Data Protection Officer.

Happy to drop Tier 2 entirely from this engagement if it helps the fee, and pick
it up in a second pass.

### Commercial

- **Fee:** I am budgeting **£400–£900** for a fixed-fee engagement on Tier 1. If
  that is unrealistic for this scope, I would rather you tell me what is
  realistic, or propose a reduced scope that fits, than decline outright.
- **Turnaround:** ideally **7–10 working days** from receipt.
- **Deliverable:** the two documents returned with tracked changes, plus short
  written answers to the Tier 1 questions. A 30-minute call afterwards if
  anything requires a product change.
- **Likely repeat work:** paid subscriptions go live later this year, and a New
  Zealand market entry is on the roadmap. Both would need a further pass.

### Attachments

1. Terms of Service v2.0 — **for review**
2. Privacy Policy v2.0 — **for review**
3. Sub-Processors index — every third party, with DPA links and transfer mechanism
4. Data Protection Impact Assessment (AI processing) — ICO 7-step format
5. Project fact sheet — one page of operator, infrastructure and data-flow context

### Optional add-on

I have also drafted a second, deeper DPIA covering AI **profiling** specifically,
with eight open questions already consolidated for a reviewer. I have not attached
it. If you would like to quote for signing that off as well, I will send it — but
I do not want to inflate the core scope.

If this suits your practice, could you let me know a fixed fee or a capped hours
estimate, your engagement letter, and the earliest date you could turn it around.
If you are not the right person, I would be grateful for an introduction to a
colleague who is.

Kind regards,

**Steven Brady**
Operator, CViper
[steven.brady1@gmail.com](mailto:steven.brady1@gmail.com) · [cviper.ai](https://cviper.ai)

---

## Attachment manifest

Pre-built as `.docx` in `ClaudeReports/legal-pack/`.

| # | File to attach | Source | Purpose |
|---|----------------|--------|---------|
| 1 | `01-CViper-Terms-of-Service-v2.0-FOR-REVIEW.docx` | `docs/TERMS_OF_SERVICE-v2.0-draft.md` | Under review |
| 2 | `02-CViper-Privacy-Policy-v2.0-FOR-REVIEW.docx` | `docs/PRIVACY_POLICY-v2.0-draft.md` | Under review |
| 3 | `03-CViper-Sub-Processors-v1.1.docx` | `docs/SUB_PROCESSORS.md` | Context for Q4 |
| 4 | `04-CViper-DPIA-AI-Processing-v1.0.docx` | `docs/DPIA-AI-Processing.md` | Context for Q2, Q3, Q6 |
| 5 | `05-CViper-Project-Fact-Sheet.docx` | Generated from this brief | Operator context |

**Held back unless asked for:** the profiling DPIA
(`docs/DPIA-AI-Profiling-DRAFT.md`) and its consolidated sign-off checklist
(`ClaudeReports/audits/2026-07-02-audit-dpia-solicitor-signoff-checklist.md`);
the Data Retention Schedule; and the currently-live v1.x documents.

---

## Project fact sheet — attachment 5

This is the source for attachment 5. `scripts/build_legal_pack.py` extracts
everything between this heading and the next one, so edit it here and rebuild —
never maintain a second copy.

This one-page summary is provided so that the operator, infrastructure and data
flows do not have to be reverse-engineered from the Terms and Privacy Policy.

### Operator and entity

| | |
|---|---|
| **Operator** | Steven Brady — sole operator, no employees |
| **Current status** | Sole trader, United Kingdom |
| **Entity question** | Whether to incorporate before binding users is **Question 1** of the engagement |
| **Governing law** | England and Wales |
| **Contact** | steven.brady1@gmail.com · support@cviper.ai |

### What the product does

A user uploads their CV and pastes or imports a job advert. The service uses a
third-party AI model to produce a fit score with transparent sub-scores rather
than a single opaque number, suggested improvements to the CV for that specific
role, an indicative salary range, and interview preparation material.

All output is advisory. The user decides whether to act on it. Nothing is
submitted to an employer automatically, and no application is ever sent on the
user's behalf.

### Commercial stage

| | |
|---|---|
| **Deployment** | Built, deployed, functional |
| **Public registration** | **Not open.** Production runs with registration disabled. This review is the last item before it opens |
| **Marketing** | None run to date. A waitlist form exists on the homepage |
| **Revenue** | None. No user has ever paid |
| **Paid tier** | A "Pro" tier exists in the product but is granted manually by the operator |
| **Payments** | Stripe integration code is present but disabled — endpoints return HTTP 503 until credentials are set |
| **Mobile** | iOS and Android wrappers of the same web app exist. iOS is in TestFlight only. Neither is publicly released |

### Users

Adult UK job seekers, initially the London IT and financial-services contracting
market. The current age floor in the Terms is 16.

### Infrastructure

| | |
|---|---|
| **Hosting** | Microsoft Azure, UK South region |
| **Database** | PostgreSQL on a private virtual network, encrypted at rest |
| **Secrets** | Azure Key Vault |
| **DNS / CDN** | Cloudflare |
| **Source control / CI** | GitHub — source code only, no production user data |

### Data processed and how long it is kept

| Data | Retention |
|---|---|
| Account data — email, name, auth tokens | Until account deletion |
| **Full CV text** | Automatically deleted after **7 days of inactivity** |
| Derived skills and job-title profile | Retained after CV deletion so job search keeps working |
| Saved jobs and applications | Until the user deletes them, or account deletion |
| AI prompts and responses | 180 days, encrypted, deleted on account deletion |
| Demo sessions | Expire within the hour; nothing persisted |
| Consent records | Retained indefinitely as proof of consent under Article 7 |

### The special-category issue

CVs routinely reveal ethnicity, health, religion, trade-union membership and
other Article 9 data, even though the service never asks for any of it. This is
inherent to the document type. The operator's position is that **explicit consent**
is the Article 9(2) condition. Confirming this is Question 2 of the engagement,
and the operator regards it as the single most consequential item.

### AI providers the user can choose from

| Provider | Location | Notes |
|---|---|---|
| OpenAI | USA | API data not used for training |
| Anthropic | USA | API data not used for training |
| Google Gemini | Global, typically USA | Paid API tier excludes training |
| Mistral AI | France / EU | EU-hosted alternative |
| xAI (Grok) | USA | DPA to be verified before enabling for non-demo users |
| OpenRouter | USA | Gateway routing to upstream providers |
| Operator demo provider | UK | Demo sessions only, never persisted |
| Ollama (self-hosted) | User's own machine | **Nothing leaves the user's network** |

The user selects the provider. The one in use is displayed in the interface at
the time of generation. A full sub-processor index with DPA links and transfer
mechanisms is provided as a separate attachment.

### Safeguards already implemented

- Explicit opt-in before any AI processing takes place
- A **separate** consent gate for cross-border transfer
- Fairness instructions embedded in every scoring prompt
- Transparent sub-scores rather than an opaque single score
- Per-user daily usage caps
- Account deletion in two clicks, removing all derived AI data
- An EU-hosted provider and a fully local provider offered as alternatives to US providers
- No analytics, no tracking pixels, no third-party advertising, no sale of data

### What users see today

The live site currently carries thinner v1.x versions of both documents. The
Terms page displays a visible notice that the document is in beta, is under
external legal review, may change, and that users will be asked to accept an
updated version if it changes materially.

---

## Operator notes — do not send

| Item | Note |
|---|---|
| **Budget realism** | £400–900 is tight for two documents totalling ~23 pages with 27 flagged judgement calls. A single-document ToS review for a UK SaaS commonly quotes £750–£2,500. The email is deliberately written to invite a counter-proposal rather than a refusal, and Tier 2 is explicitly droppable. Expect £900–£1,500 for Tier 1 on both documents. If every quote lands high, the fallback is to review the **Privacy Policy only** — it carries the higher regulatory risk, since the ICO can fine, whereas a weak liability cap only bites if someone sues. |
| **Where to find a solicitor** | Law Society directory — [solicitors.lawsociety.org.uk](https://solicitors.lawsociety.org.uk), filter by Data Protection and Technology. For this budget, target sole practitioners and small commercial firms rather than Bristows / Bird & Bird / Taylor Wessing, who will not engage at this level. Fixed-fee tech-law providers aimed at startups are also worth three enquiries in parallel. |
| **Send to 3 firms, not 1** | Quotes for this work vary by more than 3x. Parallel enquiries cost nothing and protect the 7–10 day timeline — a single non-reply burns a week. |
| **Entity question first** | Q1 may come back as "incorporate before you launch". That has its own lead time at Companies House. Ask it early rather than discovering it at sign-off. |
| **When the review lands** | CV-357 (*Apply solicitor's marked-up ToS changes*) already exists in the backlog for the follow-up work. The v2.0 drafts then replace the live v1.x files, the beta notice comes out of the Terms, and the `[SOLICITOR CONFIRM]` blocks are deleted. |
| **Interim notice** | Added 2026-08-09 under CV-1162. The live Terms page now states the document is in beta and under external review. This replaces the "not reviewed by a qualified solicitor" caveat removed under CV-1160, which had left the Terms reading as an unqualified binding contract. It must be removed when the reviewed version is adopted. |
| **If the solicitor requires product changes** | Raising the age floor, restructuring the AI consent flow, or making right-to-restriction self-service would each need development work. Do not schedule the launch date until the review is back. |
