# Karkain ERP — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain ERP.

**Purpose/problem:** Organizations using separate point applications end up with fragmented operational data and manual reconciliation, while monolithic ERPs bundle unwanted functionality at high cost. Karkain ERP provides one coordination layer for organizations using the standalone Karkain business apps.

**Vision:** A deterministic, offline-first integration hub that makes independent applications behave as one coherent system without absorbing their responsibilities.

**Users/segments:** Owner-operators and operations managers in small and midsize businesses; finance leads needing cross-app status without a browser suite; maintainers in offline or air-gapped environments; integrators needing a stable versioned contract between apps.

**Usage scenarios:** A CRM sales order triggers a coordinated fulfilment/invoicing workflow; a receipt in Inventory is matched to an accounting payable; an operator runs a daily cross-app exception report; an integrator replays a missed event after an outage.

**Positioning/value proposition:** A headless CLI/TUI integration hub, not a GUI suite. Value: one operational picture, deterministic offline operation, tiny footprint, transparent licensing, and explicitly labelled jurisdiction rule adapters never described as certified compliant.

**Non-goals/boundaries:** ERP integrates with — and does not duplicate — Karkain CRM (leads/contacts/opportunities), Karkain Accounting (journals, tax, filings) and Karkain Inventory Management (SKUs, warehouses, stock, valuation). No GUI in v1.2.0; no external SQL, JOIN-heavy reporting, legal-compliance certification or alternate language.

## B. Functional architecture

| Module | Responsibility |
|---|---|
| Org & master data | Legal entities, departments, cost centres, partner registry, cross-app identity mapping |
| Contract registry | Versioned integration schemas and event definitions |
| Event bus | Durable append-only event log with publish, subscribe, replay |
| Workflow coordinator | State machines for order-to-cash and procure-to-pay |
| Exception & reconciliation | Detect missing/duplicate events; operator work queue |
| Consolidated reporting | Precomputed summaries (no JOINs), snapshot export |
| Administration | Roles, command permissions, audit log, config profiles |
| Integration adapters | CLI/file adapters to CRM, Accounting, Inventory |
| Import/export | Partial-JSON record import/export, batch commands |

**Core workflows:** Master-data CRUD; register an app and contract version; ingest an event; advance a workflow; reconcile a mismatch; produce a snapshot.

**Admin workflows:** Define roles and command permissions; configure adapters; review audit entries; archive/restore event logs.

**Error/recovery:** Reject schema-invalid events to a dead-letter queue; replay idempotently; quarantine workflows when an app is unavailable; log manual reconciliation overrides.

**Roles/permissions:** Administrator (configuration), Operator (run workflows, resolve exceptions), Auditor (read-only), Service (publish/ingest only). Enforced at the CLI/TUI layer; not a security certification.

**Configuration/customization:** Local profiles select contract versions and adapters; rule adapters are versioned data, each labelled and never claimed compliant. No compiler or language changes.

**Import/export:** Batch master-data and event import/export via simple `json` field extraction; line-oriented text fallback; no full JSON DOM assumed.

**Integration:** Versioned contracts with the three standalone apps over local CLI commands or file drop directories; no TLS, WebSocket or concurrent HTTP framework.

## C. Competitor analysis

Competitors: **Odoo**, **SAP Business One**, **Microsoft Dynamics 365 Business Central**. Evidence and sources are in `docs/competitor-analysis/COMPETITORS.md`; labels match that document.

**Positioning:** [VF] Odoo is a suite of "open source business apps" covering finance, sales, CRM, inventory and more under one subscription. [VF] SAP Business One is an "affordable ERP solution" for SMBs (vendor claim "over 85,000 customers worldwide"). [VF] Business Central "connects finance, sales, service, and operations in a single easy-to-use solution," with Essentials/Premium/Team Member licensing. [RI] All target the same buyer but bundle functionality Karkain avoids.

| Capability | Karkain ERP [P] | Odoo | SAP B1 | Dynamics BC |
|---|---|---|---|---|
| GUI required | No (CLI/TUI/service) | Yes, Web | Yes | Yes |
| Integrates standalone CRM/Acc/Inv | Yes, core design | Bundled (duplicates) | Bundled | Bundled |
| Cross-app data without JOINs | Explicit sync contracts | Relational ORM | Relational DB | Relational DB |
| Offline/air-gapped | [P] target | Partial (on-prem) | Possible (on-prem) | Cloud-first |
| Jurisdiction tax rules | Explicit adapters, no compliance claim | Localizations | Localizations | Localizations |
| External SQL engine | No (embedded `db2`) | PostgreSQL | HANA/SQL | Azure SQL |

