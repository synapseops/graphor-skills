---
name: graphor-workflow
description: Graphor document intelligence workflow. Use when uploading documents, ingesting files/URLs/GitHub repos/YouTube videos, polling build status, asking questions about documents, extracting structured data from documents, retrieving chunks for RAG, or managing document sources in Graphor.
---

You have access to the Graphor MCP server for interacting with the Graphor API.

## Prerequisites

The user must set their Graphor API key as a shell environment variable:

```bash
export GRAPHOR_API_KEY="grlm_your_api_key_here"
```

Add this to `~/.bashrc`, `~/.zshrc`, or equivalent. The `.mcp.json` uses `${GRAPHOR_API_KEY}` expansion to read this value. If MCP operations return auth errors, the key is not set correctly.

## Core workflow: Ingest → Poll build status → Query

1. **Ingest** a source (file, URL, GitHub, YouTube) via MCP — returns **build_id** immediately (async).
2. **Poll get_build_status(build_id)** until **success** is true — then use the returned **file_id** for all subsequent operations. Status may be `Pending` (request received, build not started), then `Processing`, then `Completed`.
3. **Query** the source (ask, extract, retrieve_chunks, get_elements) or **manage** (list_sources, delete_source) using **file_id**.

Optional: **Reprocess** an existing source with a different partition method — returns **build_id**; poll get_build_status again.

## How to use

Load the relevant rule for the task at hand:

| Task | Rule |
|------|------|
| Ingest files, URLs, GitHub repos, or YouTube videos | [upload-sources](references/upload-sources.md) |
| Poll build status and reprocess documents | [parsing](references/parsing.md) |
| Ask questions about documents (conversational Q&A) | [ask-sources](references/ask-sources.md) |
| Extract structured data using JSON Schema | [extraction](references/extraction.md) |
| Retrieve chunks for custom RAG pipelines | [retrieve-chunks](references/retrieve-chunks.md) |
| List, inspect elements, or delete documents | [manage-sources](references/manage-sources.md) |

## Universal rules

1. **Use MCP tools for all operations** (ingest_file, ingest_url, ingest_github, ingest_youtube, get_build_status, list_sources, reprocess, get_elements, delete_source, ask, extract, retrieve_chunks).
2. **Always use file_id** — Get it from **get_build_status** (when success is true) or from **list_sources**. Use file_id for ask, extract, retrieve_chunks, get_elements, delete_source, reprocess.
3. **After ingest or reprocess, poll get_build_status** until **success** is true before using file_id. Status can be `Pending`, `Processing`, then `Completed` (or `Processing failed`).
4. **If you get auth errors**, the `GRAPHOR_API_KEY` environment variable is not set. Tell the user to run `export GRAPHOR_API_KEY="grlm_..."` and restart the client.
