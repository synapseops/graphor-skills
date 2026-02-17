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

If you omit `partition_method`, no processing happens at upload time — you must call parse separately afterward (Graphor will choose a default method at parse time based on file type).

If you set `partition_method` during upload, processing starts automatically — you do **not** need to call parse separately. Proceed directly to query (ask, extract, or retrieve chunks).

## URL upload

Upload from any web URL via MCP tools. Supports `crawl_urls` option — when `true`, follows and ingests linked pages. Useful for documentation sites. The page content (HTML) is extracted and parsed.

## GitHub upload

Upload an entire GitHub repository by URL via MCP tools. Text-based files (code, markdown, configs) are extracted and ingested.

## YouTube upload

Upload a YouTube video by URL via MCP tools. The transcript/captions are extracted.

## After upload: Two paths

The upload response includes `file_id`, `file_name`, `status`, and `message`. The `status` field will be `"New"` regardless of whether `partition_method` was set. **Do not rely on the `status` field to determine if processing happened** — it may lag behind actual processing state.

**`"New"` is the expected status. It is not an error.**

- **If you set `partition_method` during upload**: Processing already happened. **Skip parse. Go directly to query** (ask, extract, or retrieve chunks). The `message` field will confirm: `"Source uploaded and processed successfully"`.
- **If you did NOT set `partition_method` during upload**: The document is not processed yet. You must explicitly call parse next. See [parsing](parsing.md).

## Anti-patterns

- **Do not query immediately after upload if you did not set a `partition_method`.** Call parse first — the document must reach status `"Processed"` before it can be queried. An unprocessed `"New"` document cannot be queried.
- **Do not call parse after upload if you DID set `partition_method`.** Processing already happened during upload. Calling parse again is unnecessary and wastes time.
- **Do not assume upload starts processing if you did not set a `partition_method`.** You must explicitly trigger processing via parse.
- **Do not rely on the `status` field to determine if processing completed.** The API may return `"New"` even when processing finished. If you set `partition_method` during upload, trust that processing is done and proceed to query.
- **Do not discard `file_id` from the upload response.** It is the primary identifier. `file_name` is deprecated.
- **Uploading a file with the same name as an existing source will overwrite it.** Check existing sources via list before uploading to avoid unintended overwrites.
