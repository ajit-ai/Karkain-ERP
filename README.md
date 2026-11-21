# Karkain ERP

An offline-first, CLI/TUI enterprise integration hub that coordinates operations across the standalone Karkain business applications without duplicating them.

## Status

| Item | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain ERP is an orchestration and integration layer, not a monolithic suite. It holds shared organizational reference data (legal entities, departments, cost centres, trading partners), defines stable internal contracts between the standalone Karkain CRM, Karkain Accounting and Karkain Inventory Management applications, and coordinates cross-application workflows such as order-to-cash and procure-to-pay. It maintains a consolidated, precomputed operational view by consuming published summaries from each standalone app rather than copying their transactional data. It reports on cross-app status, surfaces exceptions when an integrated app fails to publish an expected event, and exposes a scriptable command surface for headless automation. All interaction is through a command-line interface, a text UI, and local service endpoints; there is no graphical desktop client.

## Who it serves

Owner-operators and small-to-mid-size teams that need one coherent operational picture but cannot justify tier-1 ERP cost or implementation. Typical users are operations managers, finance leads and systems maintainers working in deterministic, offline or air-gapped environments who value local data ownership, auditable behaviour and a tiny footprint over browser-based convenience. Integrators who need a stable, versioned contract between business apps are a secondary audience.

## Planned features

- **Integration contracts** — versioned sync contracts to CRM, Accounting and Inventory (planned); event publish/subscribe with replay (planned); schema validation of exchanged records (planned).
- **Shared master data** — organizations, departments, cost centres and partner registry (planned); cross-app identity mapping (planned).
- **Cross-app workflows** — order-to-cash and procure-to-pay coordination (planned); exception and reconciliation queues (planned).
- **Consolidated views** — precomputed operational dashboards without SQL JOINs (planned); exportable summary snapshots (planned).
- **Administration** — role-based access to commands (planned); audit logging of integration events (planned); configuration profiles (planned).
- **Automation** — scriptable CLI commands (planned); batch import/export via partial JSON (planned).

## Explicit non-goals

- Re-implementing CRM lead/contact/opportunity management, Accounting journal entries or Inventory stock tracking.
- Providing a native GUI desktop application in v1.2.0.
- Legal or statutory compliance certification, including jurisdiction-specific tax filing.
- External SQL engines, JOIN-heavy reporting, or a browser/mobile client.
- Replacing any standalone Karkain business app; ERP augments and coordinates them.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

Windows x86-64 toolchain VERIFIED (compile to native executable and execute). Linux cross-target output PRESENT but execution NOT VERIFIED. macOS NOT VERIFIED. These describe the toolchain, not this application, whose compilation is NOT STARTED.

## Build and run status

No application binary exists yet. The intended generic commands are `karkain build src/main.kark -o karkain-erp` and `karkain run src/main.kark`. These become active only when implementation starts; until then they are informational and have not been executed for this application.

## Known blockers

- Native GUI toolkit BLOCKED — CLI/TUI/service-first design; GUI is a planned, blocked layer.
- Full JSON DOM/schema PARTIAL — only simple field extraction is available; integration payloads must stay simple.
- Embedded `db2` SQL subset with no JOINs; no external SQL drivers.
- Concurrent HTTP server framework/routing BLOCKED — only building blocks; service surface is minimal.
- `system`/`env`/`process`/`time` PARTIAL shell wrappers — must be probed before reliance.
- OS atomic-write/file locking PARTIAL — durability of integration checkpoints unverified.

## Repository

GitHub: https://github.com/ajit-ai/Karkain-ERP — issues: https://github.com/ajit-ai/Karkain-ERP/issues — License: Apache-2.0.
