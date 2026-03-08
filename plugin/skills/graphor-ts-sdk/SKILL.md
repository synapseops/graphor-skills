---
name: graphor-ts-sdk
description: Graphor TypeScript/JavaScript SDK patterns and best practices. Use when writing TypeScript or JavaScript code that integrates with Graphor, importing from the graphor npm package, or building Node.js/Deno/Bun applications with document intelligence features.
---

You are assisting with TypeScript/JavaScript code that uses the Graphor SDK (`graphor` npm package). Follow these patterns and rules.

## Setup

```typescript
import Graphor from 'graphor';

// Preferred: uses GRAPHOR_API_KEY env var automatically
const client = new Graphor();

// Or explicit:
const client = new Graphor({ apiKey: 'grlm_...' });
```

**Requirements**: TypeScript >= 4.9, Node.js 20+ LTS. Also works in Deno 1.28+, Bun 1.0+, Cloudflare Workers, Vercel Edge Runtime, and browsers.

## Core Workflow: Ingest → Poll Build Status → Use file_id

Ingestion is **asynchronous**. Each ingest method returns an object with **build_id** immediately. Poll **getBuildStatus(build_id)** until `success` is `true` to obtain **file_id**, then use `file_id` for ask, extract, retrieveChunks, getElements, delete, and reprocess.

### Ingest Sources

```typescript
import Graphor from 'graphor';
import fs from 'fs';

const client = new Graphor();

// File ingest — returns { build_id }
const { build_id: buildId } = await client.sources.ingestFile({
  file: fs.createReadStream('./report.pdf'),
  method: 'balanced'  // optional: 'fast' | 'balanced' | 'accurate' | 'vlm' | 'agentic'
});

// Poll until ready (status: Pending → Processing → Completed)
let fileId: string;
while (true) {
  const status = await client.sources.getBuildStatus(buildId);
  if (status.success && status.file_id) {
    fileId = status.file_id;
    console.log('Ready. file_id:', fileId);
    break;
  }
  if (status.error && status.status !== 'not_found' && status.status !== 'Pending' && status.status !== 'Processing') {
    throw new Error(status.error);
  }
  await new Promise(r => setTimeout(r, 2000));
}

// URL ingest
const { build_id: urlBuildId } = await client.sources.ingestURL({
  url: 'https://example.com/article',
  crawlUrls: true,
  method: 'balanced'
});

// GitHub
const { build_id: ghBuildId } = await client.sources.ingestGitHub({ url: 'https://github.com/org/repo' });

// YouTube
const { build_id: ytBuildId } = await client.sources.ingestYoutube({ url: 'https://youtube.com/watch?v=...' });
```

**Build status values**: `Pending` (request received, build not started), `Processing`, `Completed`, `Processing failed`, `not_found`. Keep polling until `status.success` is `true` or treat `Processing failed` / non-null `error` as failure.

**Method names**: `ingestFile`, `ingestURL` (capital URL), `ingestGitHub` (capital H), `ingestYoutube`. Parameters use **snake_case** in the request object (e.g. `file_id`, `page_size`).

### Get Build Status

```typescript
const status = await client.sources.getBuildStatus(buildId, {
  suppress_elements: false,
  suppress_img_base64: false,
  page: 1,
  page_size: 50
});
// status.success, status.status, status.file_id, status.file_name, status.error
// When success: use status.file_id for subsequent calls
```

### Reprocess (optional)

Re-run the pipeline on an existing source with a different partition method. Returns an object with **build_id** — poll getBuildStatus as usual.

```typescript
const { build_id: reprocessBuildId } = await client.sources.reprocess({
  file_id: 'the-file-id',
  method: 'agentic'
});
```

### Ask Questions

```typescript
const response = await client.sources.ask({
  question: 'What are the key findings?',
  file_ids: ['file-id-1', 'file-id-2'],
  thinking_level: 'accurate'
});
console.log(response.answer);

// Follow-up with conversation memory
const followUp = await client.sources.ask({
  question: 'Elaborate on the first point',
  conversation_id: response.conversation_id
});
```

