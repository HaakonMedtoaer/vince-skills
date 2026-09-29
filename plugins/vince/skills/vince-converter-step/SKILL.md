---
name: vince-converter-step
description: The confirmed but thin shape of the Vince Live CONVERTER workflow step (target workflow-converter) for converting between xml, json and csv. Use when a workflow must convert a file or payload between those formats, or when asked "would this CONVERTER JSON work" — and to keep invented delimiter or encoding fields out of the design.
---

# `CONVERTER` step

`type: "CONVERTER"`, `target: "workflow-converter"`. Seen in a captured reference workflow.

## What's known

`definition.stepConfig[stepId]` holds a source format and a target format — each xml, json or csv —
plus delimiter configuration. **The exact field names for these haven't been recorded**, only that
the three concepts exist. Vince's product documentation mentions JSON-to-CSV and CSV-to-JSON.

## Don't round it out

Quote/escape characters, encodings, formats beyond xml/json/csv, or specific delimiter field names are
all unconfirmed. If a design needs them, flag them as needing a real capture rather than writing
plausible JSON.
