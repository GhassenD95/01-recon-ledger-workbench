# Payments Ops & Case Investigation Console
**An operator workspace for payment/invoice exceptions — event timeline, correlation, and auditable resolution.**

> Real product: web UI + API + workers + database (not a CLI).  
> Built around operations problems seen in PFE offers: **Medius** (Payments Operations, Smart Verification, expense/invoice flows), **ODDO BHF** (payments & ops context), **Tradrly** (bank reconciliation / invoice capture as capabilities), **Value** (banking data & simulation).

## Why this project
Payments and invoicing break in messy, real ways: a payment covers multiple invoices, amounts don’t line up, events arrive late or twice, and someone has to investigate and resolve the exception. A “reconcile two files” script isn’t enough — operators need a **timeline, correlated context, and an auditable decision**.

## What you can do
- **Ingest & normalize:** import/payment and invoice/statement events from files or feeds; idempotent ingestion (retries/duplicates don’t create duplicate records).
- **Event timeline & correlation:** link related items (payment ↔ invoices/lines) with suggested matches (exact, tolerance, many-to-one/one-to-many) and surface discrepancies.
- **Exception queue (UI):** flag mismatches, missing items, duplicates, delays/partials; prioritize cases for review.
- **Case investigation & resolution:** drill into a case timeline, see why items were (or weren’t) linked, apply/approve a resolution (match, split, write-off/adjust, escalate) with reason codes.
- **Audit & reporting:** immutable record of automated suggestions and human decisions; export case and reconciliation reports.

## Architecture
```text
[Web UI (React/Next)] -> [FastAPI: ingestion, correlation, cases, reporting, RBAC]
                                  |
                     [Postgres: events/records, correlation graph,
                                cases, decisions, audit log]
                                  |
                     [Workers: normalize, correlate/match, exception detection,
                               report jobs]  <->  [Object storage: raw files]
```
- Correlation/matching runs as deterministic logic over normalized events; every suggestion carries an explanation.
- Case actions are stateful and audited; ingestion and resolution are idempotent.

## Hard engineering problems
- **Correlation under ambiguity:** many-to-one/partial settlements, tolerances/fees, and conflicting candidates with explicit, explainable policy.
- **Idempotency & safe reprocessing:** duplicate/late events and reruns must not corrupt state or duplicate decisions.
- **Case workflow correctness:** clear case states and server-enforced transitions; append-only audit of who decided what and why.
- **Explainability:** each auto-suggestion and resolution is traceable to the records/rules/fields behind it.

## Proof (what “done” means)
| Claim | Evidence |
|---|---|
| Correct correlation | Labeled scenarios with precision/recall and error breakdown (miss/false/partial) |
| Deterministic suggestions | Same inputs + rules → identical suggestion output/hash |
| Safe ingestion | Idempotent import/retry tests (no duplicate records or decisions) |
| Trustworthy workflow | State-transition tests; full audit trail per case |
| Performant | Batch correlation benchmark and p95 for case/timeline queries |

## Stack
Python 3.12+, FastAPI, Pydantic v2 · PostgreSQL, SQLAlchemy/Alembic · workers (Celery/arq + Redis) · React/Next.js (timeline & case UI) · pytest, Hypothesis/property tests, Testcontainers, ruff/mypy, GitHub Actions · Docker Compose.

## Status
**Design / spec-first.** Milestones: `event model & ingestion` → `correlation/matching + explainability` → `exception & case workflow` → `timeline UI & audit/reporting` → `benchmarks & hardening`.
