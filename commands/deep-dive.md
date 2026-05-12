---
name: claudikins-github-readme-for-perfectionists:deep-dive
description: Stage 1 - Deep dive into the codebase to understand what it IS (uses graph-analysis tools via claudikins-tool-executor for semantic codebase analysis)
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - deep-dive
---

# claudikins-github-readme-for-perfectionists:deep-dive Command

You are conducting Stage 1 of the README creation workflow — understanding what the project IS.

## Workflow

1. Load the `deep-dive` skill for methodology.
2. Verify the codebase graph index (via `serena.index_status` inside `execute_code`). If the `claudikins-tool-executor` plugin is not installed or indexing fails, fall back to Read/Glob/Grep with filename heuristics.
3. Use graph-analysis tools (`get_architecture`, `search_graph`, `trace_path`, `get_code_snippet`, `query_graph`) for semantic codebase analysis. See `references/graph-analysis.md` at the plugin root for tool catalogue and the tool-executor 3-step workflow.
4. Use Read/Glob/Grep for non-indexed content: dependency files (`package.json`, `pyproject.toml`), CI configs (`.github/workflows/`), docs, chat history.
5. Document findings for later stages.

## Tool Access

Graph tools are accessed via the `claudikins-tool-executor` plugin using the canonical 3-step workflow:

```
search_tools(query) → get_tool_schema(name) → execute_code(code)
```

Inside `execute_code`, graph tools are exposed via the `serena` client. See `references/graph-analysis.md` for full details.

If the tool-executor plugin is not installed, all graph tool calls will fail. The skill's fallback path uses the original filename-heuristic methodology — no hard error.

## Key Questions

- What problem does this solve?
- Who is it for?
- What are the core features?
- How is it architected? (Use `get_architecture` first)
- What are the true entry points? (Use `search_graph` with `max_degree: 0` for graph roots)
- What makes it unique?

## Output

Generate a deep-dive report saved to `.claude/grfp/deep-dive.md`. The report must note:

- Whether graph tools were available (Yes/No)
- Which method was used for each finding (graph vs filename-fallback)

## Next Stage

When complete, prompt user for `/claudikins-github-readme-for-perfectionists:crystal-ball`.
