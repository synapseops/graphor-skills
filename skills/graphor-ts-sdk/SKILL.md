---
name: graphor-ts-sdk
description: >
  Graphor TypeScript/JavaScript SDK patterns and best practices. Use when writing
  TypeScript or JavaScript code that integrates with Graphor, importing from the
  graphor npm package, or building Node.js/Deno/Bun applications with document
  intelligence features.
user-invocable: false
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

## Core Workflow: Upload → Process → Ask

**Processing is async.** After upload, poll `client.sources.list()` until status is `"Completed"` before querying. This is the most important rule.

### Upload Sources

```typescript
import fs from 'fs';

// File upload — supports optional partition_method at upload time
const source = await client.sources.upload({
  file: fs.createReadStream('report.pdf'),
  partition_method: 'hi_res' // optional: basic | hi_res | hi_res_ft | mai | graphorlm
});
// IMPORTANT: Store source.file_id for all subsequent operations

// URL upload — supports crawl_urls to follow linked pages
await client.sources.uploadURL({
  url: 'https://example.com/article',
  crawl_urls: true // optional: follow and ingest linked pages
});

// GitHub
await client.sources.uploadGitHub({ url: 'https://github.com/org/repo' });

// YouTube
await client.sources.uploadYoutube({ url: 'https://youtube.com/watch?v=...' });
```

For advanced file inputs, use the `toFile` helper:
```typescript
import { toFile } from 'graphor';
await client.sources.upload({ file: await toFile(Buffer.from('data'), 'file.txt') });
```

File input types accepted: `fs.createReadStream()`, `new File()`, `await fetch()` Response, `await toFile()`.

### Check Processing Status

```typescript
// Poll until completed
const sources = await client.sources.list();
const myDoc = sources.find(s => s.file_id === uploadedFileId);
if (myDoc?.status === 'Completed') {
  // Safe to query
}
```

### Reprocess with Different Method

```typescript
await client.sources.parse({
  file_id: 'the-file-id',  // preferred over file_name
  partition_method: 'graphorlm'
});
```

### Ask Questions

```typescript
const response = await client.sources.ask({
  question: 'What are the key findings?',
  file_ids: ['file-id-1', 'file-id-2'],  // preferred over file_names
  thinking_level: 'accurate'               // fast | balanced | accurate (default)
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
// result.chunks: { text, file_name?, file_id?, page_number?, score?, metadata? }[]
const context = result.chunks.map(c => c.text).join('\n');
```

### Manage Sources

```typescript
const sources = await client.sources.list();
const elements = await client.sources.loadElements({
  file_id: 'the-file-id',
  page: 1,
  page_size: 50,
  filter: { page_numbers: [1, 2], type: 'text' } // optional filtering
});
await client.sources.delete({ file_id: 'the-file-id' });
```

## Error Handling

Use typed error classes:

```typescript
import Graphor from 'graphor';

try {
  await client.sources.ask({ question: '...' });
} catch (err) {
  if (err instanceof Graphor.AuthenticationError) { /* 401 - bad API key */ }
  else if (err instanceof Graphor.NotFoundError) { /* 404 - file not found/not processed */ }
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
  apiKey: process.env.GRAPHOR_API_KEY,  // default
  maxRetries: 5,                         // default: 2, exponential backoff
  timeout: 120_000                       // default: 60_000ms
});

// Per-request overrides
await client.sources.parse(params, { maxRetries: 3, timeout: 300_000 });
```

## Type Safety

The SDK exports typed params and response interfaces:

```typescript
const params: Graphor.SourceUploadParams = { file: stream };
const source: Graphor.PublicSource = await client.sources.upload(params);
```

## Anti-patterns

- **Do not use `uploadUrl()` or `uploadGithub()`.** The correct method names are `uploadURL()` (capital URL) and `uploadGitHub()` (capital H). The SDK README may show lowercase variants but the actual methods use these capitalizations.
- **Do not use `file_names` parameter.** It is deprecated. Always use `file_ids`.
- **Do not query before processing completes.** Always check status after upload.
- **Do not use `oneOf`, `anyOf`, `allOf`, or `$ref` in extraction schemas.** They are not supported.
- **Do not forget `conversation_id` for follow-up questions.** Without it, context is lost.

For the full method reference, see `references/ts-sdk-reference.md`.
