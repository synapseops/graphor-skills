---
name: upload-sources
description: Ingest files, URLs, GitHub repos, and YouTube videos to Graphor.
metadata:
  tags: upload, ingest, sources, files, url, github, youtube
---

# Ingest Sources

Graphor supports four source types. **Ingestion is asynchronous**: each ingest returns a **build_id** immediately. You must **poll get_build_status(build_id)** until **success** is true to obtain **file_id**, then use **file_id** for all subsequent operations (ask, extract, retrieve_chunks, get_elements, delete_source, reprocess).

## File ingest

Use the **ingest_file** MCP tool. Supports: PDF, DOCX, DOC, TXT, MD, HTML, CSV, XLSX, XLS, PNG, JPG, TIFF, MP4, and more.

You can optionally set the partition method with **method** (ordered from fastest to most accurate):

- `fast` — Fastest, rule-based partitioning
- `balanced` — Balanced speed and accuracy (layout-aware)
- `accurate` — High-accuracy parsing
- `vlm` — Vision-language model for complex visual documents
- `agentic` — Agentic pipeline (highest accuracy, slowest)

If omitted, the system default is used.

**Two modes for ingest_file:**
- **file_path** — absolute path to a file (local MCP)
- **file_content** + **file_name** — text content and filename (works in remote MCP for text files)

## URL ingest

Use the **ingest_url** MCP tool. Pass **url** (required). Optionally **crawlUrls** (follow linked pages) and **method**. Returns **build_id** — poll get_build_status until success.

## GitHub ingest

Use the **ingest_github** MCP tool. Pass **url** (GitHub repo URL). Returns **build_id** — poll get_build_status until success.

## YouTube ingest

Use the **ingest_youtube** MCP tool. Pass **url** (YouTube video URL). Transcript/captions are extracted. Returns **build_id** — poll get_build_status until success.

## After ingest: poll get_build_status

The ingest response contains **build_id** only. It does **not** contain file_id yet.

1. Call **get_build_status(build_id)** (e.g. every 2–5 seconds).
2. **status** may be:
   - **Pending** — request received, build not started; keep polling
   - **Processing** — build running; keep polling
   - **Completed** — build finished; response includes **file_id** and **success: true**
   - **Processing failed** — build failed; check **error**
   - **not_found** — no history yet; keep polling
3. When **success** is true, use **file_id** for ask, extract, retrieve_chunks, get_elements, delete_source, reprocess.

**Do not use file_id before success is true.** There is no synchronous "upload then get file_id" — you must poll.

## Anti-patterns

- **Do not assume ingest returns file_id.** It returns build_id only. Always poll get_build_status until success.
- **Do not query (ask/extract/retrieve_chunks) before you have file_id.** Get file_id from get_build_status first.
- **Do not discard build_id.** You need it to poll. Store file_id only after get_build_status returns success.
- **Do not treat Pending or Processing as errors.** Keep polling until Completed or Processing failed.
