---
name: upload-sources
description: Upload files, URLs, GitHub repos, and YouTube videos to Graphor.
metadata:
  tags: upload, ingest, sources, files, url, github, youtube
---

# Upload Sources

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

## After upload

Processing is **asynchronous**. The response includes `file_id`, `file_name`, and `status`. The status will be `"Uploaded"` or `"Processing"` — you must wait until it becomes `"Completed"` before querying. See [async-processing](async-processing.md).

## Anti-patterns

- **Do not query immediately after upload.** Processing takes time. Always check status first.
- **Do not discard `file_id` from the upload response.** It is the primary identifier. `file_name` is deprecated for identification.
- **Do not upload duplicate files** without checking existing sources first via list.