**Platform/deployment:** [VF] Odoo offers Online, Odoo.sh and on-premise with browser/mobile UI; SAP B1 cloud or on-premise; BC cloud-first. Karkain [P] is a local CLI/TUI service on embedded `db2` — a [RI] gap for browser-centric buyers, a fit for offline users.

**Licensing (verified only):** [VF] Odoo pricing is geo-localized (INR at access), with One App Free/Standard/Custom and a 12-month introductory discount; amounts are region-dependent. [VF] BC lists Essentials $80, Premium $110, Team Member $8 per user/month paid yearly (informational, region-dependent). SAP B1 price: **UNKNOWN — not verified** ("Request a quote"). Karkain licensing: [P] flat Apache-2.0. No prices invented.

**Strengths/limitations:** [VF/R] Odoo: breadth and deployment choice; API/customization locked to Custom tier. SAP B1: ERP depth; opaque partner-led implementation. BC: Microsoft ecosystem and transparent tiers; separately metered Copilot Credits/Azure.

**Market gaps:** [RI] Deterministic, offline/air-gapped, low-footprint operations without a browser stack; fixed auditable costs; explicit jurisdiction-scoped tax behaviour rather than implied "compliance."

**Differentiation:** [P] Headless integration hub orchestrating standalone apps through stable contracts, never re-implementing them; deterministic offline operation; pluggable, explicitly labelled rule adapters.

**Deliberately not copied:** Bundled CRM/Accounting/Inventory modules, low-code GUI studio, marketplace app-count competition, browser/mobile clients, implicit "compliance" marketing, per-user seat metering.

## D. Technical architecture

**Application/boundaries:** A command dispatcher fronts master data, contracts, events, workflow, exceptions and reporting. Each owns its `db2` tables and exposes narrow Karkain interfaces; communication is by explicit calls and logged events, not shared state. Single executable runs as one-shot CLI, TUI loop or minimal local service; standalone apps are separate processes reached by adapters. No in-process non-Karkain plug-ins.

**UI architecture:** The UI is **CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED)**. A GUI is a planned layer contingent on ecosystem capability, not part of v1.2.0. Text-only TUI; every action has a CLI equivalent.

**Domain model:** Organization, Department, CostCentre, Partner, AppRegistration, ContractVersion, IntegrationEvent, WorkflowInstance, ExceptionItem, SummarySnapshot — coordination-oriented, referencing app-local ids rather than copying transactional records.

**Storage strategy:** Embedded `db2` SQL subset for relational state plus an append-only event log; no external SQL engine and no JOINs — summaries materialized by sequential logic.

**API/events:** Stable CLI subcommands plus a minimal `http` building-block request/response interface; no routing/concurrent-server framework. Events carry id, source app, contract version and flat payload; handlers are idempotent by event id; replay is safe; invalid events are dead-lettered.

**Logging/observability:** Structured text logs with run id, counters for ingested/applied/rejected events and an audit trail. `log`/diagnostics instrumentation is planned, not implemented.

**Configuration/extensions:** A local profile selects contract versions, adapters and storage path; config via `config`/`io` once probed, no machine-specific paths shipped. Extensions are Karkain-native adapters registered in the contract registry; rule adapters are versioned data labelled "not legal advice / not certified." No FFI planned.

## E. Data design

**Entities/ownership:** Organization, Department, CostCentre, Partner, AppRegistration, ContractVersion, IntegrationEvent, WorkflowInstance, ExceptionItem, SummarySnapshot, AuditEntry. Events belong to a source app; workflows reference events; ERP owns only coordination data; standalone apps own transactional records.

**Validation/lifecycle:** Events validated against their ContractVersion before persistence; unique ids and required fields enforced in code. Events: draft→published→ingested→applied→archived; workflows: new→running→waiting→completed/failed/quarantined.

**Persistence/transactions:** `db2` tables plus append-only log. `db_begin`/`db_commit`/`db_rollback` exist in v1.2.0, but **OS-level atomic write is unverified**, so durability claims are limited to the embedded database; checkpoints are conservative and reconciled on startup.

**Backup/restore/migration:** Stop service, copy store and log as a pair, restore by replacing both; no online snapshot. Explicit version integers with adjacent-version Karkain migration routines; no pre-v1.2.0 schema exists.

**Import/export/audit:** Deterministic line-oriented format and simple `json` field extraction, versioned and re-importable. Every mutating command appends an AuditEntry with actor, command and timestamp (once the `time` wrapper is verified).

## F. Karkain implementation architecture

