---
name: manage-sources
description: List, inspect elements, and delete document sources.
metadata:
  tags: list, delete, elements, sources, manage, get_elements
---

# Manage Sources

Use the Graphor MCP tools for all operations. Never make direct API calls.

Operations for listing, inspecting, and deleting documents. All identify sources by **file_id** (from get_build_status or list_sources).

## List sources

Use the **list_sources** MCP tool. Returns all documents with **file_id**, **file_name**, **status**, and metadata.

Optional: pass **file_ids** (array) to filter and return only those sources.

Use list_sources to:
- Find **file_id** values for previously ingested documents (e.g. when you didn’t store them)
- Check current status of sources
- Verify a document exists before querying or deleting
- See what’s in the project before ingesting to avoid duplicates

## Get elements

Use the **get_elements** MCP tool to retrieve the parsed content elements (text blocks, tables, images, sections) of a processed source.

**Required:** **file_id** — from list_sources or get_build_status.

**Optional (flat parameters; no nested filter object):**
- **page**, **page_size** — pagination
- **suppress_img_base64** — omit base64 images to reduce payload size
- **type** — filter by element type (e.g. Title, Table, NarrativeText)
- **page_numbers** — restrict to specific pages (array of numbers)
- **elementsToRemove** — element types to exclude (array of strings)

Use get_elements to inspect how a document was segmented or to work with raw chunks.

## Delete source

Use the **delete_source** MCP tool. Pass **file_id** (required). Deletion is irreversible.

## Anti-patterns

- **Do not delete without user confirmation.** Deletion is irreversible.
- **Do not assume a source exists.** Check via list_sources first if unsure.
- **Do not use a nested filter for get_elements.** Use the flat parameters: type, page_numbers, elementsToRemove.
- **Use MCP tools** for list_sources, get_elements, delete_source.
