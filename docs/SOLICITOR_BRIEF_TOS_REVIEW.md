# Solicitor Briefing — Terms of Service Review

**Date:** 2026-05-19
**Prepared by:** Steven Brady, Operator — CViper
**Audience:** External UK solicitor (commercial / technology / data-protection practice)
**Status:** Draft — fill in solicitor details + send

> This document is a one-time email-ready brief. Replace the **`[BRACKETED]`**
> placeholders before sending. The intent is to give the solicitor enough
> context to scope the engagement on a first read, without forcing them to
> reverse-engineer the project. Attach the four referenced documents.

---

## Email — paste / adapt

Subject: **Terms of Service review for CViper — UK-based AI-powered SaaS, private-beta launch**

Dear **`[Firm name / partner]`**,

I'd like to engage your firm to review the Terms of Service for a small UK-based software-as-a-service product I am about to put into a private invite-only beta. The product is called **CViper** ([cviper.ai](https://cviper.ai)) — it is an AI-powered job-search and CV-analysis tool aimed at UK job seekers, initially the London IT and financial-services contracting market.

The Terms of Service have been drafted in good faith by me as the operator. I am not a lawyer and want a qualified UK solicitor to review them before they bind any real user — even a beta user — to the agreement.

### Engagement scope

I am asking for:

1. **Review of [`docs/TERMS_OF_SERVICE.md` v1.0](./TERMS_OF_SERVICE.md)** for legal soundness under English law and any necessary changes to make the document binding and defensible.
2. **A short opinion** on whether the proposed private-beta operating model (invite-only, 50–200 users, free-tier only, manual Pro grant) requires any additional disclosure or contractual structure beyond what the current ToS provides.
3. **Sign-off** on the AI-related clauses, in particular: §6 (AI-Generated Content), §7 (Acceptable Use prohibitions), §11 (Limitation of Liability — currently capped at £100 or 12-month spend).

### Specific questions I would like your view on

| # | Question |
|---|---|
| Q1 | Is the **limitation of liability** in §11 enforceable for a UK-consumer SaaS, or do I need to carve out additional non-excludable categories (Consumer Rights Act 2015)? |
| Q2 | Does the **AI-Generated Content** clause (§6) adequately allocate responsibility for hallucinated or biased output, given the active UK debate around AI accountability? |
| Q3 | Is **explicit opt-in to AI processing** sufficient as a lawful basis for sending CV text — which routinely contains special-category data — to third-party LLM providers, or do I need a separate Art. 9(2)(a) consent flow? |
| Q4 | Is the **age threshold of 16+** (§2) the right line for a UK service, or should this be 18+ given the financial-services job market context? |
| Q5 | Are the **acceptable-use** prohibitions (§7) — particularly around generating misleading CV content and respecting upstream AI provider terms — drafted in a way that would actually support enforcement (account suspension) without dispute? |
| Q6 | The product **transfers data to US-based AI providers** (OpenAI, Anthropic, Google, xAI). My transfer mechanism is the per-processor DPA + SCCs. Is anything further required for a UK controller under the UK GDPR? |
| Q7 | Does the private-beta context — invite-only, no marketing, no payment — change any of the disclosure obligations versus a public launch? |
| Q8 | Is there anything I have **missed entirely** that you would expect to see in the ToS of a UK SaaS that processes career data and uses third-party AI providers? |

### Attachments

| # | File | Purpose |
|---|---|---|
| 1 | [`docs/TERMS_OF_SERVICE.md`](./TERMS_OF_SERVICE.md) v1.0 | The document for review. |
| 2 | [`docs/PRIVACY_POLICY.md`](./PRIVACY_POLICY.md) v1.2 | Companion document — separately reviewed in-house; included for context. |
| 3 | [`docs/SUB_PROCESSORS.md`](./SUB_PROCESSORS.md) v1.0 | Full list of every third-party processor with DPA links and transfer mechanism. |
| 4 | [`docs/DPIA-AI-Processing.md`](./DPIA-AI-Processing.md) v1.0 | Data Protection Impact Assessment for the AI processing — ICO 7-step format. |

