---
name: claudikins-github-readme-for-perfectionists:brain-jam
description: Stage 4 - Collaborate with Chorus to find the perfect README angle (voice/strategy phase; graph tools available on-demand for architectural questions)
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
skills:
  - brain-jam
---

# claudikins-github-readme-for-perfectionists:brain-jam Command

You are conducting Stage 4 of the README creation workflow — multi-voice collaboration on the angle via Chorus.

Default cast: Gemini synth + MiniMax pragmatist/critic (2 rounds). Falls back to a single-provider cast when only one API key is set.

## Workflow

1. Load the `brain-jam` skill for methodology.
2. Synthesize findings from stages 1-3.
3. Run the Sound Check, then invoke Chorus for positioning dialogue.
4. Decide on the README's angle and structure.

## Graph Tool Usage

This is a voice and strategy phase — graph tools are **not** part of the standard workflow. If an architectural question arises during the jam (e.g., a participant asks about module structure), you may call `get_architecture` on demand via `execute_code`. See `references/graph-analysis.md` at the plugin root for the tool-executor 3-step workflow.

## Key Questions

- What's the hook/opening?
- What tone fits the project?
- What structure works best?
- What makes this README stand out?

## Output

Generate a brain-jam synthesis saved to `.claude/grfp/brain-jam.md`

## Next Stage

When complete, prompt user for `/claudikins-github-readme-for-perfectionists:pen-wielding`
