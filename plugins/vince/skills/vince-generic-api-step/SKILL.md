---
name: vince-generic-api-step
description: Confirmed JSON shape and content fields of the Vince Live GENERIC_API workflow step (target workflow-rest-api) — the REST step used to call M3's REST gateway, Vince Live's own API (Custom Tables, triggering another workflow) or any external endpoint. Use when drafting or reviewing a REST API step, wiring a workflow to call M3 through a Transform/REST pipeline, chaining workflows, reading or writing Custom Table rows over REST, or debugging why a REST step's output can't be found in the next Transform.
---

# `GENERIC_API` step

`type: "GENERIC_API"`, `target: "workflow-rest-api"`. The general-purpose HTTP call. Its
`definition.stepConfig[stepId].content` is a **JSON string** — serialized, not a nested object.

Evidence: shape captured from real tenant workflows; extra fields from live customer solutions and
VinceForge's captured reference (`anthropic-skills:vince-live-workflow`).

## `content` fields

| field | status | notes |
|---|---|---|
| `connectionId` | confirmed | the saved Connection that supplies auth |
| `endpoint` | confirmed | literal URL, **or** a template from an earlier step: `"{{ body.ionUrl }}"`, `"{{ $context.data.all.transform_1.ionUrl }}"` (Homewerks) — lets one Transform switch M3 environments |
| `method` | confirmed | |
| `headers` | confirmed | |
| `bodyPath` | confirmed | `"$"` in the M3-gateway calls |
| `version` | seen | `2` on some steps (10009 - Voice). Unknown whether always present or what other values exist. Product docs call v2 beta and say its output becomes an array of `{headers, body}` |
| `forEach` | name confirmed | e.g. `"body"`. Described (Europris) as calling once per item of that array — not observed executing |
| `skipIfExpression` | name confirmed | JSONata boolean, e.g. `"$contains(body.mail, 'Ingen Epost Funnet')"`. Described as skipping an item when true — not observed executing |

Don't add fields beyond these without a real capture.

## Three uses, one step type

1. **M3's REST gateway** — `/infor/M3/m3api-rest/v2/execute`.
2. **Vince Live's own API** — `api.vince.live/v1/custom-tables/...` to read, write or delete Custom
   Table rows; `api.vince.live/live/workflows/WORKFLOW-<id>/sync` to trigger another workflow (how
   Europris chains three workflows). Workflow chaining has no step type of its own.
3. **Any external REST endpoint.**

## Calling M3: the sandwich

A Transform builds the request → `GENERIC_API` posts it to the gateway → a Transform reads the result.
Two batching shapes are both real:

- **One call, N transactions:** a `$map` in the Transform emits
  `{program, maxReturnedRecords, transactions: [...]}` (captured).
- **One transaction per Transform→REST pair:** chosen by 10009 - Voice because the output is easier to
  map.

There is no loop step. Fan-out is a batched call or separate pairs, never N generated steps.

## Reading its output

In the next Transform, read the REST step by its `stepId` exactly as the captured workflow does:

```
$context.data.all.rest_api_1.body.results.records
```

The platform's default transform template notes that the **shorthand** `body` (meaning "the previous
step") nests one level deeper after a REST step — `body.body`. That applies to the shorthand only,
not to the `$context.data.all.<stepId>` form above.

## Related

`vince-transform-step` (JSONata on both sides), `vince-m3-native-api-step` (the path without the
sandwich), `vince-native-vs-pipeline-decision`, `vince-exportmi-select-step` (SQL-style reads through
this step), `vince-custom-tables-search-api` (before paging a Custom Table over REST).

## Not known

- Whether `version` is always present, and what values besides `2` exist.
- What `forEach` and `skipIfExpression` do at run time — only their names are confirmed.
- The exact output shape of a `version: 2` step (product docs say an array of `{headers, body}`; not captured).
