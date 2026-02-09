# Graphor API Reference

> **This reference is for understanding only.** Use Graphor MCP tools for all operations except local file uploads. The MCP server handles authentication securely. For local file uploads only, use Bash curl with `$GRAPHOR_API_KEY` — see [upload-sources](../rules/upload-sources.md).

## Base URL

```
https://api.graphorlm.com/api/public/v1
```

All endpoint paths below are relative to this base. Authentication is handled automatically by the MCP server.

---

## Sources — Upload

### Upload File

- **POST** `/sources/upload`
- **Content-Type**: `multipart/form-data`
- **Parameters**:
  - `file` (binary, required) — the file to upload
  - `partition_method` (string, optional) — `basic`, `hi_res`, `hi_res_ft`, `mai`, `graphorlm`
- **Response**: `PublicSource` — `{ file_name, file_id, file_size, file_source, file_type, message, project_id, project_name, status, partition_method? }`
- **Initial status**: `"New"` — source accepted but NOT processed. Must call parse next.

### Upload from URL

- **POST** `/sources/upload-url-source`
- **Parameters**:
  - `url` (string, required)
  - `crawl_urls` (boolean, optional) — follow and ingest linked pages
  - `partition_method` (string, optional)
- **Response**: `PublicSource`

### Upload from GitHub

- **POST** `/sources/upload-github-source`
- **Parameters**: `url` (string, required) — GitHub repository URL
- **Response**: `PublicSource`

### Upload from YouTube

- **POST** `/sources/upload-youtube-source`
- **Parameters**: `url` (string, required) — YouTube video URL
- **Response**: `PublicSource`

---

## Sources — Process

### Parse / Reprocess

- **POST** `/sources/process`
- **Parameters**:
  - `file_id` (string, optional, preferred) — at least one of file_id/file_name required
  - `file_name` (string, optional, deprecated)
  - `partition_method` (string, optional) — `basic`, `hi_res`, `hi_res_ft`, `mai`, `graphorlm`
- **Response**: `PublicSource`

---

## Sources — Manage

### List Sources

- **GET** `/sources`
- **Response**: Array of `PublicSource` objects
- **Status values**: `"New"` (uploaded, not processed), `"Processing"`, `"Completed"`, `"Failed"`

### Delete Source

- **POST** `/sources/delete`
- **Parameters**:
  - `file_id` (string, optional, preferred) — at least one of file_id/file_name required
  - `file_name` (string, optional, deprecated)
- **Response**: `{ file_name, file_id?, message, project_id, project_name, status }`

### Load Elements

- **GET** `/sources/elements`
- **Parameters**:
  - `file_id` (string, optional, preferred) — at least one of file_id/file_name required
  - `file_name` (string, optional, deprecated)
  - `page` (integer, optional)
  - `page_size` (integer, optional)
  - `filter` (object, optional):
    - `elements_to_remove` (string[], optional)
    - `page_numbers` (integer[], optional)
    - `type` (string, optional)
- **Response**: `{ items, total, page, page_size, total_pages }`

---

## Chat

### Ask Sources

- **POST** `/sources/ask-sources`
- **Parameters**:
  - `question` (string, required)
  - `conversation_id` (string, optional) — for follow-up questions
  - `reset` (boolean, optional) — clear conversation context
  - `file_ids` (string[], optional, preferred) — scope to specific documents
  - `file_names` (string[], optional, deprecated)
  - `output_schema` (object, optional) — JSON Schema for structured responses
  - `thinking_level` (string, optional) — `fast`, `balanced`, `accurate` (default)
- **Response**:
  - `answer` (string)
  - `conversation_id` (string)
  - `structured_output` (any, when output_schema provided)
  - `raw_json` (string, when output_schema provided)

---

## Extraction

### Run Extraction

- **POST** `/sources/run-extraction`
- **Parameters**:
  - `file_ids` (string[], optional, preferred) — at least one of file_ids/file_names required
  - `file_names` (string[], optional, deprecated)
  - `user_instruction` (string, required)
  - `output_schema` (object, required) — simplified JSON Schema
  - `thinking_level` (string, optional) — `fast`, `balanced`, `accurate` (default)
- **Response**:
  - `file_ids` (string[])
  - `file_names` (string[])
  - `structured_output` (object)
  - `raw_json` (string)

**Schema constraints**: No `oneOf`, `anyOf`, `allOf`, `$ref`. Unions only with null.

---

## RAG

### Retrieve Chunks

- **POST** `/sources/prebuilt-rag`
- **Parameters**:
  - `query` (string, required)
  - `file_ids` (string[], optional, preferred)
  - `file_names` (string[], optional, deprecated)
- **Response**:
  - `query` (string)
  - `total` (integer)
  - `chunks` — array of:
    - `text` (string)
    - `file_name` (string, optional)
    - `file_id` (string, optional)
    - `page_number` (integer, optional)
    - `score` (number, optional)
    - `metadata` (object, optional)

---

## Flows

### List Flows

- **GET** `https://flows.graphorlm.com`
- **Response**: `{ flows: [{ name, description, status, url }], total }`
- **Status values**: `Deployed`, `Not deployed`, `New`, `Failed`

### Deploy Flow

- **POST** `https://{flow_name}.flows.graphorlm.com/deploy`
- **Parameters**: `tool_description` (string, optional)
- **Response**: `{ revision_id, status }`

### List Chunking Nodes

- **GET** `https://{flow_name}.flows.graphorlm.com/chunking`
- **Response**: Array of chunking node objects with config and status

---

## Error Codes

| Code | Type | Description |
|------|------|-------------|
| 400 | Bad Request | Invalid parameters or malformed request |
| 401 | Authentication Error | Invalid or missing API key |
| 403 | Permission Denied | Insufficient permissions |
| 404 | Not Found | Resource not found or not yet processed |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable Entity | Schema validation failed |
| 429 | Rate Limit | Too many requests — back off and retry |
| 500+ | Server Error | Internal error — retry once |

## Supported File Formats

PDF, DOCX, DOC, TXT, MD, HTML, CSV, XLSX, XLS, PNG, JPG, JPEG, GIF, BMP, TIFF, MP3, MP4, WAV, WEBM
