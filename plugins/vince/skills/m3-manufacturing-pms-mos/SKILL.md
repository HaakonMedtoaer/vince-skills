---
name: m3-manufacturing-pms-mos
description: Explains Infor M3's Manufacturing module (PMS manufacturing/production orders, operations, routings; MOS scheduling) — order/operation structure, work centers, and how manufacturing consumes/produces inventory — and how to map a manufacturing report or workflow onto it. Use whenever a consultant asks about manufacturing orders, production orders, operations, routings, work centers, or wants to pull open operations / flag late operations for a Vince Live workflow, even if they just say "production" or "shop floor" without naming PMS.
---

# M3 Manufacturing (PMS / MOS) module

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction
names below are **typical/commonly-documented M3 names, not verified against any specific tenant's
catalog.** Cross-check the exact name against the tenant's own `m3-api-catalog-full.json` (or the
`search_m3_catalog` / `get_m3_program` tools) before relying on it — a name that "sounds right" is
exactly the failure mode this project's parent CLAUDE.md exists to prevent, and that discipline
carries over here even though this is general domain knowledge rather than a Vince Live JSON shape.

**Live example of that discipline, already hit in this project**: `PMS100MI/SelOperations` is used
by a real production Vince Live workflow (`operations-workflow.txt`, part of the VinceGenerator
project's confirmed evidence) but is **absent** from that tenant's catalog snapshot. That's not
proof the program doesn't exist — the snapshot is dated and known-incomplete — but it's also not
proof it does. Treat "the program isn't in the catalog" and "the program doesn't exist" as two
different claims, always, and never resolve the gap by guessing.

## What the module is for

Manufacturing (PMS = production/manufacturing orders and operations; MOS covers related order and
scheduling concepts) tracks the shop-floor work that turns purchased/component inventory into
finished or semi-finished items. The core structure a consultant needs:

- **Manufacturing order (MO) header** — one order per production run: item being produced,
  quantity, planned dates, status (planned / released / started / reported / closed).
- **Operations** — an MO is broken into a sequence of operations (e.g. cut, assemble, paint,
  inspect), each with a work center, planned times, and its own status/progress. "Operations
  behind schedule" reporting is operation-level, not order-level — the header status alone won't
  tell you which specific step is late.
- **Routing** — the template of operations and work centers an item's manufacturing orders follow,
  usually defined once per item/item-group and copied onto each new MO.
- **Work center** — the resource (machine, line, cell) an operation runs on; capacity and load
  reporting is usually work-center-level.
- **Material consumption / reporting** — as operations progress, component inventory is consumed
  and produced (semi-finished/finished) inventory is reported in, which is what actually moves
  stock — a manufacturing order isn't just a schedule, it's an inventory transaction source.

## What a consultant typically needs to do here

- **Pull open manufacturing order operations for a report** — this is a bulk, reporting-scale read
  across potentially many orders/operations. Prefer the `DATA_LAKE` Compass-SQL path over a
  transactional per-order call — see `vince-data-lake-step` for the confirmed async submit/poll
  shape, and `vince-exportmi-select-step` as the other confirmed bulk-read path if the specific
  program supports an `EXPORTMI`/`Select`-style raw-SQL parameter.
- **Flag operations behind schedule** — usually a Transform step comparing each operation's planned
  vs. actual/current date (JSONata date math) after the bulk read above, then routing flagged rows
  to `EMAIL` (see `vince-email-step`) or a Custom Table for follow-up (see
  `vince-table-updater-step`).
- **Look up a single MO's status before acting on it** (e.g. before triggering a downstream step) —
  a transactional single-record read, which is the native `API` step's territory, not the bulk
  path — see `vince-m3-native-api-step` and `vince-native-vs-pipeline-decision` for the general
  rule on which path to reach for.

## Where this goes wrong

- Reporting on MO header status alone when the actual ask is operation-level lateness — the header
  can say "started" while one operation is weeks behind and another is on time.
- Assuming a program name from general M3 familiarity is present in a specific tenant's catalog
  without checking — the `PMS100MI/SelOperations` gap above is the concrete, on-record example of
  why that assumption fails silently.
