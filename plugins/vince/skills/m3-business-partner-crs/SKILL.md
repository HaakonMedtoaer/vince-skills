---
name: m3-business-partner-crs
description: Explains Infor M3's Business Partner module (CRS) — customer and supplier master data, addresses, contacts and categories — and how to map a business-partner lookup or sync onto a Vince Live workflow. Use when asked about customer or supplier master data, customer categories or groups, pulling customer lists for a mailing or report, syncing supplier data to another system, or looking up a customer's terms before an order workflow, even if the user just says "customer data" or "supplier data".
---

# M3 Business Partner (CRS)

General M3 functional knowledge, not tenant-specific fact. Program names below were checked against
one Vince tenant's API catalog (2026-09); check the exact name in the tenant's own catalog and its
fields with `vince-field-metadata-lookup` before designing against it.

## The key idea

**A business partner is not "a customer" or "a supplier".** It's one identity that can hold either
role, both or neither; the customer and supplier data live in separate records linked to the same
partner number. Treating "customer" and "business partner" as synonyms is the most common scoping
mistake.

- **Business partner** — the shared identity: name, classification, tax IDs.
- **Customer master** — order and invoice data once the partner is a customer: price list, payment
  and delivery terms, credit limit, salesperson. Program `CRS610MI`.
- **Supplier master** — the buy-side equivalent: purchase terms, default warehouse and buyer, supplier
  category. Program `CRS620MI`.
- **Addresses and contacts** — several addresses (invoice, delivery, head office) and contacts, each
  with a role or purpose code. Programs in the `CRS690MI` area are worth searching; don't assume a
  specific one.
- **Categories and groups** — the fields that segment partners for pricing, statistics and reports
  (customer category, sales district, statistics group). A "customers in category X" report usually
  filters on one of these — and they're tenant configuration, so confirm the field and its values.

## Mapping work onto Vince Live

- **Customers or suppliers filtered by category** (a mailing, a report, a Custom Table sync) — a bulk
  read: `vince-data-lake-step` or `vince-exportmi-select-step`, not N single lookups in a `$map`.
- **One customer's terms before creating an order** — a transactional single read: the native `API`
  step, per `vince-native-vs-pipeline-decision`.
- **Sync supplier changes to another system** — a scheduled bulk pull of changed records into a
  Transform, then the sink (Custom Table, `GENERIC_API`, email). Check with
  `vince-field-metadata-lookup` that the transaction offers a changed-since filter before promising an
  incremental sync.

## Where this goes wrong

- Assuming customer number and partner number are interchangeable without checking the role link.
- Filtering on a category field without confirming it exists and holds the expected values.

## Not known

- Which category and group fields a tenant uses, and which address transaction fits a given need.