**Capabilities (status per register):** Compiler v1.2.0 VERIFIED; lint VERIFIED; C-free native Windows backend VERIFIED with Go engine required (default `kcc` BLOCKED on this machine — environment). `db` PRESENT/PARTIAL (`db2` subset; `db_begin`/`db_commit`/`db_rollback` present); `json` PARTIAL; `net`/`http` PRESENT (building blocks only); `crypto`/`encoding`/`async`/`collections`/`generics`/`io` PRESENT; `filesystem` PARTIAL; `system`/`env`/`process`/`time` PARTIAL shell wrappers needing probes. No GUI module.

**Runtime/packaging:** Karkain v1.2.0 only, no compiler modification; Go front-end with C-free native target. Required ecosystem components: the three standalone apps and their contract definitions, which must exist before integration work. One native executable packaged in a directory with default config and empty data directory; no machine-specific paths.

**Blockers:** (1) GUI toolkit BLOCKER — CLI/TUI only, GUI a blocked planned layer; (2) full JSON DOM/schema PARTIAL — flat payloads; (3) concurrent HTTP server/routing BLOCKED — minimal sequential service; (4) TLS BLOCKED — local interfaces only; (5) `system`/`env`/`time` PARTIAL — probe before reliance; (6) OS atomic write PARTIAL — checkpoint durability unverified. Each is a recorded blocker/constraint, never resolved by changing Karkain or using another language.

**Native/FFI:** None for MVP; any future FFI must be a justified Karkain-compatible native interface, never a replacement language.

## G. Non-functional requirements

Targets give workload and measurement method; no arbitrary numbers asserted. Practical machine ~4 GB RAM.

| Area | Target / method |
|---|---|
| Performance | Ingest 1,000 events, time with `bench`, expect linear scaling |
| Memory | Profile that batch with `prof`; stay within ~4 GB budget |
| CPU | `prof` during ingest; no component dominates unjustified |
| Scalability | 10× events, record time ratio; document `db2` ceiling |
| Reliability | Inject malformed events and abrupt termination; dead-letter intact, replay idempotent |
| Security/privacy | Role command checks, no secret logging, no outbound telemetry; not a certification |
| Portability | Cross-build Linux; execution UNVERIFIED on this host |
| Maintainability/accessibility | Interfaces documented and unit-tested; text-only keyboard TUI, redirectable output |
| Observability/recovery | Run id, counters, structured logs; kill mid-workflow and confirm no duplicate applied events |

## H. Compatibility and portability

Windows x86-64 toolchain VERIFIED; Linux cross-target output PRESENT but execution NOT VERIFIED; macOS NOT VERIFIED. All source targets v1.2.0 with Go front-end/C-free backend; default `kcc` unusable here (environment blocker). Native executable plus directory package. Install/uninstall by directory operations after export. Adjacent-version config/data migration; export before upgrade. Backward compatibility not required before v1.2.0. Upgrade/rollback via stop, backup pair, replace, migrate, verify; restore on failure. Unsupported: GUI, TLS, WebSocket, external SQL, concurrent HTTP server, non-Windows execution verification.

## I. Testing strategy

All tests **PLANNED**; none have been run for this application.

- **Unit:** Contract validation, identity mapping, idempotency — deterministic on valid/malformed inputs.
- **Integration:** Event bus + `db2` + adapter round-trip — events survive restart without duplicates.
- **E2E:** Order-to-cash with stub apps — expected workflow state and snapshot.
- **Platform-specific:** Windows native compile-and-run exit 0; other platforms cross-build only.
- **Security:** Role restrictions, no secret leakage — unauthorized commands rejected.
- **Performance:** Batch ingest with `bench` — measured, documented scaling.
- **Regression/compatibility:** Replay fixtures and migrate adjacent versions — identical snapshots, lossless.
- **Actual-compiled-executable:** Karkain v1.2.0 build, run native binary — expected stdout.
- **Failure/recovery:** Kill mid-workflow and replay — no duplicate applied events.

## J. Implementation roadmap

**MVP:** Master data, contract registry, event bus, one order-to-cash workflow with stub adapters. **Core:** All workflows, exception queue, snapshots, roles, backup/restore. **Advanced:** Replay tooling, pluggable jurisdiction rule adapters, richer reporting, minimal local service. **Production-readiness:** Durability verification, observability, published limits, cross-target builds, full suite.

**Dependencies/gates:** Contracts before adapters; adapters before workflows; workflows before summaries; durability probes before production readiness. Each phase exits only on passing tests and a verified actual compiled executable. No dates.

**Risks/mitigations:** PARTIAL JSON — flat contracts; no HTTP framework — CLI/file adapters; atomic write unverified — conservative checkpoints; app contracts undefined — define first; GUI BLOCKED — CLI/TUI only. No completion claims.
