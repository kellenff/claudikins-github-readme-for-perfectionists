---
name: claudikins-github-readme-for-perfectionists:crystal-ball
description: Stage 2 - Envision what this project COULD become (uses graph-analysis tools via claudikins-tool-executor for dead-code detection, complexity hotspots, and attack-surface tracing)
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - WebSearch
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - crystal-ball
---

# claudikins-github-readme-for-perfectionists:crystal-ball Command

You are conducting Stage 2 of the README creation workflow — envisioning the project's potential.

## Workflow

1. Load the `crystal-ball` skill for methodology.
2. Review the deep-dive findings (read `.claude/grfp/deep-dive.md`).
3. Use graph-analysis tools (`search_graph`, `trace_path`, `query_graph`, `get_code_snippet`) for:
   - Dead-code detection (orphaned symbols with `max_degree: 0`)
   - Complexity hotspots (high fan-in/fan-out functions)
   - Deprecated API caller enumeration
   - Attack-surface paths from public entry points
4. Use Read/Grep for content-level checks: TODO/FIXME comments, hardcoded secrets, outdated dependency versions in manifests.
5. Generate the Roadmap Candidates report.

## Tool Access

Graph tools are accessed via the `claudikins-tool-executor` plugin using the canonical 3-step workflow:

```
search_tools(query) → get_tool_schema(name) → execute_code(code)
```

Inside `execute_code`, graph tools are exposed via the `serena` client. See `references/graph-analysis.md` at the plugin root for full details.

If the tool-executor plugin is not installed, fall back to manual code reading patterns documented in the skill.

## Key Questions

- What could this become with more development?
- What's dead code we should remove? (Graph: `max_degree: 0`)
- What's the complexity profile? (Graph: high fan-in/fan-out symbols)
- Where does it fit in the ecosystem?
- What's the vision/roadmap?
- What audience segments could benefit?

## Output

Generate a crystal-ball report saved to `.claude/grfp/crystal-ball.md`. The report's Dead Code section must cite confidence based on graph data (e.g., "0 inbound edges"); confirm each finding via `get_code_snippet` before listing.

## Next Stage

When complete, prompt user for `/claudikins-github-readme-for-perfectionists:think-tank`.
