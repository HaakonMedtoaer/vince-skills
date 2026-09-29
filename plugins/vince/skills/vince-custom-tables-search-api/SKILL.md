---
name: vince-custom-tables-search-api
description: Two confirmed bugs in the Vince Live Custom Tables search endpoint (api.vince.live/v1/custom-tables/search/<table>) that silently corrupt paged results, and the safe way to fetch everything. Use before writing any multi-page read of a Custom Table — in a script or a GENERIC_API step — or when asked why a queue read is missing rows or what the search size limit is.
---

# Custom Tables search API

`api.vince.live/v1/custom-tables/search/<table>` is the Elasticsearch-backed endpoint behind every
Custom Table search. Both bugs below were confirmed against a real tenant on 2026-08-31.

## The bugs

1. **`from` is off by one.** Its minimum is 1, but Elasticsearch counts from 0, so `from=1` silently
   skips the true first record of every query. Omitting `from` is the only way to get that record.
2. **Pages don't reassemble.** Climbing `from` across separate calls silently dropped or duplicated
   rows for some large queries (`Program:C*` reliably lost ~239 rows) but not others (`Program:M*`):
   the default sort order isn't stable between calls. Neither bug raises an error.

A documented `scrollId`/`scrollMinutes` mode looks like the intended fix but has never been verified
to work — don't rely on it.

## Safe pattern: never page

`size=2000` is the hard cap per request. Narrow the query until every request fits:

1. Query `q=Program:<prefix>*` with a one-letter prefix.
2. If the reported `total` is ≤ 2000, fetch it in one request.
3. Otherwise extend the prefix (`C` → `CA`, `CB`, …) and repeat until every leaf is ≤ 2000.
4. Concatenate the leaves, de-duplicate on the table's real primary key, and **warn on any duplicate**
   — it means the split overlapped and the result needs checking.

Only the one-letter form was spot-checked in Postman; multi-letter prefixes (`Program:CMS*`) are
assumed to behave the same.

Counts and data are tenant-specific: fetching for another tenant means rerunning this with that
tenant's token.

## Not known

- Whether `scrollId`/`scrollMinutes` works.
- Whether multi-letter prefixes behave like the single-letter form.
