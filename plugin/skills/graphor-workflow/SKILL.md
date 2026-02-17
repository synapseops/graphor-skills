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

## Core workflow

**Two paths depending on whether you set `partition_method` during upload:**

**Path A — `partition_method` set at upload time:**
1. **Upload** (with `partition_method`) — processing happens automatically
2. **Query** directly (ask, extract, or retrieve chunks) — do NOT call parse

**Path B — `partition_method` not set at upload time:**
1. **Upload** (without `partition_method`) — status `"New"`, not yet processed
2. **Process** by calling parse via MCP — status becomes `"Processed"`
3. **Query** (ask, extract, or retrieve chunks)

> **Important**: The upload response always shows status `"New"` regardless of path. Do not use the status field to decide next steps — use whether you set `partition_method` during upload.

## How to use

Load the relevant rule for the task at hand:

| Task | Rule |
|------|------|
| Upload files, URLs, GitHub repos, or YouTube videos | [upload-sources](references/upload-sources.md) |
| Process documents and check status | [parsing](references/parsing.md) |
| Ask questions about documents (conversational Q&A) | [ask-sources](references/ask-sources.md) |
| Extract structured data using JSON Schema | [extraction](references/extraction.md) |
| Retrieve chunks for custom RAG pipelines | [retrieve-chunks](references/retrieve-chunks.md) |
| List, inspect, or delete documents | [manage-sources](references/manage-sources.md) |

## Universal rules

1. **Use MCP tools for all operations**.
2. **Always use `file_ids` over `file_names`** — `file_names` is deprecated. Store `file_id` from upload responses.
3. **After upload, check whether you set `partition_method`**:
   - **Set**: Skip parse. Go directly to query. Do not call parse — processing already happened.
   - **Not set**: You must call parse to process the document before querying.
4. **If you get auth errors**, the `GRAPHOR_API_KEY` environment variable is not set. Tell the user to run `export GRAPHOR_API_KEY="grlm_..."` and restart Claude Code.
