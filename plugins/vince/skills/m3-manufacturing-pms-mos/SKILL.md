---
name: m3-manufacturing-pms-mos
description: Explains Infor M3's Manufacturing module (PMS manufacturing orders, operations, routings; MOS scheduling) — order and operation structure, work centers, and how manufacturing consumes and produces inventory — and how to map it onto a Vince Live workflow. Use when asked about manufacturing or production orders, operations, routings or work centers, or when a workflow must pull open operations or flag late ones, even if the user just says "production" or "shop floor".
---

# M3 Manufacturing (PMS / MOS)

General M3 functional knowledge, not tenant-specific fact. Program names below were checked against
one Vince tenant's API catalog (2026-09); check the exact name in the tenant's own catalog and its
fields with `vince-field-metadata-lookup` before designing against it.

**Absent from a catalog is not the same as nonexistent.** A captured production Vince Live workflow
calls `PMS100MI/SelOperations`, yet that transaction is missing from both the tenant's API catalog and
its field catalog. `PMS100MI` does list `SelOpeByHead` ("selection of manufacturing order operations by
order header") and `LstOperations`. Flag such gaps; never resolve them by guessing.

## What it's for

PMS tracks the shop-floor work that turns components into finished or semi-finished items; MOS covers
related scheduling. The structure:

- **Manufacturing order (MO) header** — one per production run: item, quantity, planned dates, status
  (planned → released → started → reported → closed).
- **Operations** — the MO's steps (cut, assemble, paint, inspect), each with a work center, planned
  times and its own status. Lateness is operation-level: the header can say "started" while one
  operation is weeks behind.
- **Routing** — the template of operations an item's MOs follow, copied onto each new MO.
- **Work center** — the machine, line or cell an operation runs on; capacity and load are reported per
  work center.
- **Material consumption and reporting** — progress consumes components and reports produced items
  into stock, so an MO is also a source of inventory transactions.

## Mapping work onto Vince Live

- **Open MO operations for a report** — a bulk read: `vince-data-lake-step` (async Compass SQL) or
  `vince-exportmi-select-step`, not a per-order call.
- **Flag late operations** — after the bulk read, compare planned vs. actual dates in a JSONata
  Transform, then route flagged rows to `EMAIL` or a Custom Table (`vince-table-updater-step`).
- **Check one MO's status before acting** — a transactional single read: the native `API` step, per
  `vince-native-vs-pipeline-decision`.

## Where this goes wrong

- Reporting header status when the ask is operation-level lateness.
- Hardcoding a program name from general M3 familiarity without checking the tenant's catalog.

## Not known

- Whether `PMS100MI/SelOperations` exists outside the catalog snapshots, and how it relates to `SelOpeByHead`.
