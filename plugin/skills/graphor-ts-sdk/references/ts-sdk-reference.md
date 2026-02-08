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
| `baseURL` | `string` | `https://api.graphorlm.com/api/public/v1` | API base URL override |

## Methods

### Sources — Upload

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.upload({ file, partition_method? })` | `file`: Uploadable, `partition_method?`: PartitionMethod | `PublicSource` |
| `sources.uploadURL({ url, crawl_urls?, partition_method? })` | `url`: string, `crawl_urls?`: boolean | `PublicSource` |
| `sources.uploadGitHub({ url })` | `url`: string (GitHub repo URL) | `PublicSource` |
| `sources.uploadYoutube({ url })` | `url`: string (YouTube video URL) | `PublicSource` |

**Important**: Method names are `uploadURL` (capital URL) and `uploadGitHub` (capital H).

**PartitionMethod**: `'basic'` | `'hi_res'` | `'hi_res_ft'` | `'mai'` | `'graphorlm'`

### Sources — Process

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.parse({ file_id?, file_name?, partition_method? })` | At least one of `file_id`/`file_name` required | `PublicSource` |

### Sources — Manage

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.list()` | none | `PublicSource[]` |
| `sources.delete({ file_id?, file_name? })` | At least one required. Prefer `file_id` | `{ file_name, file_id?, message, project_id, project_name, status }` |
| `sources.loadElements({ file_id?, file_name?, page?, page_size?, filter? })` | `filter?`: `{ elementsToRemove?, page_numbers?, type? }` | `{ items, total, page, page_size, total_pages }` |

### Chat

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.ask({ question, conversation_id?, reset?, file_ids?, file_names?, output_schema?, thinking_level? })` | See below | `{ answer, conversation_id, structured_output?, raw_json? }` |

**`thinking_level`**: `'fast'` | `'balanced'` | `'accurate'` (default)

### Extraction

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.extract({ file_ids?, file_names?, user_instruction, output_schema, thinking_level? })` | At least one of `file_ids`/`file_names` required | `{ file_ids, file_names, structured_output, raw_json }` |

### RAG

| Method | Parameters | Returns |
|--------|-----------|---------|
| `sources.retrieveChunks({ query, file_ids?, file_names? })` | `query`: string | `{ query, chunks, total }` |

Each chunk: `{ text, file_name?, file_id?, page_number?, score?, metadata? }`

## Response Types

| Type | Description |
|------|-------------|
| `Graphor.PublicSource` | `{ file_name, file_id?, file_size, file_source, file_type, message, project_id, project_name, status, partition_method? }` |
| `Graphor.SourceAskResponse` | `{ answer, conversation_id, structured_output?, raw_json? }` |
| `Graphor.SourceExtractResponse` | `{ file_ids, file_names, structured_output, raw_json }` |
| `Graphor.SourceRetrieveChunksResponse` | `{ query, total, chunks }` |
| `Graphor.SourceDeleteResponse` | `{ file_name, file_id?, message, project_id, project_name, status }` |
| `Graphor.SourceLoadElementsResponse` | `{ items, total, page, page_size, total_pages }` |

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

The `file` parameter accepts:
- `fs.createReadStream(path)` — Node.js file streams
- `new File([data], name)` — Browser File API
- `await fetch(url)` — Fetch Response objects
- `await toFile(buffer, name)` — Buffer/Uint8Array via helper

Import: `import { toFile } from 'graphor'`

## Per-Request Options

Any method accepts a second argument for request-level config:

```typescript
await client.sources.ask(params, {
  maxRetries: 3,
  timeout: 300_000
});
```
