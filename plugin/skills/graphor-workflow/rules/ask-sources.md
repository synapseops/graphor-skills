---
name: ask-sources
description: Ask questions about documents with conversational memory and thinking levels.
metadata:
  tags: ask, chat, question, conversation, query
---

# Ask Sources

Use the Graphor MCP tools for all queries. Never make direct API calls.

The primary way to query documents. Supports conversational memory, file scoping, and adjustable reasoning depth.

**Prerequisite**: Documents must have status `"Completed"` before querying. If status is `"New"`, you must call parse first. See [async-processing](async-processing.md).

## Basic usage

Send a `question` string. Graphor searches all processed documents and returns a natural language `answer`.

## Conversation memory

The response includes a `conversation_id`. Pass it back in subsequent requests to maintain context for follow-up questions.

- **First question**: Send only `question`
- **Follow-up**: Send `question` + `conversation_id` from previous response
- **Reset context**: Send `reset: true` to clear the conversation and start fresh

This is valuable for multi-turn research — "What are the main findings?" followed by "Elaborate on the second point."

## File scoping

Restrict the query to specific documents:

- Use `file_ids` (preferred) — array of file IDs from upload responses
- Use `file_names` (deprecated) — array of file names

When the user's intent is clearly about specific documents, always scope the query. This reduces noise from unrelated documents and improves answer quality.

## Thinking levels

Control the depth of reasoning:

| Level | Use when | Tradeoff |
|-------|----------|----------|
| `fast` | Simple lookups, quick facts, yes/no questions | Fastest, least thorough |
| `balanced` | General questions, summaries | Good balance |
| `accurate` | Complex analysis, multi-document reasoning, nuanced questions | Slowest, most thorough (default) |

Do not default to `accurate` for simple questions — it wastes time. Match the thinking level to the query complexity.

## Structured responses

You can pass an `output_schema` (JSON Schema) to get structured data alongside the answer. The response will include `structured_output` (validated) and `raw_json` (unvalidated). For heavy extraction work, prefer the dedicated extraction endpoint instead — see [extraction](extraction.md).

## Anti-patterns

- **Do not ask about documents in `"New"` or `"Processing"` status.** Always verify `"Completed"` status first. If `"New"`, call parse first.
- **Do not make direct curl or HTTP calls** to the Graphor API. Always use MCP tools.
- **Do not forget to pass `conversation_id` for follow-ups.** Without it, each question starts from scratch with no context.
- **Do not use `accurate` thinking level for simple lookups.** It's slower for no benefit on simple queries.
- **Do not query all documents when only specific ones are relevant.** Scope with `file_ids` for better results.
