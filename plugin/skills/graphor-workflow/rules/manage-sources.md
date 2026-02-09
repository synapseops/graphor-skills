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

Returns all documents with their current status, file names, file IDs, and metadata. Use this to:

- Check processing status after upload
- Find `file_id` values for previously uploaded documents
- Verify a document exists before querying

## Load elements

Get the parsed content elements (text blocks, tables, images, sections) from a processed document. Useful for inspecting what Graphor extracted.

Parameters:
- `file_id` (preferred) or `file_name` — identify the document
- `page` and `page_size` — paginate results
- `filter` — optional filtering with:
  - `elements_to_remove` — element types to exclude
  - `page_numbers` — only elements from specific pages
  - `type` — only elements of a specific type

## Delete source

Remove a document. Use `file_id` (preferred) or `file_name`.

The response includes `file_name`, `status`, `message`, `project_id`, `project_name`, and optionally `file_id`.

## Anti-patterns

- **Do not delete documents without confirmation from the user.** Deletion is irreversible.
- **Do not assume a file exists.** Check via list first if uncertain.
- **Do not make direct curl or HTTP calls** to the Graphor API. Always use MCP tools.
