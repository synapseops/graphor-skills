---
name: upload-sources
description: Upload files, URLs, GitHub repos, and YouTube videos to Graphor.
metadata:
  tags: upload, ingest, sources, files, url, github, youtube
---

# Upload Sources

Graphor supports four ingestion methods. After upload, always store the returned `file_id` for subsequent operations.

## Known limitation: Local file uploads

The current Graphor MCP server uses a sandboxed execute container that **cannot access the user's local filesystem**. This means `fs.createReadStream('/path/to/file')` will fail with ENOENT inside the MCP execute tool.

**For local file uploads, use Bash with curl:**

```bash
curl -s -X POST "https://api.graphorlm.com/api/public/v1/sources/upload" \
  -H "Authorization: Bearer $GRAPHOR_API_KEY" \
  -F "file=@/path/to/document.pdf"
```

This reads the file from the user's local filesystem and uploads it directly. The API key comes from the shell environment variable, not hardcoded.

**For URL, GitHub, and YouTube uploads**, use MCP tools — these don't require local file access.

## File upload

Supports: PDF, DOCX, DOC, TXT, MD, HTML, CSV, XLSX, XLS, PNG, JPG, JPEG, GIF, BMP, TIFF, MP3, MP4, WAV, WEBM.

You can optionally set the parsing method at upload time with `partition_method`:
- `basic` — Fast, text-only extraction
- `hi_res` — High-resolution OCR for scanned documents
- `hi_res_ft` — Hi-res with fine-tuned models
- `mai` — Multi-modal AI parsing
- `graphorlm` — Graphor's proprietary pipeline (best quality, slowest)

If omitted, Graphor chooses automatically based on file type.

To include partition_method in the curl upload:
```bash
curl -s -X POST "https://api.graphorlm.com/api/public/v1/sources/upload" \
  -H "Authorization: Bearer $GRAPHOR_API_KEY" \
  -F "file=@/path/to/document.pdf" \
  -F "partition_method=hi_res"
```

## URL upload (use MCP)

Upload from any web URL via MCP tools. Supports `crawl_urls` option — when `true`, follows and ingests linked pages. Useful for documentation sites.

## GitHub upload (use MCP)

Upload an entire GitHub repository by URL via MCP tools.

## YouTube upload (use MCP)

Upload a YouTube video by URL via MCP tools. The transcript is extracted.

## After upload: Status is "New"

The upload response includes `file_id`, `file_name`, and `status`. The status will be `"New"` — this means the source was accepted but **has NOT been processed yet**.

**`"New"` is the expected status. It is not an error.**

**You must explicitly call the process/parse operation next.** Upload alone does not start processing. See [async-processing](async-processing.md) for the next step.

## Anti-patterns

- **Do not query immediately after upload.** Status `"New"` means unprocessed. Call parse first, then wait for `"Completed"`.
- **Do not assume upload starts processing.** It does not. You must explicitly trigger processing via parse.
- **Do not discard `file_id` from the upload response.** It is the primary identifier. `file_name` is deprecated.
- **Do not upload duplicate files** without checking existing sources first via list.
- **Do not try `fs.createReadStream()` inside MCP execute** for local files. It runs in a sandboxed container without filesystem access. Use Bash curl instead.
- **Do not hardcode API keys** in curl commands. Use `$GRAPHOR_API_KEY` from the shell environment.
