# Graphor Skills

Claude Code plugin for [Graphor](https://graphorlm.com) — document intelligence powered by AI. Upload, process, and query documents through natural language.

This plugin provides **MCP server connectivity** and **domain expertise skills** for working with Graphor, following the [Agent Skills](https://agentskills.io) open standard.

## What's Included

| Component | Description |
|-----------|-------------|
| **`.mcp.json`** | Auto-configures the `graphor-mcp` server for API access |
| **`graphor-workflow`** | Core skill: upload → process → ask workflow via MCP tools |
| **`graphor-ts-sdk`** | TypeScript SDK coding patterns and best practices |
| **`graphor-py-sdk`** | Python SDK coding patterns and best practices |

### Skill Details

**`graphor-workflow`** (auto-activates when relevant)
The primary skill. Teaches the agent to upload documents (files, URLs, GitHub repos, YouTube videos), handle async processing, and query documents via ask, extraction, and chunk retrieval. Paired with the MCP server for direct API access.

**`graphor-ts-sdk`** (background knowledge)
Activates when writing TypeScript/JavaScript code with the `graphor` npm package. Covers client setup, all SDK methods, file upload patterns, error handling, and type safety.

**`graphor-py-sdk`** (background knowledge)
Activates when writing Python code with the `graphor` PyPI package. Covers sync and async clients, all SDK methods, error handling, and configuration.

## Installation

### Claude Code — Plugin Marketplace

```
/plugin marketplace add synapseops/graphor-skills
/plugin install graphor-skills
```

### Claude Code — Direct

```bash
claude --add-dir /path/to/graphor-skills
```

### Project-Level (team-wide)

Copy the `skills/` directory into your project's `.claude/skills/`:

```bash
cp -r skills/* your-project/.claude/skills/
```

### Personal (all projects)

```bash
cp -r skills/* ~/.claude/skills/
```

## Configuration

Set your Graphor API key as an environment variable:

```bash
export GRAPHOR_API_KEY="grlm_your_api_key_here"
```

The MCP server (`graphor-mcp`) uses this key automatically. Get your API key from the [Graphor dashboard](https://graphorlm.com).

## Compatibility

This plugin follows the [Agent Skills open standard](https://agentskills.io), making the skills portable across 27+ agent tools including Claude Code, Cursor, VS Code, GitHub Copilot, Gemini CLI, and more.

The MCP server configuration (`.mcp.json`) is specific to Claude Code and compatible tools.

## Links

- [Graphor Documentation](https://docs.graphorlm.com)
- [TypeScript SDK](https://github.com/synapseops/graphor-typescript-sdk) — `npm install graphor`
- [Python SDK](https://github.com/synapseops/graphor-python-sdk) — `pip install graphor`
- [MCP Server](https://www.npmjs.com/package/graphor-mcp) — `npx -y graphor-mcp`
- [Agent Skills Standard](https://agentskills.io)

## License

MIT
