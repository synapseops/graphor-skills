# Graphor API Reference

> **This reference is for understanding only.** Use Graphor MCP tools for all operations. The MCP server handles authentication securely.

## Base URL

```
https://sources.graphorlm.com
```

Authentication is handled automatically by the MCP server.

---

## Sources — Ingest (async; return build_id)

All ingest endpoints return **build_id** immediately. Poll **GET /builds/{build_id}** (get_build_status) until **success** is true to obtain **file_id**.

### Ingest File

- **POST** `/ingest-file`
- **Content-Type**: `multipart/form-data`
- **Parameters**:
  - `file` (binary, required)
  - `method` (string, optional) — `fast`, `balanced`, `accurate`, `vlm`, `agentic`
- **Response**: `{ build_id, success: true, error: null }`

### Ingest URL

- **POST** `/ingest-url`
- **Parameters**: `url` (required), `crawlUrls` (optional), `method` (optional)
- **Response**: `{ build_id, ... }`

### Ingest GitHub

- **POST** `/ingest-github`
- **Parameters**: `url` (required) — GitHub repository URL
- **Response**: `{ build_id, ... }`

### Ingest YouTube

- **POST** `/ingest-youtube`
- **Parameters**: `url` (required) — YouTube video URL
- **Response**: `{ build_id, ... }`

---

## Get Build Status

- **GET** `/builds/{build_id}`
- **Query params**: `suppress_elements`, `suppress_img_base64`, `page`, `page_size`
- **Response**: `build_id`, `status`, `success`, `file_id`, `file_name`, `error`, `message`, `elements` (when not suppressed), pagination fields

**status** values:
- **Pending** — request received, build not started yet; keep polling
- **Processing** — build running; keep polling
- **Completed** — build finished; use **file_id**
- **Processing failed** — build failed; check **error**
- **not_found** — no history yet (build not started or invalid id)

Use **file_id** from the response when **success** is true.

---

## Sources — Reprocess (async; return build_id)

- **POST** `/reprocess`
- **Body**: `{ file_id (required), method? }` — method: `fast`, `balanced`, `accurate`, `vlm`, `agentic`
- **Response**: `{ build_id, ... }` — poll get_build_status as for ingest

---

## Sources — Manage

### List Sources

- **GET** `/` (or `/sources`)
- **Query**: `file_ids` (optional array) — filter by file IDs
- **Response**: Array of source objects with `file_id`, `file_name`, `status`, etc.

### Delete Source

- **DELETE** `/delete`
- **Body**: `{ file_id }` (required)
- **Response**: `{ message, ... }`

### Get Elements

- **GET** `/get-elements`
- **Query**: `file_id` (required), `page`, `page_size`, `suppress_img_base64`, `type`, `page_numbers`, `elements_to_remove` — flat params (no nested filter object)
- **Response**: `{ items, total, page, total_pages, ... }` — each item has `element_type`, `text`, `page_number`, etc.

---

## Chat

### Ask Sources

- **POST** `/ask-sources`
- **Body**: `question` (required), `conversation_id?`, `reset?`, `file_ids?`, `file_names?`, `output_schema?`, `thinking_level?` (`fast` | `balanced` | `accurate`)
- **Response**: `answer`, `conversation_id`, `structured_output?`, `raw_json?`

---

## Extraction

### Run Extraction

- **POST** `/run-extraction`
- **Body**: `file_ids?`, `file_names?`, `user_instruction` (required), `output_schema` (required), `thinking_level?`
- **Response**: `structured_output`, `raw_json`, `file_ids`, `file_names`

**Schema constraints**: No `oneOf`, `anyOf`, `allOf`, `$ref`. Nullable via `["type", "null"]`.

---

## RAG

### Retrieve Chunks

- **POST** `/prebuilt-rag`
- **Body**: `query` (required), `file_ids?`, `file_names?`
- **Response**: `query`, `total`, `chunks` — each chunk: `text`, `file_id`, `file_name?`, `page_number?`, `score?`, `metadata?`

---

## Flows

### List Flows

- **GET** `https://flows.graphorlm.com`
- **Response**: `{ flows: [{ name, description, status, url }], total }`

### Deploy Flow

- **POST** `https://{flow_name}.flows.graphorlm.com/deploy`
- **Body**: `tool_description?`
- **Response**: `{ revision_id, status }`

### List Chunking Nodes

- **GET** `https://{flow_name}.flows.graphorlm.com/chunking`
- **Response**: Array of chunking node objects

---

## Error Codes

| Code | Type | Description |
|------|------|-------------|
| 400 | Bad Request | Invalid parameters or malformed request |
| 401 | Authentication Error | Invalid or missing API key |
| 403 | Permission Denied | Insufficient permissions |
| 404 | Not Found | Resource not found (e.g. source not found for file_id) |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable Entity | Schema validation failed |
| 429 | Rate Limit | Too many requests — back off and retry |
| 500+ | Server Error | Internal error — retry once |

## Supported File Formats

PDF, DOC, DOCX, ODT, PPT, PPTX, CSV, TSV, XLS, XLSX, TXT, MD, HTML, PNG, JPG, TIFF, BMP, HEIC, MP4, MOV, AVI, MKV, WEBM, MP3, WAV, M4A, OGG, FLAC
