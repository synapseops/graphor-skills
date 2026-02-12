---
name: graphor-workflow
description: Graphor document intelligence workflow. Use when uploading documents, ingesting files/URLs/GitHub repos/YouTube videos, processing documents, asking questions about documents, extracting structured data from documents, retrieving chunks for RAG, or managing document sources in Graphor.
---

You have access to the Graphor MCP server for interacting with the Graphor API.

## Prerequisites

The user must set their Graphor API key as a shell environment variable:

```bash
export GRAPHOR_API_KEY="grlm_your_api_key_here"
```

Add this to `~/.bashrc`, `~/.zshrc`, or equivalent. The `.mcp.json` uses `${GRAPHOR_API_KEY}` expansion to read this value. If MCP operations return auth errors, the key is not set correctly.

## Core workflow: Upload → Process → Query

1. **Upload** a source — returns status `"New"`
2. **Process** the source by calling parse via MCP — only required if `partition_method` was not set during upload (`"New"` → `"Processed"`)
3. **Query** the source (ask, extract, or retrieve chunks) via MCP

## How to use

Load the relevant rule for the task at hand:

| Task | Rule |
|------|------|
| Upload files, URLs, GitHub repos, or YouTube videos | [upload-sources](rules/upload-sources.md) |
| Process documents and check status | [parsing](rules/parsing.md) |
| Ask questions about documents (conversational Q&A) | [ask-sources](rules/ask-sources.md) |
| Extract structured data using JSON Schema | [extraction](rules/extraction.md) |
| Retrieve chunks for custom RAG pipelines | [retrieve-chunks](rules/retrieve-chunks.md) |
| List, inspect, or delete documents | [manage-sources](rules/manage-sources.md) |

## Universal rules

1. **Use MCP tools for all operations**.
2. **Always use `file_ids` over `file_names`** — `file_names` is deprecated. Store `file_id` from upload responses.
3. **Process after upload if you did not set `partition_method`** — upload returns `"New"`, you must call parse to process the document.
4. **If you get auth errors**, the `GRAPHOR_API_KEY` environment variable is not set. Tell the user to run `export GRAPHOR_API_KEY="grlm_..."` and restart Claude Code.
