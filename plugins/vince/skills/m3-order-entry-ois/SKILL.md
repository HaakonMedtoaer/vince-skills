---
name: m3-order-entry-ois
description: Explains Infor M3's Order Entry module (OIS) — customer order header/line structure, order types, status flow, pricing and discounts, and how an order becomes a delivery and an invoice — and how to map order work onto a Vince Live workflow. Use when scoping a workflow that reads, creates or updates customer orders, explaining OIS to a customer, or answering "how do I pull open orders", "what sets the price on an order line" or "how does an order become an invoice".
---

# M3 Order Entry (OIS)

General M3 functional knowledge, not tenant-specific fact. Program names below were checked against
one Vince tenant's API catalog (2026-09); availability and fields vary by M3 version and licensing, so
check the exact name in the tenant's own catalog and its fields with `vince-field-metadata-lookup`
before designing against it.

## What OIS is for

The customer-order module: from "a customer wants to buy" through delivery and invoicing. It's the
module consultants touch most — the richest source of triggers ("order created", "line changed") and
the commonest automation target ("create orders from a webshop or EDI feed", "flag orders that match a
condition"). The order API program is `OIS100MI` (e.g. `AddOrderHead`, `AddOrderLine`, `GetHead`, and
batch-entry transactions such as `AddBatchHead`/`AddBatchLine`).

## Core structure

- **Header / line.** One header (customer, order date, order type, terms, addresses) and one or more
  lines (item, quantity, price, requested date, warehouse). Most M3 transactional modules share this
  shape — it carries straight over to Purchasing (`m3-purchasing-pps`).
- **Order type.** A configured code that drives behaviour — normal sale, return/credit, intercompany.
  It decides which fields are mandatory and which flows (pricing, credit check, delivery) apply, so
  never assume all orders behave alike.
- **Status flow.** Roughly entered → allocated/planned → picked → delivered → invoiced, with gates
  (credit, stock, approval) that depend on configuration. "Why hasn't this shipped?" usually means
  "which status is it stuck at, and why?"
- **Pricing.** Normally set at entry from a price-list/agreement hierarchy (customer, customer group,
  item, campaign) plus discount rules. When a workflow creates lines, whether it *supplies* a price or
  *lets M3 calculate it* is a real design decision: supplying one bypasses the pricing engine — right
  for a pre-agreed contract price from another system, a bug otherwise.
- **Delivery terms and addresses** on header and/or line affect warehouse and invoicing downstream.
- **Order → invoice.** Invoicing normally follows delivery, so order total and invoiced total can
  legitimately differ (partial shipments, back orders, price changes).

## Mapping work onto Vince Live

- **Read open orders** for a customer, item or date range — a bulk read: `vince-data-lake-step` or
  `vince-exportmi-select-step`, not a list transaction called row by row.
- **Create or update orders** from an external trigger — a transactional write: native `API` step or
  the `GENERIC_API` pipeline, per `vince-native-vs-pipeline-decision`.
- **Flag orders matching a condition** (e.g. held on credit for more than N days) — a bulk read, then a
  JSONata **Transform** to filter (`GENERIC_FILTER` can't see M3 output), then `EMAIL` or a
  `TABLE_UPDATER` queue if the volume needs several runs (`vince-table-updater-step`).

## Ask before promising a field or transaction

- Is this order type in use at this tenant, and does it follow the standard status flow?
- Should price come from M3's engine or from the source system?
- Is the read reporting-scale (Data Lake / EXPORTMI) or does it need live state (M3 API)?

## Not known

- Which order types, statuses and pricing rules a given tenant uses — that's configuration, not general M3 knowledge.
