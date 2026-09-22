---
name: vince-field-metadata-lookup
description: Use when a consultant needs the real input/output field names, types, lengths, or mandatory flags for an M3 program/transaction before writing it into a Vince Live workflow — e.g. "what fields does OIS300MI/AddLine take", "is this field mandatory", "check this transaction's field shape", or any moment about to hand-guess a field name in a Transform template or an API step. Also use when asked about m3-field-catalog.sqlite or how it was built.
---

# M3 field-metadata lookup

Never guess an M3 field name. This is the confirmed endpoint that returns the real, per-tenant answer.

## The confirmed endpoint

```
GET https://api.vince.live/metadata/{connectionId}?details=FIELDS%23{program}%23{transaction}
```

The `#` in `FIELDS#{program}#{transaction}` **must** be percent-encoded as `%23` in the query string.
One call returns every input/output field's `name`, `descr`, `type`, `length`, and `mandatory` for that
transaction.

Auth: `Authorization: Bearer <token>` — a short-lived Cognito ID token, tenant-specific, expiring
roughly hourly. Ask for a fresh one rather than assuming an old one still works; a stale token fails
quietly enough that it's worth checking first if a call behaves unexpectedly.

## m3-field-catalog.sqlite

This same endpoint, called across an entire tenant's catalog, is what built
`m3-field-catalog.sqlite`: 24,227 transactions, 242,189 field rows (`Program`, `TransactionName`,
`Direction` I/O, `FieldName`, `FieldLabel`, `Type`, `Length`, `Mandatory`, `FromPos`/`ToPos`), built
from the same tenant as `token.txt`.

**This is a real, usable resource today for a consultant doing manual field lookups.** It is *not*
yet wired into the generator (`app-template.html`) — there is no build-time or draft-time lookup path
that verifies field references the way `verify()` already verifies program/transaction names against
the catalog. That integration is a documented, not-yet-started next step. Don't assume the generator
checks field names just because it checks program/transaction names — as of now, it doesn't.

## Practical use

Before writing a field name into:
- an `API` step's per-transaction `input`/`output` map (see `vince-m3-native-api-step`),
- a JSONata Transform template that reads or writes a specific field (see `vince-transform-step`),
- a `GENERIC_FILTER` fact/value pair (see `vince-generic-filter-step`),

look the field up with this endpoint (or query `m3-field-catalog.sqlite` if it's available for the
tenant in question) rather than pattern-matching from a similar transaction you remember. Field names
in M3 do not follow a guessable convention reliably enough to skip this step — that's the same reason
program/transaction names get verified against `m3-api-catalog-full.json` rather than trusted from
memory (see the catalog's own `verify()` logic in this project).

## Scope limit

This endpoint gives field *metadata* — name, type, length, mandatory. It does not tell you the
runtime *value* semantics (what a given code actually means, what `type: "ADD"` vs `"LIST"` really
controls on a transaction — see the open question noted in `vince-m3-native-api-step`). Field
metadata answers "does this field exist and what shape is it," not "what should I put in it."
