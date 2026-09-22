---
name: vince-data-lake-step
description: Confirmed JSON shape and async job pattern for the DATA_LAKE workflow step (target workflow-data-lake) — Compass SQL run straight against M3 tables, for reporting-scale bulk reads. Use whenever drafting or reviewing a bulk data-lake/reporting read, deciding between DATA_LAKE and a transactional M3 API call, or asked "how do I pull a large SQL-style dataset out of M3 in a Vince Live workflow."
---

## What DATA_LAKE is for

`type: "DATA_LAKE"`, `target: "workflow-data-lake"`. Runs Compass SQL directly against M3 tables (`SELECT ... FROM MITMAS WHERE ...`) — a completely different route from the `API`/`GENERIC_API` transactional paths. **Prefer DATA_LAKE for reporting-scale reads; prefer `API`/`GENERIC_API` (see `vince-m3-native-api-step`, `vince-generic-api-step`) for transactional reads and anything that writes.**

## Confirmed M3-side shape

From Topro's MMS090 export — a real, confirmed M3 (not Syteline) capture:

- Endpoint: `https://mingle-ionapi.eu1.inforcloudsuite.com/<TENANT>/DATAFABRIC/compass/v2/jobs`
- `content`: `{endpoint, body, connectionId, saveResultToS3}`
- `body` is itself a template string carrying the SQL built by a prior Transform, e.g. `"{{{ body.query }}}"`

`saveResultToS3` implies an alternate output path (write the result set to S3 instead of/alongside the normal response) — this has been seen in the field list but **not seen actually exercised**, so don't assume you know what happens when it's set.

## It's asynchronous — submit, poll, paginate

DATA_LAKE is not a single request/response call. The confirmed pattern (from the Allett Mowers Syteline integration, and now corroborated for M3 itself via the Topro capture above) is: submit a query to `.../jobs`, poll `.../jobs/{queryId}/status` every 5 seconds, then read results paginated at up to 30,000 rows per call.

Historical note on confidence: the polling/pagination mechanics were originally documented only via Allett Mowers, which is a **Syteline** Datalake integration, not M3 — "M3's Data Lake works the same way" was originally just the customer doc's own unverified claim. That doubt is now resolved: the Topro capture directly confirms the M3-side job/endpoint shape. Treat the async submit/poll/paginate mechanics as confirmed for M3, not just inferred from a different platform.

## Don't confuse this with EXPORTMI/Select

There are two, structurally distinct confirmed bulk-SQL-style read paths in Vince Live — do not conflate them:

- **`DATA_LAKE`** (this skill): asynchronous job submitted to Compass, polled, paginated.
- **`EXPORTMI`/`Select`** (see `vince-exportmi-select-step`): an ordinary, **synchronous** `GENERIC_API` call to M3's normal REST gateway, whose `record.QERY` field happens to carry raw SQL. No job, no polling.

If a workflow needs a quick synchronous bulk pull with no polling logic, `EXPORTMI`/`Select` is very likely the pattern actually in use in production — check that skill before assuming DATA_LAKE is the only bulk-read option.
