---
name: claudikins-github-readme-for-perfectionists:pen-wielding
description: Stage 5 - Write the complete README with all insights combined (optionally uses get_architecture via claudikins-tool-executor to auto-generate Mermaid architecture diagrams)
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Task
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - pen-wielding
---

# claudikins-github-readme-for-perfectionists:pen-wielding Command

You are conducting Stage 5 of the README creation workflow — writing the final README.

## Workflow

1. Load the `pen-wielding` skill for methodology.
2. Review all previous stage outputs (`.claude/grfp/deep-dive.md`, `crystal-ball.md`, etc.).
3. In Step 5 (Visual Engineering), if `claudikins-tool-executor` is available, use `execute_code` to call `serena.get_architecture({ aspects: ["packages", "services", "entry_points"] })` and generate a Mermaid module diagram from the result. If graph tools are unavailable, hand-write the diagram from the deep-dive Reality Report. See the skill for full diagram-generation code.
4. Write the README section by section following the Anti-Slop protocol.
5. Polish and finalise.

## Tool Access

Graph tools are accessed via the `claudikins-tool-executor` plugin:

```
search_tools(query) → get_tool_schema(name) → execute_code(code)
```

Inside `execute_code`, graph tools are exposed via the `serena` client. See `references/graph-analysis.md` at the plugin root for details. Graph tool usage in this stage is optional — the README can be written without it.

## Key Sections

- Hook/Opening
- What it does
- Architecture diagram (from `get_architecture` or hand-written)
- Quick start
- Features
- Installation
- Usage examples
- Configuration
- Contributing
- License

## Output

Write the final README to `README.md` in the project root.

## Completion

This is the final stage. Celebrate the perfection!
