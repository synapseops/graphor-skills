---
name: retrieve-chunks
description: Retrieve semantically relevant chunks for custom RAG pipelines.
metadata:
  tags: rag, chunks, retrieve, search, semantic, embeddings
---

# Retrieve Chunks

Get raw, scored text passages from documents for use in custom RAG pipelines. Unlike ask-sources (which returns a synthesized answer), this returns the relevant source chunks directly.

## When to use

- Building a custom RAG pipeline with your own LLM
- Needing the raw source text, not a synthesized answer
- Wanting to control how context is assembled and prompted
- Feeding document context to external systems

## Core usage

Send a `query` string. Optionally scope with `file_ids` (preferred) or `file_names`.

The response includes:
- `chunks` — array of matches, each with:
  - `text` — the chunk content
  - `file_name` — source document name
  - `file_id` — source document ID
  - `page_number` — page in source document
  - `score` — relevance score (higher = more relevant)
  - `metadata` — additional chunk metadata
- `total` — total number of matching chunks

## Building custom RAG

Typical pattern:
1. Retrieve chunks with your query
2. Assemble the chunk texts into a context string
3. Pass that context to your preferred LLM (OpenAI, Anthropic, etc.)

## Anti-patterns

- **Do not use retrieve-chunks when ask-sources would suffice.** If you just want an answer, use ask-sources — it handles RAG internally with better quality.
- **Do not ignore scores.** Low-scoring chunks may be noise. Consider filtering by a score threshold.
- **Do not retrieve from unprocessed documents.** Same rule as always — verify `"Completed"` status first.
