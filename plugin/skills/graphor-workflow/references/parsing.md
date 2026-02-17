---
name: parsing
description: Process documents after upload, check status, and reprocess if needed.
metadata:
  tags: processing, status, parse, reprocess, new
---

# Processing Documents

Use the Graphor MCP tools for processing. Never make direct API calls.

**This step is only required if you did not set a `partition_method` during upload.** If you set `partition_method` at upload time, processing already happened — skip this step entirely and go directly to query (ask, extract, or retrieve chunks).

## The workflow

```
1. Upload (without partition_method)  →  status: "New"       (source accepted, NOT processed)
2. Parse                              →  status: "Processed" (ready to query)
```

Both calls are **synchronous** — parsing is performed within the parse call itself. There is no need to poll or wait for status changes.

## Step 1: After upload — status is "New"

When you upload a source without setting `partition_method`, the response returns status `"New"`. This means:
- The source was accepted and stored
- It has **NOT** been processed yet
- It **cannot** be queried yet

**"New" is not an error.** It is the expected initial status.

## Step 2: Call parse to process

Call the parse operation using the `file_id` from the upload response.

Specify a `partition_method` (ordered from lowest to highest accuracy):
- `basic` — Fast - Text-only extraction (least accurate)
- `hi_res` — Balanced - High-resolution OCR for scanned documents
- `hi_res_ft` — Accurate - Hi-res with fine-tuned models
- `mai` — VLM - Multi-modal AI parsing
- `graphorlm` — Agentic - Graphor's proprietary pipeline (most accurate, slowest)

If you don't specify a method, Graphor chooses the simplest method depending on the file type.

When the parse call returns, the status will be `"Processed"` — the document is ready to query.

## Reprocessing

If results are unsatisfactory (garbled text, missing tables, wrong structure), call parse again with a different `partition_method`.

Use `file_id` to identify the document. `file_name` is deprecated as an identifier.

## Anti-patterns

- **Do not skip the parse step if you did not set `partition_method` during upload.** Status will stay `"New"` forever otherwise.
- **Do not call parse if you DID set `partition_method` during upload.** Processing already completed at upload time. Calling parse again is redundant and wastes time. Go directly to query.
- **Do not treat `"New"` as an error or loop on it.** It is the expected status after upload without `partition_method`. Call parse to advance.
- **Do not rely on the `status` field from list_sources to decide whether to parse.** If you set `partition_method` during upload, the document is processed regardless of what status shows. The status field may lag behind actual state.
- **Do not reprocess without reason.** Only reprocess if results are unsatisfactory.
- **Use MCP tools.**
