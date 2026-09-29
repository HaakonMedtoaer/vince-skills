---
name: vince-native-vs-pipeline-decision
description: The decision rule for implementing an M3 read or write in a Vince Live workflow — the native API step versus the Transform → GENERIC_API → Transform pipeline. Use whenever designing a new M3-touching step, when asked "should this be an API step or GENERIC_API" or "why do real customer workflows use Transform+REST", or when reviewing a design that picked one path without saying why.
---

# Native `API` step vs. the `GENERIC_API` pipeline

Both are real, confirmed ways to call M3:

- **Native** — an `API` step (`m3_api_1`), optionally with a `GENERIC_FILTER` before it and an `EXCEL`
  after it. See `vince-m3-native-api-step`.
- **Pipeline** — Transform builds the request → `GENERIC_API` posts it to M3's REST gateway →
  Transform reads the result. See `vince-generic-api-step`.

Every live customer workflow reviewed so far (Homewerks, Europris, 10009 - Voice, and a captured
production workflow) uses the pipeline. That's a consequence of their requirements, not a rule —
decide per workflow.

## Ask in order

1. **Must it filter on a value an earlier M3 call returned in this run?** → pipeline. `GENERIC_FILTER`
   sees only raw trigger rows and runs once, before the whole M3 step.
2. **Must it gate on a numeric or date comparison (`>`, `<`, before/after)?** → pipeline, with the
   comparison in a JSONata Transform.
3. **Must it skip or vary individual calls per row?** → pipeline, e.g. `skipIfExpression` on the REST
   step — the field name is confirmed; its behaviour is the customer's description, so test it if it's
   load-bearing.
4. **None of the above?** → prefer native: automatic pagination, stricter save-time validation, and
   per-field metadata that catches wrong field names before the workflow runs. It's also less JSON to
   write by hand. Writing results to a spreadsheet (`EXCEL`) requires native.

Rules 1–2 come from VinceForge's captured reference (`anthropic-skills:vince-live-workflow`).

## An unresolved contradiction — flag it, don't pick a side

Vince's product documentation describes an **"M3 Filter"** component that filters rows *returned by*
an M3 API step, with operators including less-than and greater-than, and it also lists `<`/`>` for
Generic Filter. That contradicts parts of rules 1–2. No JSON capture or `target` for M3 Filter has
been seen. Until one is, design by the rule above and note the contradiction in the design.

## Not known

- The JSON shape and `target` of the documented M3 Filter component, and whether it removes the limits in rules 1–2.
