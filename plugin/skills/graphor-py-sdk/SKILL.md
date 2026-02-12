---
name: graphor-py-sdk
description: >
  Graphor Python SDK patterns and best practices. Use when writing Python code
  that integrates with Graphor, importing the graphor package, or building Python
  applications with document intelligence features including async client support.
user-invocable: false
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

## Core Workflow: Upload → Process → Ask

Upload returns status `"New"`. If you set `partition_method` during upload, processing starts automatically. Otherwise, you must call parse explicitly — it is **synchronous** and returns when done.

### Upload Sources

```python
# File upload — supports optional partition_method at upload time
source = client.sources.upload(
    file=open("report.pdf", "rb"),
    partition_method="hi_res"  # optional: basic | hi_res | hi_res_ft | mai | graphorlm
)
# IMPORTANT: Store source.file_id for all subsequent operations

# URL upload — supports crawl_urls to follow linked pages
client.sources.upload_url(
    url="https://example.com/article",
    crawl_urls=True  # optional: follow and ingest linked pages
)

# GitHub
client.sources.upload_github(url="https://github.com/org/repo")

# YouTube
client.sources.upload_youtube(url="https://youtube.com/watch?v=...")
```

### Process (parse)

Only required if `partition_method` was not set during upload. The call is **synchronous** — when it returns, the document is `"Processed"` and ready to query.

```python
client.sources.parse(
    file_id="the-file-id",  # preferred over file_name
    partition_method="graphorlm"
)
```

You can also call parse again to reprocess with a different method if results are unsatisfactory.

### Ask Questions

```python
response = client.sources.ask(
    question="What are the key findings?",
    file_ids=["file-id-1", "file-id-2"],  # preferred over file_names
    thinking_level="accurate"               # fast | balanced | accurate (default)
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
for chunk in result.chunks:
    print(f"[{chunk.file_name}, p.{chunk.page_number}] score={chunk.score:.2f}")
    print(chunk.text)
```

### Manage Sources

```python
sources = client.sources.list()
elements = client.sources.load_elements(
    file_id="the-file-id",
    page=1,
    page_size=50,
    filter={"page_numbers": [1, 2], "type": "text"}  # optional filtering
)
client.sources.delete(file_id="the-file-id")
```

## Async Usage

```python
from graphor import AsyncGraphor, DefaultAioHttpClient

async def main():
    async with AsyncGraphor(http_client=DefaultAioHttpClient()) as client:
        source = await client.sources.upload(file=open("doc.pdf", "rb"))
        response = await client.sources.ask(question="Summarize this document")
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
except graphor.NotFoundError:                # 404 - file not found/not processed
    print("File not found")
except graphor.RateLimitError:               # 429 - back off and retry
    print("Rate limited")
except graphor.BadRequestError:              # 400
    print("Bad request")
except graphor.ConflictError:                # 409
    print("Conflict")
except graphor.UnprocessableEntityError:     # 422 - schema validation failed
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
    api_key=os.environ.get("GRAPHOR_API_KEY"),  # default
    max_retries=5,                                # default: 2
    timeout=120.0                                 # default: 60.0 seconds
)

# Per-request overrides via .with_options()
client.with_options(max_retries=3, timeout=300.0).sources.parse(
    file_id="large-doc-id",
    partition_method="graphorlm"
)
```

## Anti-patterns

- **Do not catch errors as `Graphor.BadRequestError`.** The Python SDK does not attach error classes to the client. Use `graphor.BadRequestError` or import directly from `graphor`.
- **Do not use `file_names` parameter.** It is deprecated. Always use `file_ids`.
- **Do not query before processing if you did not set `partition_method` during upload.** Call parse first.
- **Do not use `oneOf`, `anyOf`, `allOf`, or `$ref` in extraction schemas.** They are not supported.
- **Do not forget `conversation_id` for follow-up questions.** Without it, context is lost.

For the full method reference, see `references/py-sdk-reference.md`.
