---
name: vince-exportmi-select-step
description: The confirmed EXPORTMI/Select pattern — an ordinary GENERIC_API call to M3 that carries raw SQL as a parameter and returns a specially-encoded string response. Use whenever a brief needs a quick, synchronous SQL-style read against an M3 table, whenever someone asks "how do I run a query against M3 without the async Data Lake path," or whenever reviewing a Transform step that re-parses a REPL/separator-joined string — that's the signature of this pattern.
---

# EXPORTMI/Select pattern

This is **not a distinct step type**. It's an ordinary `GENERIC_API` call (same `m3api-rest/v2/execute` gateway as any other M3 program/transaction call — see `vince-generic-api-step` for the general shape) targeting the M3 program/transaction `EXPORTMI`/`Select`.

## The confirmed shape

The request's `record.QERY` field carries a raw SQL fragment as a string, e.g.:

```
"OACUNO, OAORNO ... from OOHEAD where OACONO = 100 and ..."
```

This runs **synchronously** against an M3 table — no submit/poll/paginate cycle, unlike `DATA_LAKE` (see `vince-data-lake-step` — these are two distinct confirmed bulk-read paths, do not conflate them: `DATA_LAKE` is async; this is a normal synchronous API call that happens to carry SQL as a parameter).

Confirmed independently across three unrelated customers (Homewerks, Europris, and implied by Berggård Amundsen) — strong evidence this is a general M3 capability, not a one-off integration trick.

## The response shape is the gotcha

The response is **not** normal JSON records. It comes back as one `REPL` string per row, with columns joined by a caller-chosen separator field (`SEPC`, e.g. `"§"`), and a header row first when `HDRS: "1"` is set.

Every production example seen re-parses this with the identical pattern in the following `TRANSFORMER_MORPH` step: `$split(...)` on the separator, then `$map(...)` to turn each split row into a proper object. This is common enough across unrelated customers that it reads as a semi-standard, copy-pasted idiom at Vince rather than something invented per customer — expect to see it again, and reach for the same shape rather than inventing a new one.

A skeleton of the idiom (JSONata, illustrative — match field names/positions to the real query and `SEPC`/`HDRS` values in use):

```
$rows := $split(body.body.results.records[0].REPL, "§");
$header := $rows[0]; /* only if HDRS was "1" */
$data := $rows[[1..$count($rows)-1]]; /* skip header row if present */
$data ~> $map(function($row){
  $cols := $split($row, "§");
  { "custNo": $cols[0], "orderNo": $cols[1] /* map by position per the QERY select list */ }
})
```

Treat this as illustrative structure, not a copy-paste-ready snippet — the actual column positions depend entirely on the `QERY` select list used in that specific call.

## When to reach for this skill

- A brief needs a fast, synchronous, SQL-filtered read against one M3 table — this is often lighter-weight than `DATA_LAKE` for that.
- Reviewing a Transform step that splits a string on a separator right after a `GENERIC_API` call — recognize this as the expected pattern, not a red flag.
- Don't invent a different response shape (e.g. assuming normal JSON rows) for an `EXPORTMI`/`Select` call — the `REPL`/`SEPC`/`HDRS` shape is what's confirmed.
