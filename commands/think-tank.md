---
name: claudikins-github-readme-for-perfectionists:think-tank
description: Stage 3 - Research how similar projects nail their READMEs (external web research phase; graph tools available on-demand for local contrast)
allowed-tools:
  - Read
  - WebFetch
  - WebSearch
  - Task
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - think-tank
---

# claudikins-github-readme-for-perfectionists:think-tank Command

You are conducting Stage 3 of the README creation workflow — researching exemplar READMEs.

## Workflow

1. Load the `think-tank` skill for methodology.
2. Identify similar/competing projects.
3. Analyse their README strategies.
4. Extract patterns and inspiration.

## Research Tool Usage

**Primary:** `WebSearch` + `WebFetch` (no API keys, no Exa required). See `skills/think-tank/references/web-research.md`.

**Optional enhancements** (never hard-fail if missing):

- Exa MCP — faster search if installed; pipeline continues without it
- `claudikins-tool-executor` Gemini deep research — legacy path only

## Graph Tool Usage

This is an external web research phase — graph tools are **not** part of the standard workflow. If a locally-grounded comparison is needed (e.g., confirming our API surface matches patterns found in exemplars), you may call `get_architecture` on demand via `execute_code`. See `references/graph-analysis.md` at the plugin root for the tool-executor 3-step workflow.

## Key Questions

- What READMEs in this space are excellent?
- What patterns do they use?
- What makes them effective?
- What can we learn and adapt?

## Output

Generate a think-tank report saved to `.claude/grfp/think-tank.md`

## Next Stage

When complete, prompt user for `/claudikins-github-readme-for-perfectionists:brain-jam`
