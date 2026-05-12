---
name: claudikins-github-readme-for-perfectionists:grfp
description: Begin README creation workflow or check current progress and missed areas
argument-hint: [--status | --resume]
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - WebFetch
  - WebSearch
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - grfp
---

# claudikins-github-readme-for-perfectionists:grfp Command

You are orchestrating the GitHub README For Perfectionists workflow.

## Arguments

- `--status` → Show current progress through the 5-stage workflow
- `--resume` → Continue from last completed stage
- No argument → Start fresh or show overview

## Workflow

1. Load the `grfp` skill for methodology.
2. **Run Phase 0: Graph Index Check** before any other phase begins. Use `execute_code` to call `serena.index_status()`. If not indexed, call `serena.index_repository()`. If indexing fails, write `graph-status.json` with `available: false` and warn the user once — do not hard-fail. See the skill for full Phase 0 code.
3. Guide user through the 5-stage README creation process.
4. Track progress and ensure quality at each stage.

## Tool Access

Graph tools are accessed via the `claudikins-tool-executor` plugin using the canonical 3-step workflow:

```
search_tools(query) → get_tool_schema(name) → execute_code(code)
```

Inside `execute_code`, graph tools are exposed via the `serena` client. See `references/graph-analysis.md` at the plugin root for the full tool catalogue and usage patterns.

## The 5 Stages

| #   | Stage        | Command                                                     | Purpose                          |
| --- | ------------ | ----------------------------------------------------------- | -------------------------------- |
| 1   | Deep Dive    | `/claudikins-github-readme-for-perfectionists:deep-dive`    | Understand what the project IS   |
| 2   | Crystal Ball | `/claudikins-github-readme-for-perfectionists:crystal-ball` | Envision what it COULD become    |
| 3   | Brain Jam    | `/claudikins-github-readme-for-perfectionists:brain-jam`    | Collaborate with Gemini on angle |
| 4   | Think Tank   | `/claudikins-github-readme-for-perfectionists:think-tank`   | Research exemplar READMEs        |
| 5   | Pen Wielding | `/claudikins-github-readme-for-perfectionists:pen-wielding` | Write the final README           |

## Philosophy

> "A README is your project's first impression. Make it count."

- Dual-AI collaboration (Claude + Gemini)
- Research-driven, not template-driven
- Quality over speed