### Extract Structured Data

```typescript
const result = await client.sources.extract({
  file_ids: ['invoice-file-id'],
  user_instruction: 'Extract invoice details',
  output_schema: {
    type: 'object',
    properties: {
      invoice_number: { type: 'string' },
      total_amount: { type: 'number' },
      line_items: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            description: { type: 'string' },
            amount: { type: 'number' }
          }
        }
      }
    }
  }
});
console.log(result.structured_output);
```

### Retrieve Chunks (RAG)

```typescript
const result = await client.sources.retrieveChunks({
  query: 'payment terms',
  file_ids: ['contract-file-id']
});
// result.chunks: { text, file_id, file_name?, page_number?, score?, metadata? }[]
const context = (result.chunks ?? []).map(c => `[${c.file_id}, p.${c.page_number}] ${c.text}`).join('\n');
```

### Manage Sources

```typescript
const sources = await client.sources.list();
// Optional: await client.sources.list({ file_ids: ['id1', 'id2'] })

const elements = await client.sources.getElements({
  file_id: 'the-file-id',
  page: 1,
  page_size: 50,
  type: 'Title',              // optional: filter by element type
  page_numbers: [1, 2],       // optional: restrict to pages
  elementsToRemove: ['Footer'] // optional: exclude element types
});

await client.sources.delete({ file_id: 'the-file-id' });
```

**getElements** uses flat parameters (no nested `filter`): `type`, `page_numbers`, `elementsToRemove`.

## Error Handling

Use typed error classes:

```typescript
import Graphor from 'graphor';

try {
  await client.sources.ask({ question: '...' });
} catch (err) {
  if (err instanceof Graphor.AuthenticationError) { /* 401 - bad API key */ }
  else if (err instanceof Graphor.NotFoundError) { /* 404 - source not found */ }
  else if (err instanceof Graphor.RateLimitError) { /* 429 - back off and retry */ }
  else if (err instanceof Graphor.BadRequestError) { /* 400 */ }
  else if (err instanceof Graphor.ConflictError) { /* 409 */ }
  else if (err instanceof Graphor.UnprocessableEntityError) { /* 422 - schema validation failed */ }
  else if (err instanceof Graphor.APIConnectionError) { /* network error */ }
  else if (err instanceof Graphor.APIError) { /* catch-all for other API errors */ }
  else throw err;
}
```

## Configuration

```typescript
const client = new Graphor({
  apiKey: process.env.GRAPHOR_API_KEY,
  maxRetries: 5,
  timeout: 120_000  // milliseconds
});

// Per-request overrides
await client.sources.reprocess({ file_id: 'id', method: 'agentic' }, { maxRetries: 3, timeout: 300_000 });
```

## Type Safety

The SDK exports typed params and response interfaces:

```typescript
const { build_id } = await client.sources.ingestFile({ file: stream });
const status = await client.sources.getBuildStatus(build_id);
// status: { success, status, file_id?, file_name?, error?, ... }
```

## Anti-patterns

- **Do not use `upload` or `uploadURL`/`uploadGitHub`.** Use **ingestFile**, **ingestURL**, **ingestGitHub**, **ingestYoutube**. Method names: `ingestURL` (capital URL), `ingestGitHub` (capital H).
- **Do not use `parse`.** Use **reprocess** with `file_id` and `method`. It is async and returns `build_id`; poll getBuildStatus.
- **Do not assume ingest or reprocess is synchronous.** They return `build_id`; always poll **getBuildStatus** until `success` is `true` before using `file_id`.
- **Do not use a nested `filter` for getElements.** Use flat params: `type`, `page_numbers`, `elementsToRemove`.
- **Do not use `file_names` when `file_ids` is available.** Prefer `file_ids` for ask, extract, retrieveChunks.
- **Do not use `oneOf`, `anyOf`, `allOf`, or `$ref` in extraction schemas.** They are not supported.
- **Do not forget `conversation_id` for follow-up questions.** Without it, context is lost.

For the full method reference, see `references/ts-sdk-reference.md`.
