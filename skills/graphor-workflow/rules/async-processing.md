---
name: async-processing
description: Handle asynchronous document processing, status checking, and reprocessing.
metadata:
  tags: processing, async, status, parse, reprocess
---

# Async Processing

This is the most important workflow constraint in Graphor.

## The core rule

**Documents must reach `"Completed"` status before they can be queried.** Attempting to ask, extract, or retrieve chunks from a document that is still processing will return errors or empty results.

## Status transitions

```
Uploaded → Processing → Completed
                     → Failed
```

## Checking status

Use the list sources operation to check document status. Look for the `status` field on each source.

When a document is still processing:
1. **Inform the user** that the document is being processed
2. **Wait a reasonable interval** (10-30 seconds for small files, longer for large/complex ones)
3. **Check again** until status is `"Completed"` or `"Failed"`

## Reprocessing

If the default parsing produces poor results (garbled text, missing tables, wrong structure), reprocess with a different method:

- `basic` — Try this for clean text-only documents
- `hi_res` — Try this for scanned PDFs or documents with complex layouts
- `hi_res_ft` — Fine-tuned hi-res, better for specialized document types
- `mai` — Multi-modal AI, good for mixed content (text + images + tables)
- `graphorlm` — Best quality, use for important documents where accuracy matters most

Reprocessing is also async — the document goes back to `"Processing"` status.

Use `file_id` (preferred) or `file_name` to identify the document when reprocessing.

## Anti-patterns

- **Never skip status checking.** This is the #1 source of errors with Graphor.
- **Never poll in a tight loop.** Use reasonable intervals (10+ seconds).
- **Never assume upload means ready.** Upload only starts processing — it does not complete it.
- **Do not reprocess without reason.** The default parsing method works well for most documents. Only reprocess if results are unsatisfactory.
