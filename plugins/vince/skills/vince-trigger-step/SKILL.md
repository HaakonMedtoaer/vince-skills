---
name: vince-trigger-step
description: Confirmed JSON shape of the Vince Live TRIGGER workflow step (target workflow-trigger) — the entry point and the form a user fills in to start a workflow. Use when building a workflow's entry point, when asked "what fields does the trigger need" or "how does the user start this app", or for any question about triggerType or trigger form fields.
---

# `TRIGGER` step

`type: "TRIGGER"`, `target: "workflow-trigger"`. Every workflow starts here. Shape captured from real
tenant workflows.

## Shape

`definition.stepConfig[stepId]`: `triggerType`, `title`, `buttonText`, `fields[]`, `formGroups[]`.

The step tree itself carries only `type`, `name`, `stepId`, `target`, `retryAttempts`, `childSteps`;
all configuration lives in `stepConfig`, keyed by `stepId` — true of every step type.

## Why the field names matter

What the user submits becomes `$context.data.trigger.body.<field>` in JSONata
(`vince-transform-step`) and `{{ header.<field> }}`-style references in a `GENERIC_FILTER`
(`vince-generic-filter-step`). Every downstream reference depends on these names matching exactly.

## From Vince's product documentation (claims, not captures)

- Triggers are Manual, Scheduled, or Events & Webhooks.
- **Scheduled and event triggers can't take manual input** — the workflow must carry defaults for
  everything it needs.

## Not known

- The full set of `triggerType` values and their JSON names.
- The shape of an entry in `fields[]` and `formGroups[]`.
- Event trigger conditions — the docs describe them only by example.
