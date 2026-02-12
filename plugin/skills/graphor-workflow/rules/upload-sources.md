---
name: upload-sources
description: Upload files, URLs, GitHub repos, and YouTube videos to Graphor.
metadata:
  tags: upload, ingest, sources, files, url, github, youtube
---

# Upload Sources

Graphor supports four source types. After upload, always store the returned `file_id` for subsequent operations. Prefer `file_id` over `file_name` — `file_name` is deprecated as an identifier.

## File upload

Supports: PDF, DOCX, DOC, TXT, MD, HTML, CSV, XLSX, XLS, PNG, JPG, JPEG, GIF, BMP, TIFF, MP3, MP4, WAV, WEBM.

You can optionally set the parsing method at upload time with `partition_method` (ordered from lowest to highest accuracy):
- `basic` — Fast - Text-only extraction (least accurate)
- `hi_res` — Balanced - High-resolution OCR for scanned documents
- `hi_res_ft` — Accurate - Hi-res with fine-tuned models
- `mai` — VLM - Multi-modal AI parsing
- `graphorlm` — Agentic - Graphor's proprietary pipeline (most accurate, slowest)

If omitted, Graphor chooses the simplest method depending on the file type.

If you set `partition_method` during upload, processing starts automatically — you do **not** need to call parse separately.

## URL upload

Upload from any web URL via MCP tools. Supports `crawl_urls` option — when `true`, follows and ingests linked pages. Useful for documentation sites. The page content (HTML) is extracted and parsed.

## GitHub upload

Upload an entire GitHub repository by URL via MCP tools. Text-based files (code, markdown, configs) are extracted and ingested.

## YouTube upload

Upload a YouTube video by URL via MCP tools. The transcript/captions are extracted.

## After upload: Status is "New"

The upload response includes `file_id`, `file_name`, and `status`. The status will be `"New"` — even if `partition_method` was set.

**`"New"` is the expected status. It is not an error.**

**If you did not set a `partition_method` during upload, you must explicitly call the parse operation next.** See [parsing](parsing.md) for the next step.

## Anti-patterns

- **Do not query immediately after upload if you did not set a `partition_method`.** Call parse first — the document must have status `"Processed"` or `"New"` before it can be queried.
- **Do not assume upload starts processing if you did not set a `partition_method`.** You must explicitly trigger processing via parse.
- **Do not discard `file_id` from the upload response.** It is the primary identifier. `file_name` is deprecated.
- **Uploading a file with the same name as an existing source will overwrite it.** Check existing sources via list before uploading to avoid unintended overwrites.
