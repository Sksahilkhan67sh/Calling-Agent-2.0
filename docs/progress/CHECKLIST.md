# Progress Checklist (source of truth)

Tick a box only when all acceptance criteria pass and evidence is linked.

## Track A: Decisions (before Phase 4, runs in parallel)
- [ ] Volume: 100k per month or per day, and peak concurrency
- [ ] Dograh hosting: self-hosted or Cloud, region
- [ ] Telephony: Dograh-supported carrier, DLT and number series (India)
- [ ] STT, LLM, TTS providers and their retention/training terms
- [ ] Analysis LLM provider (post-call)
- [ ] India legal review: TRAI DND/TCCCPR, calling hours, AI disclosure, recording consent
- [ ] Dograh sandbox: 1 manual test call
- [ ] Answers to the 5 open [verify] items
- [ ] GATE: everything written down and test call successful

## Phase 1: Foundation + Data models
- [ ] P1-0 Open items confirmed, decisions recorded, checklist expanded
- [ ] P1-1 Skeleton, settings split, 13 apps + config, custom User model
- [ ] P1-2 Docker compose (Postgres host port 5544), one-command start
- [ ] P1-3 Celery: 5 queues, acks_late, reject_on_worker_lost, time limits
- [ ] P1-4 trace_id, JSON logging, redaction (tests prove no secrets or phones in logs)
- [ ] P1-5 Health endpoints: live, ready (DB + Redis)
- [ ] P1-6 CI: lint, check --deploy, bandit, pip-audit, tests, secret scan
- [ ] P1-7 .env.example (docs variable names) and secrets plan
- [ ] P1-8 Crypto: AES-GCM field + phone HMAC, rotation tests
- [ ] P1-8b Encryption scope list approved
- [ ] P1-9 Models: campaign, retry_policy, agent_config
- [ ] P1-10 Models: contact, contact_history, suppression
- [ ] P1-11 Models: call_attempt, disconnect_event, working_memory_snapshot, webhook_receipt
- [ ] P1-12 Models: transcript, recording_ref, analysis, final_output
- [ ] P1-13 Models: api_client, audit_event
- [ ] P1-14 Indexes + EXPLAIN at 100k rows documented
- [ ] P1-15 Partition-ready design + DB roles design (documents)
- [ ] P1-16 Gate verification + PHASE-1-SUMMARY
- [ ] GATE: one command starts stack, CI green, constraint tests pass, owner verified every PR
