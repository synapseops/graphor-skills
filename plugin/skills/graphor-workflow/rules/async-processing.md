---
name: async-processing
description: Process documents after upload, check status, and reprocess if needed.
metadata:
  tags: processing, async, status, parse, reprocess, new
---

# Processing Documents

This is the most critical workflow step. **Upload does NOT process documents.** You must explicitly trigger processing.

## The three-step workflow

```
1. Upload  →  status: "New"        (source accepted, NOT processed)
2. Parse   →  status: "Processing" (processing started)
3. Wait    →  status: "Completed"  (ready to query)
                    or "Failed"    (processing error)
```

## Step 1: After upload — status is "New"

When you upload a source (file, URL, GitHub, YouTube), the response returns status `"New"`. This means:
- The source was accepted and stored
- It has **NOT** been processed yet
- It **cannot** be queried yet

**"New" is not an error.** It is the expected initial status.

## Step 2: Call parse to start processing

After upload, you must call the parse/process operation using the `file_id` (preferred) or `file_name` from the upload response.

Optionally specify a `partition_method`:
- `basic` — Fast, text-only extraction
- `hi_res` — High-resolution OCR for scanned documents
- `hi_res_ft` — Fine-tuned hi-res, better for specialized document types
- `mai` — Multi-modal AI, good for mixed content (text + images + tables)
- `graphorlm` — Best quality, use for important documents where accuracy matters most

If you don't specify a method, Graphor chooses automatically.

After calling parse, the status transitions to `"Processing"`.

## Step 3: Wait for "Completed"

Processing is **asynchronous**. After calling parse:
1. **Inform the user** that the document is being processed
2. **Wait a reasonable interval** (10-30 seconds for small files, longer for large/complex ones)
3. **Check status** via list sources — look for the `status` field
4. If `"Completed"` — the document is ready to query
5. If `"Processing"` — wait and check again
6. If `"Failed"` — inform the user, suggest trying a different partition method

## Reprocessing

If results are unsatisfactory (garbled text, missing tables, wrong structure), call parse again with a different `partition_method`. The document goes back to `"Processing"` status.

Use `file_id` (preferred) or `file_name` to identify the document.

## Anti-patterns

- **Do not skip the parse step.** Upload alone does NOT process documents. Status will stay `"New"` forever.
- **Do not treat `"New"` as an error or loop on it.** It is the expected status after upload. Call parse to advance.
- **Do not poll in a tight loop.** Use reasonable intervals (10+ seconds between checks).
- **Do not query documents in `"New"` or `"Processing"` status.** Wait for `"Completed"`.
- **Do not reprocess without reason.** The default parsing method works well for most documents. Only reprocess if results are unsatisfactory.
- **Do not make direct API calls.** Always use MCP tools.
