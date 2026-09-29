---
name: vince-field-metadata-lookup
description: How to get the real input/output field names, types, lengths and mandatory flags for an M3 program/transaction before using them in a Vince Live workflow — the confirmed metadata endpoint and the m3-field-catalog.sqlite extract. Use when asked "what fields does OIS100MI/AddOrderLine take" or "is this field mandatory", before writing any M3 field name into an API step, a Transform or a filter, or when asked about m3-field-catalog.sqlite.
---

# M3 field-metadata lookup

M3 field names don't follow a convention you can guess from a similar transaction. Look them up.

## The endpoint (confirmed)

```
GET https://api.vince.live/metadata/{connectionId}?details=FIELDS%23{program}%23{transaction}
Authorization: Bearer <token>
```

The `#` separators must be encoded as `%23`. One call returns every input and output field's `name`,
`descr`, `type`, `length` and `mandatory` for that transaction. The token is a short-lived,
tenant-specific Cognito ID token (roughly an hour); if calls start failing, get a fresh one before
suspecting the request.

## `m3-field-catalog.sqlite`

The same endpoint, run across a whole Vince tenant (in the `M3 APIs` project folder):

- `transactions` — `Program`, `TransactionName`, `Description`, `Simu`, `Status`, `FieldsFetchedAt`
  (24,227 rows)
- `fields` — `Program`, `TransactionName`, `Direction` (I/O), `FieldName`, `FieldLabel`, `Type`,
  `Length`, `Mandatory`, `FromPos`, `ToPos` (242,189 rows)

52 transactions have no field rows. It's one tenant's catalog: a transaction missing from it may still
exist elsewhere, so flag a miss rather than concluding.

## Before writing a field name into…

- an `API` step's `input`/`output` (`vince-m3-native-api-step`),
- a Transform that reads or writes a specific field (`vince-transform-step`),
- a `GENERIC_FILTER` `fact` (`vince-generic-filter-step`),

look it up here. Metadata tells you a field exists and its shape — not what value to put in it, and not
what an API step's `type` controls.

## Not known

- Why 52 transactions returned no field rows.
- How far field metadata differs between tenants or M3 versions.
