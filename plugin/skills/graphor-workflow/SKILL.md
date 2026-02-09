---
name: graphor-workflow
description: >
  Graphor document intelligence workflow. Use when uploading documents, ingesting
  files/URLs/GitHub repos/YouTube videos, processing documents, asking questions
  about documents, extracting structured data from documents, retrieving chunks
  for RAG, or managing document sources in Graphor.
---

You have access to the Graphor MCP server for interacting with the Graphor API.

## How to use MCP tools vs direct API calls

**Use MCP tools** for all operations that don't involve local files: parse, ask, extract, retrieve chunks, list, delete. The MCP server handles authentication securely.

**For local file uploads**, the current MCP server uses a sandboxed execute container that cannot access the user's local filesystem. Use Bash with curl to upload local files directly. This is the documented workaround until the MCP server adds a dedicated file upload tool. For URL, GitHub, and YouTube uploads, use MCP tools.

## Prerequisites

The user must set their Graphor API key as a shell environment variable:

```bash
export GRAPHOR_API_KEY="grlm_your_api_key_here"
```

Add this to `~/.bashrc`, `~/.zshrc`, or equivalent. The `.mcp.json` uses `${GRAPHOR_API_KEY}` expansion to read this value. If MCP operations return auth errors, the key is not set correctly.

## Core workflow: Upload → Process → Query

This is a **three-step workflow**. All three steps are required:

1. **Upload** a source — returns status `"New"`
2. **Process** the source by calling parse via MCP — triggers async processing (`"New"` → `"Processing"` → `"Completed"`)
3. **Query** the source (ask, extract, or retrieve chunks) via MCP — only after status is `"Completed"`

**Upload alone does NOT start processing.** You must explicitly call parse after upload.

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

1. **Use MCP tools for all non-upload operations** — parse, ask, extract, list, delete all go through MCP.
2. **Use Bash curl for local file uploads** — the MCP execute container cannot access the local filesystem. See [upload-sources](rules/upload-sources.md).
3. **Always use `file_ids` over `file_names`** — `file_names` is deprecated. Store `file_id` from upload responses.
4. **Always process after upload** — upload returns `"New"`, you must call parse to start processing.
5. **Always check status is `"Completed"`** before querying documents.
6. **If you get auth errors**, the `GRAPHOR_API_KEY` environment variable is not set. Tell the user to run `export GRAPHOR_API_KEY="grlm_..."` and restart Claude Code.
