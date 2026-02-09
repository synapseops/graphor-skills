---
name: graphor-workflow
description: >
  Graphor document intelligence workflow. Use when uploading documents, ingesting
  files/URLs/GitHub repos/YouTube videos, processing documents, asking questions
  about documents, extracting structured data from documents, retrieving chunks
  for RAG, or managing document sources in Graphor.
---

You have access to the Graphor MCP server. **Always use MCP tools for all Graphor operations.** Never make direct API calls via curl, fetch, or any HTTP client — the MCP server handles authentication and requests securely without exposing credentials.

## Mandatory: Use MCP tools only

- **DO**: Use the Graphor MCP tools provided by the `graphor` MCP server
- **DO NOT**: Make curl commands, use fetch/axios, or call the Graphor REST API directly
- **DO NOT**: Reference or use API URLs, Bearer tokens, or auth headers in Bash commands
- **WHY**: Direct API calls expose the user's API key to the conversation. The MCP server keeps credentials local and secure.

## Core workflow: Upload → Process → Query

This is a **three-step workflow**. All three steps are required:

1. **Upload** a source (file, URL, GitHub, YouTube) — returns status `"New"`
2. **Process** the source by calling parse — triggers async processing (`"New"` → `"Processing"` → `"Completed"`)
3. **Query** the source (ask, extract, or retrieve chunks) — only after status is `"Completed"`

**Upload alone does NOT start processing.** You must explicitly call the process/parse operation after upload.

See [async-processing](rules/async-processing.md) for the full status flow.

## How to use

Load the relevant rule for the task at hand:

| Task | Rule |
|------|------|
| Upload files, URLs, GitHub repos, or YouTube videos | [upload-sources](rules/upload-sources.md) |
| Process documents and check status | [async-processing](rules/async-processing.md) |
| Ask questions about documents (conversational Q&A) | [ask-sources](rules/ask-sources.md) |
| Extract structured data using JSON Schema | [extraction](rules/extraction.md) |
| Retrieve chunks for custom RAG pipelines | [retrieve-chunks](rules/retrieve-chunks.md) |
| List, inspect, or delete documents | [manage-sources](rules/manage-sources.md) |

## Universal rules

1. **Always use MCP tools** — never curl or direct HTTP calls to Graphor API.
2. **Always use `file_ids` over `file_names`** — `file_names` is deprecated. Store `file_id` from upload responses.
3. **Always process after upload** — upload returns `"New"`, you must call parse to start processing.
4. **Always check status is `"Completed"`** before querying documents.
5. **Handle errors gracefully** — 401 means bad API key, 404 means file not found or not yet processed, 429 means back off.
