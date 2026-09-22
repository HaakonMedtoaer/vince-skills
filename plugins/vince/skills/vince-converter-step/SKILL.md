---
name: vince-converter-step
description: The confirmed (thin) JSON shape of the Vince Live CONVERTER workflow step (target workflow-converter) for xml/json/csv format conversion. Use when a brief needs a workflow to convert a file or payload between xml, json, and csv, or when someone asks "would this CONVERTER JSON work" — and to be reminded this step type is thinly confirmed, so don't invent delimiter or encoding fields beyond what's known.
---

# CONVERTER step

`type: "CONVERTER"`, `target: "workflow-converter"`.

## Confirmed config shape

`definition.stepConfig[stepId]` holds a `source`/`target` format pair, chosen from xml/json/csv, plus delimiter configuration. This is deliberately imprecise because the exact field names for `source`, `target`, and the delimiter config have **not** been captured in detail in this project's confirmed facts — only that these three concepts (source format, target format, delimiter config) exist in the step.

## Don't over-fill this one

This is one of the thinner-confirmed step types. Resist the temptation to round it out with plausible-sounding fields — a specific delimiter character field name, an encoding option, quote/escape character handling, or support for formats beyond xml/json/csv — unless you've seen them in a real capture. If a brief needs conversion behavior beyond the three known formats or needs precise field names, flag that as an open question requiring a real workflow capture, rather than presenting a guess as confirmed.

## When to reach for this skill

- A brief needs format conversion between xml/json/csv inside a workflow.
- Reviewing a draft `CONVERTER` step — check that its format choices are within xml/json/csv, and flag (don't silently accept) any field name in the step config that isn't independently confirmed elsewhere in this project's records.
