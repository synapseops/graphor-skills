---
name: graphor-py-sdk
description: Graphor Python SDK patterns and best practices. Use when writing Python code that integrates with Graphor, importing the graphor package, or building Python applications with document intelligence features including async client support.
---

You are assisting with Python code that uses the Graphor SDK (`graphor` PyPI package). Follow these patterns and rules.

## Setup

```python
from graphor import Graphor

# Preferred: uses GRAPHOR_API_KEY env var automatically
client = Graphor()

# Or explicit:
client = Graphor(api_key="grlm_...")
```

**Requirements**: Python 3.9+. Optional: `pip install graphor[aiohttp]` for better async performance.

## Core Workflow: Ingest → Poll Build Status → Use file_id

Ingestion is **asynchronous**. Each ingest method returns a **build_id** immediately. Poll **get_build_status(build_id)** until `success` is `True` to obtain the **file_id**, then use `file_id` for ask, extract, retrieve_chunks, get_elements, delete, and reprocess.

### Ingest Sources

```python
from pathlib import Path
import time

# File ingest — returns response with build_id
response = client.sources.ingest_file(
    file=Path("./report.pdf"),
    method="balanced"  # optional: fast | balanced | accurate | vlm | agentic
)
build_id = response.build_id

# Poll until ready (status: Pending → Processing → Completed)
while True:
    status = client.sources.get_build_status(build_id)
    if status.success:
        file_id = status.file_id
        print(f"Ready. file_id: {file_id}")
        break
    if status.error and status.status not in ("not_found", "Pending", "Processing"):
        raise RuntimeError(status.error)
    time.sleep(2)

# URL ingest
response = client.sources.ingest_url(
    url="https://example.com/article",
    crawl_urls=True,  # optional: follow and ingest linked pages
    method="balanced"
)
build_id = response.build_id

# GitHub
response = client.sources.ingest_github(url="https://github.com/org/repo")
build_id = response.build_id

# YouTube
response = client.sources.ingest_youtube(url="https://youtube.com/watch?v=...")
build_id = response.build_id
```

**Build status values**: `Pending` (request received, build not started), `Processing`, `Completed`, `Processing failed`, `not_found`. Keep polling until `status.success` is `True` or treat `Processing failed` / non-null `error` as failure.

### Get Build Status

```python
status = client.sources.get_build_status(
    build_id,
    suppress_elements=False,
    suppress_img_base64=False,
    page=1,
    page_size=50
)
# status.success, status.status, status.file_id, status.file_name, status.error
# When success: use status.file_id for subsequent calls
```

### Reprocess (optional)

Re-run the pipeline on an existing source with a different partition method. Returns a **build_id** — poll get_build_status as usual.

```python
response = client.sources.reprocess(
    file_id="the-file-id",
    method="agentic"  # fast | balanced | accurate | vlm | agentic
)
build_id = response.build_id
```

### Ask Questions

```python
response = client.sources.ask(
    question="What are the key findings?",
    file_ids=["file-id-1", "file-id-2"],
    thinking_level="accurate"  # fast | balanced | accurate (default)
)
print(response.answer)

# Follow-up with conversation memory
follow_up = client.sources.ask(
    question="Elaborate on the first point",
    conversation_id=response.conversation_id
)
```

### Extract Structured Data

```python
result = client.sources.extract(
    file_ids=["invoice-file-id"],
    user_instruction="Extract invoice details",
    output_schema={
        "type": "object",
        "properties": {
            "invoice_number": {"type": "string"},
            "total_amount": {"type": "number"},
            "line_items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "amount": {"type": "number"}
                    }
                }
            }
        }
    }
)
print(result.structured_output)
```

### Retrieve Chunks (RAG)

```python
result = client.sources.retrieve_chunks(
    query="payment terms",
    file_ids=["contract-file-id"]
)
for chunk in result.chunks or []:
    print(f"[{chunk.file_id}, p.{chunk.page_number}] score={chunk.score:.2f}")
    print(chunk.text)
```

### Manage Sources

```python
sources = client.sources.list()
# Optional filter: client.sources.list(file_ids=["id1", "id2"])

elements = client.sources.get_elements(
    file_id="the-file-id",
    page=1,
    page_size=50,
    type="Title",                    # optional: filter by element type
    page_numbers=[1, 2],             # optional: restrict to pages
    elements_to_remove=["Footer"]     # optional: exclude element types
)

client.sources.delete(file_id="the-file-id")
```

## Async Usage

```python
from graphor import AsyncGraphor, DefaultAioHttpClient
import asyncio

async def main():
    async with AsyncGraphor(http_client=DefaultAioHttpClient()) as client:
        response = await client.sources.ingest_file(file=Path("doc.pdf"))
        build_id = response.build_id
        while True:
            status = await client.sources.get_build_status(build_id)
            if status.success:
                file_id = status.file_id
                break
            await asyncio.sleep(2)
        response = await client.sources.ask(question="Summarize this document", file_ids=[file_id])
        print(response.answer)
```

Requires: `pip install graphor[aiohttp]`

All sync methods have async equivalents with identical signatures.

## Error Handling

Import error classes from the `graphor` module — **not** from the `Graphor` client class:

```python
import graphor

client = graphor.Graphor()
try:
    response = client.sources.ask(question="...")
except graphor.AuthenticationError:          # 401 - bad API key
    print("Invalid API key")
except graphor.NotFoundError:               # 404 - source not found
    print("Source not found")
except graphor.RateLimitError:               # 429 - back off and retry
    print("Rate limited")
except graphor.BadRequestError:              # 400
    print("Bad request")
except graphor.ConflictError:                # 409
    print("Conflict")
except graphor.UnprocessableEntityError:    # 422 - schema validation failed
    print("Schema validation failed")
except graphor.APIConnectionError:           # network error
    print("Connection error")
except graphor.APIError as e:                # catch-all for other API errors
    print(f"API error: {e.status_code}")
```

Or use direct imports:
```python
from graphor import AuthenticationError, NotFoundError, RateLimitError
```

## Configuration

```python
client = Graphor(
    api_key=os.environ.get("GRAPHOR_API_KEY"),
    max_retries=5,
    timeout=120.0
)

# Per-request overrides via .with_options()
client.with_options(timeout=300.0).sources.reprocess(
    file_id="large-doc-id",
    method="agentic"
)
```

## Anti-patterns

- **Do not catch errors as `Graphor.BadRequestError`.** The Python SDK does not attach error classes to the client. Use `graphor.BadRequestError` or import directly from `graphor`.
- **Do not use `file_names` when `file_ids` is available.** Prefer `file_ids` for ask, extract, retrieve_chunks.
- **Do not assume ingest or reprocess is synchronous.** They return `build_id`; always poll `get_build_status` until `success` is `True` before using `file_id`.
- **Do not use a nested `filter` for get_elements.** Use flat parameters: `type=`, `page_numbers=`, `elements_to_remove=`.
- **Do not use `oneOf`, `anyOf`, `allOf`, or `$ref` in extraction schemas.** They are not supported.
- **Do not forget `conversation_id` for follow-up questions.** Without it, context is lost.

For the full method reference, see `references/py-sdk-reference.md`.
