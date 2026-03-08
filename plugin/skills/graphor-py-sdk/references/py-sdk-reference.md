# Graphor Python SDK Reference

## Package

- **PyPI**: `graphor`
- **Install**: `pip install graphor` / `pip install graphor[aiohttp]`
- **Import**: `from graphor import Graphor`
- **Docs**: https://docs.graphorlm.com/sdk/overview
- **Repo**: https://github.com/synapseops/graphor-python-sdk

## Client Constructors

### Sync

```python
Graphor(api_key=None, max_retries=2, timeout=60.0)
```

### Async

```python
from graphor import AsyncGraphor, DefaultAioHttpClient
AsyncGraphor(api_key=None, max_retries=2, timeout=60.0, http_client=DefaultAioHttpClient())
```

Use as async context manager: `async with AsyncGraphor(...) as client:`

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `api_key` | `str` | `os.environ["GRAPHOR_API_KEY"]` | API key (`grlm_...`) |
| `max_retries` | `int` | `2` | Max retry attempts with exponential backoff |
| `timeout` | `float` | `60.0` | Request timeout in seconds |
| `base_url` | `str` | (API base URL) | API base URL override |

## Methods

### Sources — Ingest (async; return build_id)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ingest_file(file=, method=)` | `file`: path, bytes, or file-like; `method?`: str | `SourceIngestFileResponse` (`.build_id`) |
| `sources.ingest_url(url=, crawl_urls=, method=)` | `url`: str, `crawl_urls?`: bool, `method?`: str | `SourceIngestURLResponse` (`.build_id`) |
| `sources.ingest_github(url=)` | `url`: str (GitHub repo URL) | `SourceIngestGitHubResponse` (`.build_id`) |
| `sources.ingest_youtube(url=)` | `url`: str (YouTube video URL) | `SourceIngestYoutubeResponse` (`.build_id`) |

**method** (partition strategy): `"fast"` | `"balanced"` | `"accurate"` | `"vlm"` | `"agentic"`

### Sources — Build Status

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.get_build_status(build_id, suppress_elements=, suppress_img_base64=, page=, page_size=)` | `build_id`: str (required); optional pagination/filter | `BuildStatus` |

**BuildStatus.status**: `"Pending"` (request received, build not started), `"Processing"`, `"Completed"`, `"Processing failed"`, `"not_found"`. Use `success` to detect completion; use `file_id` when `success` is `True`.

### Sources — Reprocess (async; return build_id)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.reprocess(file_id=, method=)` | `file_id`: str (required), `method?`: str | `SourceReprocessResponse` (`.build_id`) |

### Sources — Manage

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.list(file_ids=)` | `file_ids?`: list[str] — optional filter | list of source objects (each: `file_id`, `file_name`, `status`, etc.) |
| `sources.delete(file_id=)` | `file_id`: str (required) | `SourceDeleteResponse` |
| `sources.get_elements(file_id=, page=, page_size=, suppress_img_base64=, type=, page_numbers=, elements_to_remove=)` | `file_id`: str (required); optional: pagination, `type`, `page_numbers`, `elements_to_remove` | `SourceGetElementsResponse` |

Filter for `get_elements` is via flat params (no nested `filter`): `type`, `page_numbers`, `elements_to_remove`.

### Chat

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ask(question=, conversation_id=, reset=, file_ids=, file_names=, output_schema=, thinking_level=)` | Prefer `file_ids` | `SourceAskResponse` |

**thinking_level**: `"fast"` | `"balanced"` | `"accurate"` (default)

### Extraction

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.extract(file_ids=, file_names=, user_instruction=, output_schema=, thinking_level=)` | At least one of `file_ids`/`file_names`; prefer `file_ids` | `SourceExtractResponse` |

### RAG

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.retrieve_chunks(query=, file_ids=, file_names=)` | `query`: str; prefer `file_ids` | `SourceRetrieveChunksResponse` |

## Response Types

| Type | Fields |
|------|--------|
| `SourceIngestFileResponse` / `SourceIngestURLResponse` / etc. | `build_id` |
| `BuildStatus` | `build_id`, `status`, `success`, `file_id`, `file_name`, `error`, `method`, `total_partitions`, `total_pages`, `created_at`, `updated_at`, `message`, `elements`, `total_elements`, `page`, `page_size`, `total_pages_elements`. **status**: `Pending`, `Processing`, `Completed`, `Processing failed`, `not_found` |
| `SourceReprocessResponse` | `build_id` |
| `SourceGetElementsResponse` | `items` (list of elements), `total`, `page`, `total_pages`, etc. Each element: `element_id`, `element_type`, `text`, `markdown`, `html`, `page_number`, `position`, `bounding_box`, etc. |
| `SourceAskResponse` | `answer`, `conversation_id`, `structured_output?`, `raw_json?` |
| `SourceExtractResponse` | `structured_output`, `raw_json`, `file_ids`, `file_names` |
| `SourceRetrieveChunksResponse` | `query`, `total`, `chunks` (each: `text`, `file_id`, `file_name?`, `page_number?`, `score?`, `metadata?`) |
| `SourceDeleteResponse` | `message`, etc. |

Import types from `graphor` as needed (e.g. `from graphor.types import ...`).

## Error Classes

Errors are imported from the `graphor` module, **not** from the client class:

```python
import graphor
# graphor.BadRequestError, graphor.AuthenticationError, etc.

# Or direct import:
from graphor import BadRequestError, AuthenticationError
```

| Class | HTTP Code |
|-------|-----------|
| `graphor.BadRequestError` | 400 |
| `graphor.AuthenticationError` | 401 |
| `graphor.PermissionDeniedError` | 403 |
| `graphor.NotFoundError` | 404 |
| `graphor.ConflictError` | 409 |
| `graphor.UnprocessableEntityError` | 422 |
| `graphor.RateLimitError` | 429 |
| `graphor.InternalServerError` | 500+ |
| `graphor.APIConnectionError` | Network |
| `graphor.APITimeoutError` | Timeout |
| `graphor.APIError` | Base class for all API errors |

## Per-Request Options

Use `.with_options()` to override config per-request:

```python
client.with_options(max_retries=3, timeout=300.0).sources.reprocess(
    file_id="large-doc-id",
    method="agentic"
)
```

## Async Pattern

```python
from graphor import AsyncGraphor, DefaultAioHttpClient

async with AsyncGraphor(http_client=DefaultAioHttpClient()) as client:
    response = await client.sources.ingest_file(file=Path("doc.pdf"))
    build_id = response.build_id
    # Poll get_build_status(build_id) until success
    status = await client.sources.get_build_status(build_id)
    if status.success:
        file_id = status.file_id
    sources = await client.sources.list()
    response = await client.sources.ask(question="Summarize", file_ids=[file_id])
    chunks = await client.sources.retrieve_chunks(query="payment terms", file_ids=[file_id])
```

All sync methods have async equivalents with identical signatures.
