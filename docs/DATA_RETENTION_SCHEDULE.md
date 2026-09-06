# Data Retention Schedule

**Version:** 1.0
**Last updated:** 2026-04-12
**Owner:** DPO (Data Protection Officer role — currently the project owner)
**Legal basis:** GDPR Art. 5(1)(e) — storage limitation

## Principles

1. Personal data is kept only as long as necessary for the purpose it was collected
2. When retention expires, data is archived (JSON export) then deleted
3. Users can delete their account at any time via `DELETE /api/gdpr/delete-account` (Art. 17)
4. Users can export their data at any time via `GET /api/gdpr/export` (Art. 15/20)

## Retention Periods

| Data Category | Table(s) | Retention Period | Automated | Justification |
|---|---|---|---|---|
| **User account** | `users` | Until user deletes account | Manual (user-initiated) | Necessary for service provision (Art. 6(1)(b)) |
| **Saved jobs** | `jobs` | Until user deletes account | Manual | Core feature — user's job tracker |
| **Companies** | `companies` | Until user deletes account | Manual | User-curated data |
| **CV versions** | `cv_versions` | Until user deletes account | Manual | User-uploaded content |
| **CV analyses** | `cv_analyses` | Until user deletes account | Manual | Derived from user's CV |
| **Search profiles** | `search_profiles` | Until user deletes account | Manual | User-configured criteria |
| **Search history** | `searches` | 180 days | Yes | Historical analytics; not needed long-term |
| **Score history** | `score_history` | 180 days | Yes | Trend tracking; not needed long-term |
| **Seen jobs** | `seen_jobs` | 180 days | Yes | Deduplication; expires naturally |
| **AI prompt logs** | `ai_prompt_logs` | 180 days | Yes | Debugging; contains encrypted prompts |
| **Career insights** | `career_insights` | 180 days | Yes | Periodic analysis; regenerated on demand |
| **Notifications** | `notifications` | 180 days | Yes | Transient UI alerts |
| **Affiliate clicks** | `affiliate_clicks` | 180 days | Yes | Analytics tracking |
| **Admin audit logs** | `admin_audit_logs` | 365 days | Yes | Compliance — longer retention for accountability |
| **Withdrawn consents** | `user_consents` (withdrawn) | 365 days | Yes | GDPR Art. 7(1) — proof of withdrawal |
| **Active consents** | `user_consents` (active) | Until withdrawn or account deleted | Manual | Required for lawful processing |
| **Password reset tokens** | `password_reset_tokens` | 7 days past expiry | Yes | Security; single-use tokens |
| **Email verification tokens** | `email_verification_tokens` | 7 days past expiry | Yes | Security; single-use tokens |
| **Failed login attempts** | `failed_login_attempts` | 90 days | Yes | Security monitoring |
| **Session tokens** | `user_sessions` | 24h (or 30d with remember-me) | Yes | Session management |
| **Sandbox users** | `users` (is_sandbox=true) | 30 minutes | Yes | Ephemeral demo accounts |
| **Sandbox events** | `sandbox_events` | 90 days | Yes | Abuse monitoring |
| **Sandbox usage** | `sandbox_usage` | 90 days | Yes | Rate tracking |
| **Waitlist signups** | `waitlist_signups` | Until the invitation is actioned, then deleted within 7 days | Planned — CV-1390 (purge in `_scheduled_maintenance()` after the CViper Light launch invitation, CV-1388) | Art. 5(1)(e) storage limitation; matches Privacy Policy §6 "deleted once your invitation is actioned" |

## Automated Enforcement

All automated retention is enforced by the `_scheduled_maintenance()` loop in `backend/main.py`:
- Runs every 24 hours (configurable via `MAINTENANCE_INTERVAL_HOURS`)
- Default retention: 180 days (configurable via `AUTO_CLEANUP_DAYS`)
- Archives records as JSON before deletion
- Archive files auto-deleted after 30 days

## User Rights

| Right | Endpoint | Description |
|---|---|---|
| Access (Art. 15) | `GET /api/gdpr/export` | Full data export in JSON |
| Erasure (Art. 17) | `DELETE /api/gdpr/delete-account` | Complete account deletion with cascade |
| Consent withdrawal (Art. 7) | `DELETE /api/gdpr/consents/{type}` | Withdraw specific consent |
| Portability (Art. 20) | `GET /api/gdpr/export` | Machine-readable JSON export |

## Sub-Processors

| Processor | Purpose | Data Shared | DPA |
|---|---|---|---|
| Azure (Microsoft) | Hosting, database, blob storage | All data (encrypted at rest) | Covered by Azure DPA |
| Cloudflare | DNS, CDN, DDoS protection | IP addresses, request headers | Covered by Cloudflare DPA |
| OpenAI | AI analysis (when selected by user) | CV text, job descriptions (truncated) | Covered by OpenAI DPA |
| Anthropic (Claude) | AI analysis (when selected by user) | CV text, job descriptions (truncated) | Covered by Anthropic DPA |
| Google (Gemini) | AI analysis (when selected by user) | CV text, job descriptions (truncated) | Covered by Google Cloud DPA |
| Mistral AI | AI analysis (when selected by user) | CV text, job descriptions (truncated) | Covered by Mistral DPA |

## Review Schedule

This schedule is reviewed:
- Quarterly (routine)
- When new data categories are added
- When legal advice changes the retention basis
- After any data protection incident
