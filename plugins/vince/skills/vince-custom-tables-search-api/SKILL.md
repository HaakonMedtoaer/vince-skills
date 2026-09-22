---
name: vince-custom-tables-search-api
description: Use whenever a Vince Live workflow or script reads from a Custom Table via the search endpoint (api.vince.live/v1/custom-tables/search/<table>) — e.g. "how do I page through this Custom Table", "why is my TABLE_UPDATER queue read missing rows", "what's the size limit on custom-tables/search", or any GENERIC_API step that queries a Custom Table. This endpoint has two confirmed production bugs around pagination that silently corrupt results — consult this before writing any multi-page fetch against it.
---

# Vince Live Custom Tables search API

Confirmed empirically against a real tenant on **2026-08-31** — not inferred from docs. This is the
same generic, Elasticsearch-backed endpoint every Custom Table read goes through, including the read
side of the `TABLE_UPDATER` queue pattern (see `vince-table-updater-step`) and how
`m3-api-catalog-full.json` itself was built.

## The two confirmed bugs

**1. `from` has a minimum of 1, but Elasticsearch's real `from` is 0-indexed.**
Passing `from=1` silently skips the true first record of *every* query — there is no error, the
response just quietly starts one record late. Omitting `from` entirely is the only confirmed way to
get the real first record.

**2. Multi-page fetches cannot be reassembled client-side.**
Climbing `from` across separate HTTP calls silently dropped or duplicated rows for some large queries
(`Program:C*` reliably lost ~239 rows) but not others (`Program:M*`, also large, never did) — the
default sort order isn't stable across calls, so two calls at different `from` offsets aren't
guaranteed to see a consistent ordering. This is not a rare edge case; it's the default behavior.

A documented `scrollId`/`scrollMinutes` mode exists on the same endpoint and looks like the intended
tool for this. **It has never been verified to work.** Do not assume it does just because it's
documented — nobody has watched it return correct results here.

## The safe pattern: never page

`size=2000` is the confirmed hard cap per request. Instead of paging, narrow the query itself:

1. Query with `q=Program:<prefix>*` for a single-character prefix (e.g. `C`).
2. If the reported `total` is ≤ 2000, that's a safe single fetch — take it.
3. If `total` > 2000, recursively extend the prefix (`C` → `CA`, `CB`, `CC`, …) and repeat, until every
   leaf's reported `total` is ≤ 2000.
4. Concatenate all leaves, de-duplicate on the table's real primary key, and **warn on any duplicate
   found post-fetch** — a duplicate means the prefix split still overlapped somewhere and the result
   should not be trusted blindly.

This guarantees every actual fetch is a single, un-paged request, which is the only confirmed-safe
mode of this endpoint.

**Caveat on the workaround itself:** the single-letter prefix form (`Program:C*`) was independently
spot-checked against Postman. The multi-character form (`Program:CMS*`) was never independently
checked the same way — it's assumed to behave identically, not confirmed.

## Practical implication

If you're about to write code or a workflow step that pages through a Custom Table by incrementing
`from`, stop — that is the exact failure mode this skill exists to prevent. Rewrite it as a
prefix-narrowing single-request fetch instead.

Row counts and the catalog itself are tenant-specific. Rebuilding a catalog or fetching a different
Custom Table for a different tenant means rerunning this same safe-fetch pattern against that
tenant's token — it is not something that can be assumed stable across tenants.
