---
name: m3-inventory-warehouse-mms-mws
description: Explains Infor M3's Inventory Management (MMS — item master, warehouse balances, stock transactions) and Warehouse Management (MWS — picking, put-away, warehouse tasks) together, since consultants almost always need both at once. Use whenever a Vince Live workflow needs current stock levels, low-stock flags, stock-transaction reporting, or reconciling a warehouse task against an inventory movement.
---

# M3 Inventory (MMS) and Warehouse Management (MWS)

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction names
below (e.g. `MMS200MI`) are **typical/commonly-documented across M3 implementations, not verified
against any specific tenant's catalog.** Cross-check the exact name against the tenant's own
`m3-api-catalog-full.json` or the `search_m3_catalog` tool before relying on it.

## What these modules are for

MMS owns *what an item is and how much of it exists*; MWS owns *the physical work of moving it*.
They're covered together because almost every consulting ask ("what's our stock level," "why did this
item's quantity change") sits at the boundary between the two: MMS has the balance, MWS has the task
that's about to change it.

## Core structure

- **Item master vs. warehouse-specific data.** The item master (MMS) holds attributes that don't vary
  by location (description, unit of measure, item group). Warehouse-specific data (often a separate
  "item/warehouse" record) holds things that do vary by location: reorder point, lead time, planning
  method, and the balances themselves. A consultant asking "what's this item's reorder point" needs
  the item/warehouse record, not the item master — the same item can have different reorder points in
  different warehouses.
- **Balance concepts.** Three numbers matter and are frequently conflated: **on-hand** (physically in
  the warehouse), **allocated/reserved** (on-hand but committed to an order and not available to sell
  or use elsewhere), and **available** (on-hand minus allocated — usually the number that actually
  matters for "can I promise this to a customer" or "do I need to reorder"). A "low stock" report that
  filters on on-hand instead of available will systematically under- or over-flag items.
- **Stock transaction types.** Every balance change happens through a typed transaction (receipt, issue,
  transfer, adjustment, count correction) that's individually logged — this transaction log is usually
  the right source for "what changed and why," not just diffing balance snapshots over time.
- **Warehouse tasks (MWS).** Picking and put-away are generated as tasks assigned to a warehouse
  worker/device; completing a task is what actually creates the underlying stock transaction. If a
  workflow needs to reconcile "the task said X" against "the inventory transaction said Y," the task
  and the transaction are two different records that should agree but aren't the same record — a
  discrepancy here is a real, reportable condition, not a data error to explain away.

## What a consultant typically needs to do here

- **Pull current stock levels** for a set of items/warehouses — bulk/reporting-scale read. Prefer
  `vince-data-lake-step` or `vince-exportmi-select-step` over calling a balance-inquiry transaction
  per item; at real item-count scale, only the bulk-SQL-style paths keep the workflow inside its
  runtime budget.
- **Flag low-stock items for reorder** — a bulk read of available balance vs. reorder point, filtered
  with `GENERIC_FILTER` or a Transform, delivered via `vince-email-step` or written to a
  `TABLE_UPDATER` queue (see `vince-table-updater-step`) if downstream processing needs to happen in
  a separate, more frequent workflow run.
- **Reconcile a warehouse task against an inventory transaction** — usually a bulk read of both,
  joined and compared in a Transform; genuinely mismatched pairs are the report, not something to
  silently drop.

## Questions to ask before promising a specific field or transaction name

- Does "stock level" mean on-hand, available, or something location-specific like "available in
  warehouse X only" — these give different numbers and the customer may mean any of them.
- Is the reorder point itself maintained in M3 (item/warehouse record) or in an external planning
  tool — if the latter, the workflow's read/write direction may be the opposite of what's assumed.
- Is a "discrepancy" between task and transaction actually possible at this tenant's MWS
  configuration, or does their setup make the two atomic (i.e. no discrepancy state can exist)?

Confirm these with the customer or in the tenant's own catalog before committing to a workflow design.
