# Phase 1 Decisions

Status: **Proposed** (owner sets each to `Accepted`).

## D0. Repository bootstrap
- The repo was empty (no `main`). On the owner's explicit instruction the agent created `main` with an initial commit (`README.md`, `.gitignore`). This is a one-time exception to "never push to `main`".
- Repo is **public**. `docs/source/` is git-ignored and stays local. Branch protection on `main` cannot be verified by the agent; owner to enable it (PR required, 1 approval, no force push, auto-merge off).
- Consequence: no Track A outputs exist in the repo, so no Track A decision is used in Phase 1.

## D1. Schema authority: exactly 16 tables
`Database-Design.md` is final: campaign, retry_policy, agent_config, contact, call_attempt, working_memory_snapshot, disconnect_event, transcript, recording_ref, analysis, contact_history, suppression, final_output, webhook_receipt, api_client, audit_event. Table names set explicitly via `db_table`. UUID primary keys. Every enum is `TextChoices` plus a DB CHECK.

## D2. Approach-doc names mapped to the 16 tables
Verified against the approach `.docx` entity list (it lists `campaign_contacts`, `conversation_sessions`, `conversation_messages`, `call_events`, `retry_policies`, `provider_accounts`, `lead_scores`, `call_analysis`, `recordings`, `audit_logs`).

| Approach-doc name | Becomes | Note |
|---|---|---|
| contacts | `contact` | direct |
| campaigns | `campaign` | direct |
| call_attempts | `call_attempt` | direct |
| retry_policies | `retry_policy` | direct |
| suppression | `suppression` | direct |
| audit_logs | `audit_event` | direct |
| call_analysis, lead_scores | `analysis` | scores are columns |
| recordings | `recording_ref` | metadata only, no URLs |
| conversation_messages | `transcript.content` | one encrypted text per attempt |
| conversation_sessions | `working_memory_snapshot` (loosely) | |
| call_events | `disconnect_event` + `webhook_receipt` (partly) | |
| campaign_contacts | none | `contact.campaign_id` |
| calls | none | `call_attempt` |
| provider_accounts | none | per-campaign Dograh values on `campaign`; secrets in env / Secrets Manager |

## D3. Docs win over any other wording
- `analysis.conversion_probability`: int 0 to 100.
- Transcript text column is `transcript.content`.
- `retry_policy.retry_spacing_seconds`: JSON array; CHECK `jsonb_array_length(...) = max_retries`; every element a positive integer (validated in app code).
- `suppression.contact_id` is the primary key; `phone_hmac` not null and indexed.
- `contact.campaign_id` is a single FK (same phone in two campaigns = two rows).
- Calling window (`window_start`, `window_end`, defaults 10:00 / 18:00) lives on `retry_policy`.

## D4. Webhook header
`X-Dograh-Webhook-Secret` is the normative Dograh header (name configurable via `DOGRAH_WEBHOOK_HEADER_NAME`). `X-Provider-Signature` is a generic future scheme Dograh does not use. Log redaction covers both plus the configured name.

## D5. Tooling and layout
Python 3.12, `uv` with exact pins, `ruff`, `pytest` + `pytest-django`. Layout `backend/config/`, `backend/apps/<app>/`, `backend/tests/`. Backend only: no cookie auth, no CSRF work (later auth is Bearer JWT and `api_client` keys).

## D6. Encryption and partitioning
AES-GCM custom field with key id in the stored value; keys from `FIELD_ENCRYPTION_KEYS` (newest first); separate `PHONE_HMAC_KEY`. `call_attempt` is not partitioned now; stays partition-ready.

## D7. Schema additions beyond Database-Design.md (assumed approved; recommended defaults)
| Addition | Reason | Consequence |
|---|---|---|
| `campaign`: Dograh agent UUID, telephony config id, caller ID; CPS, concurrency, daily call limit, budget limit | Dograh-Integration §4 and Environment-Config §2.2 say these live on the campaign | extra nullable columns |
| `contact`: optional timezone/region | needed later for calling-window logic | store only |
| `call_attempt.created_at` not null | partition key candidate; docs only have `started_at` | one extra column |
| `audit_event.trace_id` nullable | correlate audit rows with logs | one extra column |

## D8. Suppression edge case
Default: follow the docs (`contact_id` PK). A number cannot be suppressed before a contact row exists; re-imports are still caught through `phone_hmac`. Logged in OPEN-ISSUES for Phase 2. Alternative: own `id` PK and nullable `contact_id`.

## D9. Models per app
campaigns: campaign, retry_policy, agent_config. contacts: contact, contact_history. suppression: suppression. calls: call_attempt, disconnect_event, transcript, recording_ref. analysis: analysis, final_output. webhooks: webhook_receipt. accounts: User (minimal), api_client. audit: audit_event. recovery: working_memory_snapshot.

Note: D7 to D8 were not explicitly answered by the owner; the owner said "start your work", so the recommended defaults were applied. Reject any of them in the PR and the change will follow as a new commit.
