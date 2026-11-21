# Karkain ERP — First Implementation Backlog

## First vertical slice

**Slice goal:** Prove that a CRM sales-order event can be ingested, validated against a registered contract, persisted in `db2`, and advanced through a one-step workflow that emits a consolidated summary — real work, not empty scaffolding.

**Karkain modules/files expected:** `src/main.kark` (command dispatcher), `src/master_data.kark`, `src/contracts.kark`, `src/events.kark`, `src/workflow.kark`, `src/reporting.kark`, `src/store.kark` (using `db`, `io`, `json` PARTIAL, `collections`, `encoding`, `crypto`).

**Exact input:** A line-oriented command `karkain-erp event ingest --contract sales_order.v1` with a single flat JSON-style record on stdin containing `event_id=ord-1001`, `source_app=crm`, `partner_id=partner-7`, `amount=250.00`, `currency=USD`.

**Exact expected observable output:** On stdout, `Accepted ord-1001 contract=sales_order.v1 state=applied`; a new row is visible in the `db2` event table; and `karkain-erp report snapshot` prints one summary line `partners=1 applied_events=1 total=250.00`. A duplicate ingest of `ord-1001` prints `Duplicate ignored` and leaves counts unchanged.

**Verification command pattern (Karkain v1.2.0):** With the Go front-end engine selected, build/run the source with the verified C-free native target —
```
karkain build src/main.kark --target native-x86_64-windows -o karkain-erp
.\karkain-erp event ingest --contract sales_order.v1 < fixture.txt
.\karkain-erp report snapshot
```
The build must exit 0 and the executable must print the exact lines above. No test has been run yet; this is the planned verification pattern.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| ERP-001 | Command dispatcher & config profile loading | `main.kark`, `config_profile.kark` | None | All planned subcommands dispatch; no paths hard-coded | Unit, CLI smoke | S | `system`/`env` PARTIAL | Builds, help text lists commands |
| ERP-002 | Master data: organizations & partners | `master_data.kark`, `store.kark` | ERP-001 | CRUD via CLI with unique ids; persisted in `db2` | Unit, integration | M | `db` PRESENT/PARTIAL | CRUD survives restart |
| ERP-003 | Contract registry & version validation | `contracts.kark` | ERP-001 | Events validated against declared version; invalid rejected | Unit | M | `json` PARTIAL | Flat contract validation deterministic |
| ERP-004 | Append-only event bus & idempotent ingest | `events.kark`, `store.kark` | ERP-002, ERP-003 | Duplicate event id ignored; log append-only | Integration, failure | M | OS atomic-write PARTIAL | Replay produces no duplicates |
| ERP-005 | Order-to-cash workflow coordinator | `workflow.kark` | ERP-004 | State transitions New→running→completed; exceptions on failure | E2E, regression | L | `async` PRESENT (cooperative) | Stub-app E2E passes |
| ERP-006 | Exception & reconciliation queue | `exceptions.kark` | ERP-004 | Malformed events dead-lettered; operator override logged | Unit, integration | M | None | All rejected events inspectable |
| ERP-007 | Consolidated snapshot reporting (no JOINs) | `reporting.kark` | ERP-002, ERP-004 | Precomputed summary matches fixture totals | Integration, regression | M | `db2` no JOINs | Snapshot reproducible |
| ERP-008 | Role-based command permissions & audit log | `auth.kark`, `audit.kark` | ERP-001, ERP-002 | Unauthorized command rejected; mutating commands logged | Security, unit | M | `time` PARTIAL wrapper | Roles enforced, log complete |
| ERP-009 | Partial-JSON import/export & batch CLI | `importexport.kark` | ERP-002, ERP-004 | Round-trip of flat records without loss | Integration | M | `json` PARTIAL | Exported file re-imports identically |
| ERP-010 | Minimal local service interface | `service.kark` | ERP-004 | Sequential request/response round-trip; no routing assumed | Integration, platform | L | HTTP framework BLOCKED, TLS BLOCKED | Local-only loopback works |

## Dependency order and gates

1. ERP-001 (foundation) → 2. ERP-002 + ERP-003 (parallel) → 3. ERP-004 (event bus) → 4. ERP-005 + ERP-006 + ERP-007 (workflow/reconciliation/reporting) → 5. ERP-008 + ERP-009 (admin/IO) → 6. ERP-010 (service). Gate between each phase: the actual Karkain v1.2.0 compiled executable must be run and its expected output verified before the next phase begins; all blockers (GUI, JSON, HTTP, TLS, atomic write) remain recorded, not worked around by changing Karkain.

## Status

All items: **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run and no binary exists for this application.
