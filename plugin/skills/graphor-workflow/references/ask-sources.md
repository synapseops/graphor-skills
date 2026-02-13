---
name: ask-sources
description: Ask questions about documents with conversational memory and thinking levels.
metadata:
  tags: ask, chat, question, conversation, query
---

# Ask Sources

Use the Graphor MCP tools for all queries. Never make direct API calls.

The primary way to query documents in natural language. For structured data extraction, use [extraction](extraction.md). For raw chunk retrieval (custom RAG), use [retrieve-chunks](retrieve-chunks.md).

## Basic usage

Send a `question` string. Graphor searches all documents and returns a response containing:
- `answer` — natural language answer
- `conversation_id` — identifier for follow-up questions

## Conversation memory

Pass `conversation_id` from the previous response to maintain context for follow-up questions.

- **First question**: Send only `question` — a new `conversation_id` is returned
- **Follow-up**: Send `question` + `conversation_id` from the previous response
- **New conversation**: Omit `conversation_id` to start a fresh, independent conversation
- **Reset existing conversation**: Send `reset: true` + `conversation_id` to clear history of that conversation and start over

This is valuable for multi-turn research — "What are the main findings?" followed by "Elaborate on the second point."

## File scoping

Restrict the query to specific documents:

- Use `file_ids` (preferred) — array of file IDs from upload responses or list-sources
- Use `file_names` (deprecated) — array of file names

When the user's intent is clearly about specific documents, always scope the query. This reduces noise from unrelated documents and improves answer quality.

## Thinking levels

Control the depth of reasoning:

| Level | Use when | Tradeoff |
|-------|----------|----------|
| `fast` | Simple lookups, quick facts, yes/no questions | Fastest, least thorough |
| `balanced` | General questions, summaries | Good balance |
| `accurate` | Complex analysis, multi-document reasoning, nuanced questions | Slowest, most thorough (default) |

## Structured responses

You can pass an `output_schema` (JSON Schema) to get structured data alongside the answer. The response will include `structured_output` (validated) and `raw_json` (unvalidated). For heavy extraction work, prefer the dedicated extraction endpoint instead — see [extraction](extraction.md).

## Anti-patterns

- **Use MCP tools**.
- **Do not forget to pass `conversation_id` for follow-ups.** Without it, each question starts from scratch with no context.
- **Do not query all documents when only specific ones are relevant.** Scope with `file_ids` for better results.
