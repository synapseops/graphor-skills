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
| `base_url` | `str` | `https://api.graphorlm.com/api/public/v1` | API base URL override |

## Methods

### Sources — Upload

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.upload(file=, partition_method=)` | `file`: file object or bytes, `partition_method?`: PartitionMethod | `PublicSource` |
| `sources.upload_url(url=, crawl_urls=, partition_method=)` | `url`: str, `crawl_urls?`: bool | `PublicSource` |
| `sources.upload_github(url=)` | `url`: str (GitHub repo URL) | `PublicSource` |
| `sources.upload_youtube(url=)` | `url`: str (YouTube video URL) | `PublicSource` |

**PartitionMethod**: `"basic"` | `"hi_res"` | `"hi_res_ft"` | `"mai"` | `"graphorlm"`

### Sources — Process

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.parse(file_id=, file_name=, partition_method=)` | At least one of `file_id`/`file_name` required. Prefer `file_id` (`file_name` is deprecated) | `PublicSource` |

### Sources — Manage

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.list()` | none | `list[PublicSource]` |
| `sources.delete(file_id=, file_name=)` | At least one required. Prefer `file_id` (`file_name` is deprecated) | `SourceDeleteResponse` |
| `sources.load_elements(file_id=, file_name=, page=, page_size=, filter=)` | Prefer `file_id` (`file_name` is deprecated). `filter?`: `{ elements_to_remove?, page_numbers?, type? }` | `SourceLoadElementsResponse` |

### Chat

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ask(question=, conversation_id=, reset=, file_ids=, file_names=, output_schema=, thinking_level=)` | Prefer `file_ids` (`file_names` is deprecated) | `SourceAskResponse` |

**`thinking_level`**: `"fast"` | `"balanced"` | `"accurate"` (default)

### Extraction

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.extract(file_ids=, file_names=, user_instruction=, output_schema=, thinking_level=)` | At least one of `file_ids`/`file_names` required. Prefer `file_ids` (`file_names` is deprecated) | `SourceExtractResponse` |

### RAG

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.retrieve_chunks(query=, file_ids=, file_names=)` | `query`: str. Prefer `file_ids` (`file_names` is deprecated) | `SourceRetrieveChunksResponse` |

## Response Types

All response objects are Pydantic models with typed fields.

| Type | Fields |
|------|--------|
| `PublicSource` | `file_name`, `file_id?`, `file_size`, `file_source`, `file_type`, `message`, `project_id`, `project_name`, `status`, `partition_method?`. `status` values: `"New"`, `"Waiting"`, `"Uploading"`, `"Not parsed"`, `"Processing"`, `"Processed"`, `"Completed"`, `"Failed"`, `"Processing failed"`, `"Upload failed"`, `"Service unavailable"` |
| `SourceAskResponse` | `answer`, `conversation_id`, `structured_output?`, `raw_json?` |
| `SourceExtractResponse` | `file_ids`, `file_names`, `structured_output`, `raw_json` |
| `SourceRetrieveChunksResponse` | `query`, `total`, `chunks` (each: `text`, `file_name?`, `file_id?`, `page_number?`, `score?`, `metadata?`) |
| `SourceDeleteResponse` | `file_name`, `file_id?`, `message`, `project_id`, `project_name`, `status` |
| `SourceLoadElementsResponse` | `items`, `total`, `page`, `page_size`, `total_pages` |

Import types: `from graphor.types import PublicSource, SourceAskResponse, ...`

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
client.with_options(max_retries=3, timeout=300.0).sources.parse(
    file_id="large-doc-id",
    partition_method="graphorlm"
)
```

## Async Pattern

```python
from graphor import AsyncGraphor, DefaultAioHttpClient

async with AsyncGraphor(http_client=DefaultAioHttpClient()) as client:
    source = await client.sources.upload(file=open("doc.pdf", "rb"))
    sources = await client.sources.list()
    response = await client.sources.ask(question="Summarize")
    chunks = await client.sources.retrieve_chunks(query="payment terms")
```

All sync methods have async equivalents with identical signatures.
