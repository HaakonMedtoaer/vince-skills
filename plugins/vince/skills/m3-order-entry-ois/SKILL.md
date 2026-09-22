---
name: m3-order-entry-ois
description: Explains Infor M3's Order Entry module (OIS) — customer order header/line structure, order status flow, pricing/discount determination, and how an order becomes a delivery and an invoice. Use whenever a consultant is scoping a Vince Live workflow that reads, creates, or updates customer orders, needs to explain OIS concepts to a customer, or asks things like "how do I pull open orders," "what determines the price on an order line," or "how does an order become an invoice in M3."
---

# M3 Order Entry (OIS)

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction
names below (e.g. `OIS300MI`) are **typical/commonly-documented across M3 implementations, not
verified against any specific tenant's catalog.** Before building anything against them, cross-check
the exact name against the tenant's own `m3-api-catalog-full.json` or the `search_m3_catalog` tool —
naming, availability, and field sets vary by M3 version and by what's actually licensed/configured
for that tenant.

## What OIS is for

Order Entry is the customer-order module: everything from "a customer wants to buy something" through
delivery and invoicing. It's the module a consultant touches most often because it's usually the
richest source of triggers ("new order created," "order line changed") and the most common target for
automation ("auto-create an order from a webshop/EDI feed," "flag orders matching a condition").

## Core structure

- **Header / line pattern.** An order has one header (customer, order date, order type, terms,
  addresses) and one-to-many lines (item, quantity, price, requested delivery date, warehouse). Almost
  every M3 transactional module follows this same header/line shape — learning it here transfers
  directly to Purchasing (see `m3-purchasing-pps`).
- **Order type.** A configured code that drives behavior: normal sales order, credit/return order,
  intercompany order, etc. Order type is usually the first thing that determines which fields are
  mandatory and which flow (pricing, credit check, delivery) applies — never assume "an order" behaves
  uniformly across order types.
- **Status flow.** Orders move through a status progression roughly: entered → allocated/planned →
  picked → delivered → invoiced. Each status transition can be gated (credit check, stock availability,
  approval) depending on configuration. A consultant asking "why hasn't this order shipped" is usually
  asking "what status is it stuck at and why."
- **Pricing and discounts.** Price is normally determined at order-entry time from a price list /
  agreement hierarchy (customer-specific, customer-group, item-specific, campaign) plus discount rules,
  not hardcoded on the order line. If a Vince Live workflow creates order lines programmatically,
  whether it should *supply* a price or *let M3 calculate it* is a real design decision — supplying a
  price bypasses the pricing engine entirely, which is sometimes exactly what's wanted (e.g. a
  pre-negotiated contract price from an external system) and sometimes a bug.
- **Delivery terms and addresses.** Delivery terms (Incoterms-style codes), ship-to address, and
  requested/confirmed delivery dates live on header and/or line and affect downstream warehouse and
  invoicing behavior.
- **Order → invoice.** Invoicing is normally triggered off delivery (invoice what was actually shipped),
  not off the original order line — so "the order total" and "the invoiced total" can legitimately
  differ (partial shipments, backorders, price changes between order and delivery).

## What a consultant typically needs to do here

- **Read open/outstanding orders** for a customer, item, or date range — a bulk/reporting-scale read.
  Prefer the patterns in `vince-data-lake-step` (Compass SQL against M3 tables) or
  `vince-exportmi-select-step` (`EXPORTMI`/`Select` raw-SQL-via-API pattern) over calling an order-list
  transaction row-by-row.
- **Create or update an order/order line** from an external trigger (webshop, EDI, a scheduled feed).
  This is a transactional write — use the native M3 API path (`vince-m3-native-api-step`) or the
  Transform/`GENERIC_API` pipeline (`vince-generic-api-step`) per the decision rule in
  `vince-native-vs-pipeline-decision`, not a bulk-read path.
- **Report on or flag orders** matching a business condition (e.g. "orders held on credit block for
  more than N days") — usually a bulk read followed by a `GENERIC_FILTER` or a Transform, output via
  `vince-email-step` or a `TABLE_UPDATER` queue (see `vince-table-updater-step`) if the volume needs
  batching across multiple workflow runs.

## Questions to ask before promising a specific field or transaction name

- Is this order type actually in use at this tenant, and does it follow the "normal" status flow, or
  has it been customized?
- Is pricing expected to come from M3's own engine, or is the source system meant to be authoritative
  for price?
- Is the read reporting-scale (use `DATA_LAKE`/`EXPORTMI`) or does it need to see uncommitted/real-time
  state (use the transactional `API`/`GENERIC_API` path)?

None of the above answers can be assumed — confirm them with the customer or in the tenant's own
catalog before designing the workflow.
