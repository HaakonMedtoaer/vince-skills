---
name: vince-exportmi-select-step
description: The confirmed EXPORTMI/Select pattern — an ordinary GENERIC_API call to M3 that carries raw SQL in record.QERY and returns one separator-joined REPL string per row. Use when a Vince Live workflow needs a quick synchronous SQL-style read against one M3 table, when choosing between it and the async DATA_LAKE path, or when reviewing a Transform that splits a REPL string — that split is the signature of this pattern.
---

# `EXPORTMI`/`Select`

Not a step type: an ordinary `GENERIC_API` call to M3's REST gateway (`m3api-rest/v2/execute`) that
targets program `EXPORTMI`, transaction `Select`. See `vince-generic-api-step` for the step itself.

Evidence: seen in live customer solutions at Homewerks and Europris, implied at Berggård Amundsen — a
general M3 capability, not a one-off. `EXPORTMI/Select` is present in the Vince tenant's API catalog.

## Request

`record.QERY` carries a raw SQL fragment as a string, e.g.

```
"OACUNO, OAORNO … from OOHEAD where OACONO = 100 and …"
```

It runs **synchronously** — no job, no polling. That is the difference from `DATA_LAKE` (async
submit/poll/paginate; see `vince-data-lake-step`). Don't conflate the two.

## Response — the part that trips people up

Not JSON records. Each row comes back as **one `REPL` string**, columns joined by the separator you
passed in `SEPC` (e.g. `"§"`). With `HDRS: "1"` the first row is the header.

Every production example unpacks it with the same `$split`/`$map`/`$merge` idiom in the next
Transform — reuse it rather than inventing a new one. An illustrative skeleton (match the path, the
separator and the column positions to your own call and `QERY` select list):

```
(
  $rows := $context.data.all.rest_api_1.body.results.records#$i[$i > 0];  /* skip header when HDRS = "1" */
  $rows.(
    $c := $split(REPL, "§");
    { "custNo": $c[0], "orderNo": $c[1] }   /* by position in the QERY select list */
  )
)
```

## When to use it

- A fast, SQL-filtered read against one M3 table, inside a normal synchronous workflow run.
- Reporting-scale volume or long-running queries → `vince-data-lake-step` instead.
- Transactional reads of a few records, or anything that writes → the normal M3 API paths.

## Not known

- The full request record beyond `QERY`, `SEPC` and `HDRS`.
- Row or size limits, and whether `QERY` accepts joins.
