---
name: upload-sources
description: Upload files, URLs, GitHub repos, and YouTube videos to Graphor.
metadata:
  tags: upload, ingest, sources, files, url, github, youtube
---

# Upload Sources

Use the Graphor MCP tools to upload. Never make direct API calls.

Graphor supports four ingestion methods. After upload, always store the returned `file_id` for subsequent operations.

## File upload

Upload local files. Supports: PDF, DOCX, DOC, TXT, MD, HTML, CSV, XLSX, XLS, PNG, JPG, JPEG, GIF, BMP, TIFF, MP3, MP4, WAV, WEBM.

You can optionally set the parsing method at upload time with `partition_method`:
- `basic` — Fast, text-only extraction
- `hi_res` — High-resolution OCR for scanned documents
- `hi_res_ft` — Hi-res with fine-tuned models
- `mai` — Multi-modal AI parsing
- `graphorlm` — Graphor's proprietary pipeline (best quality, slowest)

If omitted, Graphor chooses automatically based on file type.

## URL upload

Upload from any web URL. Supports an additional `crawl_urls` option — when `true`, Graphor will follow and ingest linked pages from the source URL. Useful for documentation sites.

## GitHub upload

Upload an entire GitHub repository by URL. Code files, markdown, and documentation are processed.

## YouTube upload

Upload a YouTube video by URL. The video transcript is extracted and processed.

## After upload: Status is "New"

The upload response includes `file_id`, `file_name`, and `status`. The status will be `"New"` — this means the source was accepted but **has NOT been processed yet**.

**You must explicitly call the process/parse operation next.** Upload alone does not start processing. See [async-processing](async-processing.md) for the next step.

## Anti-patterns

- **Do not query immediately after upload.** Status `"New"` means unprocessed. You must call parse first, then wait for `"Completed"`.
- **Do not assume upload starts processing.** It does not. You must explicitly trigger processing via the parse operation.
- **Do not discard `file_id` from the upload response.** It is the primary identifier. `file_name` is deprecated for identification.
- **Do not upload duplicate files** without checking existing sources first via list.
- **Do not make direct curl or HTTP calls** to the Graphor API. Always use MCP tools.
