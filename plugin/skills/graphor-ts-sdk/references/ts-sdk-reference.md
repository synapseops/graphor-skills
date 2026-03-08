# Graphor TypeScript SDK Reference

## Package

- **npm**: `graphor`
- **Install**: `npm install graphor` / `yarn add graphor` / `pnpm add graphor`
- **Import**: `import Graphor from 'graphor'`
- **Docs**: https://docs.graphorlm.com/sdk/overview
- **Repo**: https://github.com/synapseops/graphor-typescript-sdk

## Client Constructor

```typescript
new Graphor(options?)
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `apiKey` | `string` | `process.env.GRAPHOR_API_KEY` | API key (`grlm_...`) |
| `maxRetries` | `number` | `2` | Max retry attempts with exponential backoff |
| `timeout` | `number` | `60000` | Request timeout in milliseconds |
| `baseURL` | `string` | (API base URL) | API base URL override |

## Methods

### Sources — Ingest (async; return build_id)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ingestFile({ file, method? })` | `file`: Uploadable, `method?`: string | `Promise<{ build_id }>` |
| `sources.ingestURL({ url, crawlUrls?, method? })` | `url`: string, `crawlUrls?`: boolean, `method?`: string | `Promise<{ build_id }>` |
| `sources.ingestGitHub({ url })` | `url`: string (GitHub repo URL) | `Promise<{ build_id }>` |
| `sources.ingestYoutube({ url })` | `url`: string (YouTube video URL) | `Promise<{ build_id }>` |

**Method names**: `ingestURL` (capital URL), `ingestGitHub` (capital H), `ingestYoutube`.

**method** (partition strategy): `'fast'` | `'balanced'` | `'accurate'` | `'vlm'` | `'agentic'`

### Sources — Build Status

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.getBuildStatus(buildId, { suppress_elements?, suppress_img_base64?, page?, page_size? }?)` | `buildId`: string (required); optional options object | `Promise<BuildStatus>` |

**BuildStatus**: `build_id`, `status`, `success`, `file_id`, `file_name`, `error`, `method`, `total_partitions`, `total_pages`, `created_at`, `updated_at`, `message`, `elements`, `total_elements`, `page`, `page_size`, `total_pages_elements`.

**status**: `'Pending'` (request received, build not started), `'Processing'`, `'Completed'`, `'Processing failed'`, `'not_found'`. Use `success` to detect completion; use `file_id` when `success` is `true`.

### Sources — Reprocess (async; return build_id)

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.reprocess({ file_id, method? })` | `file_id`: string (required), `method?`: string | `Promise<{ build_id }>` |

### Sources — Manage

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.list({ file_ids? }?)` | `file_ids?`: string[] — optional filter | `Promise<Source[]>` |
| `sources.delete({ file_id })` | `file_id`: string (required) | `Promise<SourceDeleteResponse>` |
| `sources.getElements({ file_id, page?, page_size?, suppress_img_base64?, type?, page_numbers?, elementsToRemove? })` | `file_id`: string (required); optional pagination and filter params | `Promise<SourceGetElementsResponse>` |

Filter for getElements is via flat params (no nested `filter`): `type`, `page_numbers`, `elementsToRemove`. Parameter names: **snake_case** for `file_id`, `page_size`, `suppress_img_base64`, `page_numbers`; **camelCase** for `elementsToRemove` (check SDK for exact casing).

### Chat

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ask({ question, conversation_id?, reset?, file_ids?, file_names?, output_schema?, thinking_level? })` | Prefer `file_ids` | `Promise<{ answer, conversation_id, structured_output?, raw_json? }>` |

**thinking_level**: `'fast'` | `'balanced'` | `'accurate'` (default)

### Extraction

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.extract({ file_ids?, file_names?, user_instruction, output_schema, thinking_level? })` | At least one of `file_ids`/`file_names`; prefer `file_ids` | `Promise<{ structured_output, raw_json, file_ids, file_names }>` |

### RAG

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.retrieveChunks({ query, file_ids?, file_names? })` | `query`: string; prefer `file_ids` | `Promise<{ query, total, chunks }>` |

Each chunk: `{ text, file_id, file_name?, page_number?, score?, metadata? }`

## Response Types

| Type | Description |
|------|-------------|
| Ingest / reprocess responses | `{ build_id: string }` |
| `BuildStatus` | `build_id`, `status`, `success`, `file_id`, `file_name`, `error`, `method`, `elements`, pagination fields. **status**: `Pending`, `Processing`, `Completed`, `Processing failed`, `not_found` |
| List item (source) | `file_id`, `file_name`, `status`, and other metadata |
| `SourceGetElementsResponse` | `items` (element array), `total`, `page`, `total_pages`, etc. Each element: `element_id`, `element_type`, `text`, `markdown`, `html`, `page_number`, `position`, etc. |
| `SourceAskResponse` | `answer`, `conversation_id`, `structured_output?`, `raw_json?` |
| `SourceExtractResponse` | `structured_output`, `raw_json`, `file_ids`, `file_names` |
| `SourceRetrieveChunksResponse` | `query`, `total`, `chunks` (each with `file_id`, `text`, `page_number`, `score`, etc.) |
| `SourceDeleteResponse` | `message`, etc. |

## Error Classes

| Class | HTTP Code |
|-------|-----------|
| `Graphor.BadRequestError` | 400 |
| `Graphor.AuthenticationError` | 401 |
| `Graphor.PermissionDeniedError` | 403 |
| `Graphor.NotFoundError` | 404 |
| `Graphor.ConflictError` | 409 |
| `Graphor.UnprocessableEntityError` | 422 |
| `Graphor.RateLimitError` | 429 |
| `Graphor.InternalServerError` | 500+ |
| `Graphor.APIConnectionError` | Network |
| `Graphor.APIConnectionTimeoutError` | Timeout |
| `Graphor.APIError` | Base class for all API errors |

## File Upload Input Types

The `file` parameter in `ingestFile` accepts:
- `fs.createReadStream(path)` — Node.js file streams
- `new File([data], name)` — Browser File API
- `await fetch(url)` — Fetch Response objects
- `await toFile(buffer, name)` — Buffer/Uint8Array via helper: `import { toFile } from 'graphor'`

## Per-Request Options

Any method accepts a second argument for request-level config:

```typescript
await client.sources.ask(params, {
  maxRetries: 3,
  timeout: 300_000
});
```
