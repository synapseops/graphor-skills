---
name: extraction
description: Extract structured data from documents using JSON Schema.
metadata:
  tags: extract, structured, schema, json, data
---

# Structured Extraction

Use the Graphor MCP tools for extraction. Never make direct API calls.

Extract typed, structured data from documents using JSON Schema definitions.

**Prerequisite**: Documents must have status `"Completed"` before extraction. If status is `"New"`, you must call parse first.

## Core usage

Three required inputs:
1. **Documents** — `file_ids` (preferred) or `file_names` to extract from
2. **Instruction** — `user_instruction` string guiding what to extract ("Extract invoice line items", "Pull out all person names and roles")
3. **Schema** — `output_schema` defining the expected JSON structure

The response includes:
- `structured_output` — The extracted data, validated against your schema
- `raw_json` — The model's raw output before validation (useful for debugging)

## Schema rules

Graphor uses **simplified JSON Schema only**:

- Supports: `object`, `array`, `string`, `number`, `boolean`, `null`
- Supports: `properties`, `items`, `required`, `type`
- Supports: nullable types via `["string", "null"]`

**Forbidden schema constructs:**
- `oneOf`, `anyOf`, `allOf` — not supported
- `$ref` — no schema references
- Complex union types — only `["type", "null"]` is allowed
- `enum` with complex values

## Thinking levels

Same as ask-sources: `fast`, `balanced`, `accurate` (default). Use `accurate` for extraction — precision matters more than speed here.

## Anti-patterns

- **Do not use deeply nested schemas.** Flat or shallow structures produce more reliable results. If you need deep nesting, consider multiple extraction passes.
- **Do not use `oneOf`, `anyOf`, or `$ref`.** They will cause 400 errors. Flatten your schema instead.
- **Do not skip `user_instruction`.** The instruction is critical for guiding the model. A schema alone is ambiguous — the instruction tells the model what to look for and how to interpret it.
- **Do not extract from documents in `"New"` or `"Processing"` status.** Always verify `"Completed"` status first. If `"New"`, call parse first.
- **Do not use `file_names` for identification.** Use `file_ids` — `file_names` is deprecated.
- **Use MCP tools for extraction** — not curl. Only local file upload requires curl.
