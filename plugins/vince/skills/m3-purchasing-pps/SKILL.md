---
name: m3-purchasing-pps
description: Explains Infor M3's Purchasing module (PPS) — purchase order header/line structure, requisition-to-PO flow, supplier agreements, and how a PO ties to goods receipt and supplier invoicing. Use whenever a consultant is scoping a Vince Live workflow that reads outstanding POs, auto-creates a PO from an external system, reports on receipt discrepancies, or asks how M3 purchasing status flow works.
---

# M3 Purchasing (PPS)

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction names
below (e.g. `PPS200MI`) are **typical/commonly-documented across M3 implementations, not verified
against any specific tenant's catalog.** Cross-check the exact name against the tenant's own
`m3-api-catalog-full.json` or the `search_m3_catalog` tool before relying on it — naming and field sets
vary by version and by what's licensed/configured for that tenant.

## What PPS is for

Purchasing covers the buy-side counterpart to Order Entry (see `m3-order-entry-ois`): requisitions,
purchase orders, supplier agreements, and goods receipt. It shares the same header/line shape and a
similar status-progression mental model, which makes it fast to pick up once OIS is understood.

## Core structure

- **Header / line pattern.** A purchase order has a header (supplier, order date, buyer, terms) and
  lines (item, quantity, price, requested/confirmed delivery date, warehouse). As in OIS, line-level
  fields can override header defaults.
- **Requisition → PO.** In many implementations a requisition (internal request to buy) is created
  first and converted to a PO after approval; in simpler setups POs are entered directly. Whether a
  given tenant uses requisitions at all is a real, not-assumable, config question.
- **Status flow.** Roughly: entered → approved/released → sent to supplier → received (fully or
  partially) → matched/invoiced → closed. A PO can sit "open" for a long time between partial receipts,
  so "outstanding PO" usually means "PO lines with quantity ordered > quantity received," not "PO not
  yet closed."
- **Supplier agreements / price agreements.** Like OIS's price-list hierarchy, PPS can source the order
  price from a negotiated supplier agreement rather than a price typed on the PO line. Same design
  question as OIS applies: if a workflow creates POs programmatically, should M3's agreement pricing
  win, or should the source system's price be authoritative?
- **Goods receipt.** Receiving a PO line creates a receipt transaction against inventory (ties into
  `m3-inventory-warehouse-mms-mws`) and is the event that typically drives three-way matching against
  the supplier invoice (PO price/qty vs. receipt qty vs. invoice). Receipt discrepancies (over-receipt,
  price variance) are a common reporting ask.
- **Closing.** A PO line can be manually or automatically closed once fully received and matched;
  partially-received lines that will never be completed usually need an explicit close action, not an
  automatic one — worth confirming per tenant if a workflow depends on "closed" as a filter condition.

## What a consultant typically needs to do here

- **Read outstanding POs** for a supplier, item, or date range — bulk/reporting-scale read. Prefer
  `vince-data-lake-step` or `vince-exportmi-select-step` over iterating a PO-list transaction.
- **Auto-create a PO from an external system** (e.g. a supplier portal, a replenishment calculation
  run outside M3) — a transactional write. Use the native M3 API path (`vince-m3-native-api-step`) or
  the Transform/`GENERIC_API` pipeline (`vince-generic-api-step`), per `vince-native-vs-pipeline-decision`.
- **Report on receipt discrepancies** — usually a bulk read joining PO lines against receipt
  transactions, filtered with `GENERIC_FILTER` or a Transform, delivered via `vince-email-step`.

## Questions to ask before promising a specific field or transaction name

- Does this tenant use requisitions, or are POs entered directly?
- Is PO pricing expected to come from a supplier agreement, or should the source system's price win?
- Does "outstanding" mean "not fully received" or "not yet closed" for this specific report — these
  are different filters and the customer may mean either.

Confirm these with the customer or in the tenant's own catalog before committing to a workflow design.
