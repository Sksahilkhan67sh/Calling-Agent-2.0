# Phase 1 Open Issues

Found while reading the docs for P1-0. Docs in `docs/source/` are not edited. More may be added by later tasks.

| # | Issue | Sources | Phase 1 handling | Needs owner? |
|---|---|---|---|---|
| 1 | One contact in several campaigns (approach doc) vs one campaign per contact (DB design, ER §3) | approach `.docx` vs Database-Design §2.4, ER-Diagram §3 | Follow DB design: one row per campaign | Confirm |
| 2 | Reconnect is a **new** attempt (Call State Machine §3) vs retried **in place** on the same attempt, "not a new attempt record" (Database Design §3) | Call-State-Machine §3 vs Database-Design §3 | Schema supports both (unique `(contact_id, attempt_number)`, snapshot per attempt, no uniqueness on snapshot). Phase 2 must pick one | Yes, before Phase 2 |
| 3 | Dialer states `DISPATCHING`, `DIALING`, `IN_CALL`, `DIAL_REQUESTED` appear in Django-Backend-Design §5-6 and Dograh-Integration but are not in the `call_attempt.state` enum (Initiated, Connected, FailedToConnect, DroppedMidCall, EndedNormally) | Django-Backend-Design §6 vs Database-Design §2.5 | Use the docs enum. Where `DIAL_REQUESTED` is stored is undecided; this also decides the partial-index predicate on `next_dispatch_at` (P1-14) | Yes, before P1-11 |
| 4 | `final_output`: one per contact ever (ER §3) vs one per contact per terminal call (Database Design §2.13) | ER-Diagram §3 vs Database-Design §2.13 | Unique on `source_attempt_id` (as the task spec says). Not unique on `contact_id` | Confirm |
| 5 | Storage placement: `working_memory_snapshot` in an in-memory store, `final_output` in an Analytics Store (Database Design §4) vs Postgres as single source of truth (golden rule 1) | Database-Design §4, Environment-Config `MEMORY_STORE_URL` / `ANALYTICS_STORE_URL` | All 16 tables in Postgres. Those two env vars are not created in Phase 1 | Confirm |
| 6 | `final_output.transcript_ref` / `recording_ref` typed as string in the ER diagram, uuid FK in the table spec | Database-Design §1 vs §2.13 | uuid FKs (table spec) | No |
| 7 | Suppression keyed by `contact_id` cannot hold a number that has no contact row yet (for example an imported DNC list). Docs note says check "against `phone_number`" while the table only has `phone_hmac` | Database-Design §2.12 | Follow docs; match by `phone_hmac` | Yes, Phase 2 |
| 8 | Dedupe at DB level: FR-1.3 asks for dedupe on import but Database Design only lists a non-unique `(phone_hmac, campaign_id)` index | Database-Design §2.4 | **No** unique constraint on `(campaign_id, phone_hmac)` in Phase 1; non-unique index only | Yes, P1-10 |
| 9 | Timestamps are "timestamp" with no timezone in the docs | Database-Design | Use timezone-aware (`USE_TZ=True`) | No |
| 10 | Django-Backend-Design §4.1 describes httpOnly cookie JWT and CSRF for a dashboard | Django-Backend-Design §4.1 | Out of scope (backend only, Bearer JWT) | No |
| 11 | Docs require analysis model/prompt version to be persisted per analysis (Environment-Config §2.3) but `analysis` has no such columns | Environment-Config §2.3 vs Database-Design §2.10 | Add only if Lead-Scoring-Specification requires it, decided in P1-12 | Yes, P1-12 |
| 12 | `PRIMARY_DB_URL` is the docs' variable name; Django settings usually use `DATABASE_URL` | Environment-Config §2.1 | Use `PRIMARY_DB_URL` | No |
