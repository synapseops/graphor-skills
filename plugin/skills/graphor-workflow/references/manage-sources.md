---
name: manage-sources
description: List, inspect, and delete document sources.
metadata:
  tags: list, delete, elements, sources, manage
---

# Manage Sources

Use the Graphor MCP tools for all operations. Never make direct API calls.

Operations for listing, inspecting, and deleting documents.

## List sources

Returns all documents with their status, file names, file IDs, and metadata. Use this to:

- Find `file_id` values for previously uploaded documents
- Verify a document exists before querying
- Check for existing sources before uploading to avoid unintended overwrites

**Status field caveat**: The `status` field may not immediately reflect processing state. If you uploaded with `partition_method` set, the document is processed and queryable even if list_sources still shows `"New"`. Do not use the status from list_sources to decide whether to call parse — follow the upload workflow instead: if `partition_method` was set during upload, skip parse; if not, call parse.

## Load elements

Get the parsed content elements (text blocks, tables, images, sections) from a processed document. Useful for inspecting what Graphor extracted.

Parameters:
- `file_id` (preferred) or `file_name` (deprecated) — identify the document
- `page` and `page_size` — paginate results
- `filter` — optional filtering with:
  - `elements_to_remove` — element types to exclude
  - `page_numbers` — only elements from specific pages
  - `type` — only elements of a specific type

## Delete source

Remove a document. Use `file_id` (preferred) or `file_name` (deprecated). Deletion is irreversible.

## Anti-patterns

- **Do not use list_sources status to decide whether to call parse.** If you set `partition_method` during upload, the document is already processed regardless of the status shown. The status field may lag behind.
- **Do not delete documents without confirmation from the user.** Deletion is irreversible.
- **Do not assume a file exists.** Check via list first if uncertain.
- **Use MCP tools.**