### Project & operator context

- **Operator:** Steven Brady (sole operator).
- **Entity:** **`[insert sole trader / Ltd company / N/A]`** — please advise if I should incorporate before binding beta users to these Terms.
- **Domicile:** United Kingdom; governing law in the ToS is England and Wales.
- **Infrastructure:** Microsoft Azure (UK South region, private VNet PostgreSQL); Cloudflare DNS + CDN.
- **Stage:** Built but never opened to public traffic. Private invite-only beta of 50–200 users planned for the next 2–3 weeks. Public launch later — phasing held back behind compliance, observability re-enablement, and Stripe billing completion.
- **Monetisation at beta:** None. Free tier only; Pro tier exists but granted manually by the operator (no payment flow active).
- **Data subjects:** Adult UK job seekers (current ToS age limit 16+).
- **Data processed:** Account data; full CV text (auto-deleted 7 days after inactivity); job applications; AI prompt/response logs (180-day retention, encrypted at rest, deleted on account deletion); demo sessions never persist data.
- **Sub-processors:** See attachment 3.

### Commercial details

- **Budget:** **`[insert budget — fixed-fee engagement preferred; suggest £[NNN]–£[NNN] for the scope above]`**.
- **Turnaround requested:** **`[insert — ideally within 10 working days from receipt of attachments]`**.
- **Engagement format:** Written marked-up document, plus a 30-minute call to discuss any changes that materially affect the product design.
- **Renewal:** Likely repeat engagement when (a) Stripe billing goes live, (b) public registration opens, (c) additional jurisdictions are added (US / EU / NZ on the roadmap).

### Out of scope for this engagement

- Privacy Policy redrafting (in-house reviewed; will be revisited if Stripe / public launch changes the data flow).
- Sub-processor DPA negotiation (we accept each provider's published DPA as-is).
- US / EU / other-jurisdiction terms (UK only at this stage).
- Trademark, IP, employment, or company-structure advice unless it intersects with the ToS scope.

### Next steps

If the scope above suits your practice, please reply with:

1. A fixed-fee quote (or an hours estimate with a cap).
2. Your standard engagement letter.
3. The earliest date you could turn this around.

If you are not the right person at the firm, I would be grateful for a forwarding introduction.

Kind regards,

**Steven Brady**
Operator, CViper
[steven.brady1@gmail.com](mailto:steven.brady1@gmail.com) · [cviper.ai](https://cviper.ai)

---

## Operator notes — not to send

These are reminders for the operator, not part of the email:

| Item | Note |
|---|---|
| Pre-send checklist | (a) Replace every `[BRACKETED]` placeholder; (b) decide entity question (Q on entity) before sending — solicitor will ask; (c) attach the four documents as PDFs or links; (d) BCC yourself for a record. |
| Where to find a solicitor | UK Law Society directory ([solicitors.lawsociety.org.uk](https://solicitors.lawsociety.org.uk)) — filter by Technology / Data Protection / Commercial. Specialist firms used by UK SaaS founders: Bristows, Bird & Bird, Taylor Wessing (large); for smaller-budget options consider Keystone Law, Marriott Harrison, Wiggin, or a regional commercial firm. |
| Fixed-fee expectation | Single-document ToS review for a UK SaaS at this scale usually £750–£2,500 fixed fee for a 5-page document. The DPIA review may be charged separately as DPO advice. Confirm scope before commit. |
| If turnaround is critical | The roadmap has the solicitor review on a 1–2 week elapsed timeline; private-beta Wave 1 can launch with a "Beta — terms subject to change" wording if the solicitor isn't back in time, with re-acceptance flow when the final version lands. |
| Risk acceptance check | If the solicitor flags issues that require material product changes (e.g. raise the age limit, restructure AI consent flow), pause the beta-launch timeline and replan. |
