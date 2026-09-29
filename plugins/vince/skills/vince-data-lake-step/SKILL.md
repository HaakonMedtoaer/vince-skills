---
name: vince-data-lake-step
description: Confirmed JSON shape and async job pattern of the Vince Live DATA_LAKE workflow step (target workflow-data-lake) — Compass SQL run straight against M3 tables for reporting-scale reads. Use when drafting or reviewing a bulk or reporting read, when choosing between DATA_LAKE, EXPORTMI/Select and a transactional M3 call, or when asked how to pull a large SQL-style dataset out of M3 in a Vince Live workflow.
---

# `DATA_LAKE` step

`type: "DATA_LAKE"`, `target: "workflow-data-lake"`. Runs Compass SQL directly against M3 tables
(`SELECT … FROM MITMAS WHERE …`) — a separate route from the M3 API.

## Confirmed shape (Topro's MMS090 export, a live M3 solution)

- Endpoint: `https://mingle-ionapi.eu1.inforcloudsuite.com/<TENANT>/DATAFABRIC/compass/v2/jobs`
- `content`: `{endpoint, body, connectionId, saveResultToS3}`
- `body` is a template carrying SQL built by the previous Transform, e.g. `"{{{ body.query }}}"`.
- `saveResultToS3` exists but has never been seen exercised — don't assume what it does.

## It's asynchronous

The pattern is submit → poll → read in pages: post the query to `…/jobs`, poll
`…/jobs/{queryId}/status`, then read results paginated. The timings — poll every 5 s, up to 30,000
rows per page — come from Allett Mowers, a **Syteline** Data Lake integration. The M3 endpoint above is
confirmed; that M3 uses the same timings is the Syteline customer doc's claim, not independently
checked.

## Choosing a read path

| need | use |
|---|---|
| reporting-scale or long-running SQL-style read | `DATA_LAKE` |
| quick synchronous SQL-filtered read in a normal run | `EXPORTMI`/`Select` (`vince-exportmi-select-step`) |
| transactional read of a few records, or any write | the M3 API paths (`vince-native-vs-pipeline-decision`) |

## Not known

- Whether M3 uses the same poll interval and page size as the Syteline integration.
- What `saveResultToS3` does.
- Whether the step polls and pages by itself or the workflow must do it with further steps.
