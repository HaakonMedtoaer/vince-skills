---
name: vince-trigger-step
description: Use whenever building the entry point of a Vince Live workflow — a TRIGGER step with its form fields/buttons. Trigger on "how do I set up the trigger for this workflow", "what fields does the trigger step need", "how does the user kick off this app", or any question about workflow-trigger, triggerType, or the form a user fills to start a workflow.
---

# `TRIGGER` step

`type: TRIGGER`, `target: workflow-trigger`. Every workflow starts here.

## Confirmed config shape

`definition.stepConfig[stepId]`:
- `triggerType`
- `title`
- `buttonText`
- `fields[]`
- `formGroups[]`

## Why it matters downstream

Whatever the user submits through this step's `fields[]` becomes available to every later step as
`$context.data.trigger.body.<field>` in JSONata (see `vince-transform-step`), or as
`{{ header.Filter_A }}`-style double-brace references in a `GENERIC_FILTER` (see
`vince-generic-filter-step`). Getting the field names right here is load-bearing for the entire rest
of the workflow — every downstream reference to trigger data depends on matching these names exactly.

## Structural note

The step tree itself only carries `type`, `name`, `stepId`, `target`, `retryAttempts`, `childSteps` —
all the real configuration above lives in `definition.stepConfig`, keyed by `stepId`, separate from
the tree. This split holds for all ten confirmed step types, not just this one.
