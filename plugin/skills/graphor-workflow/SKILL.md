---
name: graphor-workflow
description: >
  Graphor document intelligence workflow. Use when uploading documents, ingesting
  files/URLs/GitHub repos/YouTube videos, processing documents, asking questions
  about documents, extracting structured data from documents, retrieving chunks
  for RAG, or managing document sources in Graphor.
---

You have access to the Graphor MCP server. Use it to upload, process, and query documents on behalf of the user.

## When to use

Use this skill whenever interacting with Graphor — uploading files, checking processing status, asking questions about documents, extracting structured data, or building RAG pipelines.

## Critical constraint

**Document processing is asynchronous.** After any upload, the document must reach `"Completed"` status before it can be queried. Never attempt to ask, extract, or retrieve chunks from a document that is still processing. See [async-processing](rules/async-processing.md).

## How to use

Load the relevant rule for the task at hand:

| Task | Rule |
|------|------|
| Upload files, URLs, GitHub repos, or YouTube videos | [upload-sources](rules/upload-sources.md) |
| Wait for processing, check status, reprocess documents | [async-processing](rules/async-processing.md) |
| Ask questions about documents (conversational Q&A) | [ask-sources](rules/ask-sources.md) |
| Extract structured data using JSON Schema | [extraction](rules/extraction.md) |
| Retrieve chunks for custom RAG pipelines | [retrieve-chunks](rules/retrieve-chunks.md) |
| List, inspect, or delete documents | [manage-sources](rules/manage-sources.md) |

For full API endpoint details, see [references/api-reference.md](references/api-reference.md).

## Universal rules

These apply to ALL Graphor operations:

1. **Always use `file_ids` over `file_names`** — `file_names` is deprecated across the entire API. Store and pass `file_id` from upload responses.
2. **Always check processing status** before querying newly uploaded documents.
3. **Handle errors gracefully** — 401 means bad API key, 404 means file not found or not yet processed, 429 means back off.
