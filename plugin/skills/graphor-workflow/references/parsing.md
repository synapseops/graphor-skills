---
name: parsing
description: Poll build status after ingest or reprocess, and reprocess documents when needed.
metadata:
  tags: processing, status, build status, reprocess, pending, polling
---

# Build Status and Reprocessing

After **ingest** (ingest_file, ingest_url, ingest_github, ingest_youtube) or **reprocess**, you receive a **build_id**. The build runs asynchronously. Use the **get_build_status** MCP tool to poll until the job completes and you get **file_id**.

## Get build status (polling)

Call **get_build_status** with the **build_id** returned by any ingest or by reprocess.

**Response fields:**
- **success** — `true` only when the build completed successfully; then **file_id** and **file_name** are present
- **status** — one of:
  - **Pending** — request received, build has not started yet; keep polling
  - **Processing** — build is running; keep polling
  - **Completed** — build finished successfully; use **file_id**
  - **Processing failed** — build failed; check **error**
  - **not_found** — no history yet (e.g. build not started or invalid build_id)
- **file_id**, **file_name** — present when success is true; use file_id for ask, extract, retrieve_chunks, get_elements, delete_source, reprocess
- **error** — error message when the build failed

**Typical loop:** call get_build_status every 2–5 seconds until **success** is true (then use file_id) or until you treat failure (e.g. status === "Processing failed" or error is set).

Optional query params for get_build_status: **suppress_elements**, **suppress_img_base64**, **page**, **page_size** (when you want parsed elements in the response and pagination).

## Reprocessing

To re-run the ingestion pipeline on an **existing** source with a different partition method, use the **reprocess** MCP tool.

- **file_id** (required) — from list_sources or get_build_status
- **method** (optional) — `fast`, `balanced`, `accurate`, `vlm`, `agentic`

**reprocess** returns a **build_id** (not file_id). Poll **get_build_status(build_id)** the same way as after ingest. When success is true, the **file_id** is unchanged — you keep using the same file_id for that source.

Use reprocess when results were unsatisfactory (e.g. wrong structure, missing tables) and you want to try a different **method**.

## Anti-patterns

- **Do not use file_id before get_build_status returns success.** Poll until success is true.
- **Do not treat Pending or Processing as failure.** Keep polling; only treat Processing failed or a non-null error as failure when appropriate.
- **Do not forget to poll after reprocess.** Reprocess returns build_id; you must poll get_build_status again to confirm completion (file_id stays the same).
- **Use MCP tools** for get_build_status and reprocess.
