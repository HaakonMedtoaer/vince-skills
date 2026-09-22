---
name: vince-generic-api-step
description: Confirmed JSON shape and every known content field for the GENERIC_API workflow step (target workflow-rest-api) — the Transform/REST pipeline used to call M3's REST gateway, Vince Live's own API (custom tables, triggering another workflow), or any external REST endpoint. Use whenever drafting or reviewing a GENERIC_API / REST API step, wiring a workflow to call M3, chaining workflows together, reading or writing Custom Table rows via REST, or debugging why a REST step's output can't be found in the next Transform.
---

## What GENERIC_API is

`type: "GENERIC_API"`, `target: "workflow-rest-api"`. This is the general-purpose "make an HTTP call" step. `content` is a **JSON string** (not a nested object — it's serialized). It is used for three genuinely different things, all through the same step type:

1. Calling M3's own REST gateway: `/infor/M3/m3api-rest/v2/execute`
2. Calling Vince Live's **own** REST API — `api.vince.live/v1/custom-tables/...` (read/write/delete Custom Table rows directly) and `api.vince.live/live/workflows/WORKFLOW-<id>/sync` (trigger another workflow — this is how workflow chaining works; it is not a separate step type)
3. Any other external REST endpoint the tenant needs

## Confirmed `content` fields

- `connectionId`
- `endpoint` — can be a literal string, **or** a Handlebars template pulling from an earlier step, e.g. `{{ body.ionUrl }}` or `{{ $context.data.all.transform_1.ionUrl }}` (confirmed in Homewerks' auto-release workflow). This lets a workflow switch M3 environments by editing one Transform instead of every REST step.
- `method`
- `headers`
- `bodyPath` — `"$"` in the confirmed M3-gateway example
- `version: 2` — seen alongside the above in 10009 - Voice's order-lines and claim-register workflows. Not confirmed whether `version` is always present, or what other values it can take.
- `forEach` — e.g. `"forEach": "body"` makes the call iterate once per item of the named array, instead of firing once with the whole payload (confirmed in Europris's invoice-reminder solution).
- `skipIfExpression` — a JSONata boolean string (e.g. `"$contains(body.mail, 'Ingen Epost Funnet')"`); when true, the call is skipped for that item.

**Caveat on `forEach`/`skipIfExpression`**: only the field *names* are independently confirmed (via the peer `vince-live-workflow` skill). Their actual runtime behavior is the customer's own claim about their workflow (Europris's documentation), not something watched executing — the vendor's own doc is also inconsistent about the `version` number involved. Treat the *existence* of these fields as solid, their precise behavior as customer-claimed, not platform-verified.

## The sandwich pattern

M3 is called through this step as a sandwich: a Transform builds the request body → `GENERIC_API` sends it → another Transform reads the response back. Never treat `GENERIC_API` as calling M3 directly with no Transform on either side — the real workflow this project has read always wraps it this way.

## The one gotcha that will break your JSONata: `body.body`

Every other step's output is read by the next Transform as `$context.data.all.<stepId>.body...`. **`GENERIC_API` nests its payload one level deeper** — the platform's own default transform template documents this as a stated context rule. So a REST step named `rest_api_1` is read as:

```
$context.data.all.rest_api_1.body.body.results.records
```

(the real workflow this project captured reads exactly `…rest_api_1.body.results.records` relative to that extra `body` nesting — don't drop it when writing the next Transform's JSONata).

## Batching M3 transactions: two real, equally valid patterns

**Fan-shaped batching**: a JSONata `$map` in the preceding Transform emits `{program, maxReturnedRecords, transactions: [...]}`, carrying N M3 transactions in one `GENERIC_API` call. This is real and confirmed (`operations-workflow.txt`) — anything fan-shaped like this is one batched call, never N separate steps, and never a loop (there is no loop step type).

**Per-transaction pairs**: splitting one transaction per Transform→REST pair — instead of one `$map` emitting several — is also a real, deliberate pattern (confirmed via 10009 - Voice's documentation), chosen specifically because it makes mapping the output easier. **Neither pattern is "the" rule** — pick based on whether per-transaction output mapping or call-count matters more for the workflow you're drafting.

## Calling Vince Live's own API through this same step

- Custom Tables: `api.vince.live/v1/custom-tables/...` for reading/writing/deleting rows directly — see `vince-table-updater-step` for the queue-pattern usage of this (read side, up to 100 rows at a time via search, delete-as-processed).
- Triggering another workflow: `POST https://api.vince.live/live/workflows/WORKFLOW-<id>/sync` with the next workflow's payload as the body — confirmed via Europris's three-chained-workflow invoice-reminder solution. This resolved what was once described as "trigger another workflow" needing its own step type — it doesn't; it's this step pointed at Vince Live's own gateway instead of M3's.

## Related skills

`vince-transform-step` for the JSONata on either side of the sandwich, `vince-m3-native-api-step` for the alternative that skips the sandwich entirely, `vince-native-vs-pipeline-decision` for when to pick this path over native, `vince-exportmi-select-step` for a specific confirmed M3-gateway usage pattern (bulk SQL-style reads via `EXPORTMI`/`Select`).
