# Brain-Jam Engine Swap: Gemini → MiniMax (via m2-brainstorm)

**Date:** 2026-05-25
**Status:** Design approved, awaiting implementation plan
**Plugin:** `claudikins-grfp` (claudikins-github-readme-for-perfectionists)

## Summary

Replace the GRFP pipeline's Stage 4 brain-jam engine. The local `brain-jam` skill currently calls `gemini["gemini-brainstorm"]` through the `claudikins-tool-executor` plugin. Switch it to delegate to `m2-brainstorm:readme-brain-jam`, which runs MiniMax-M2.7-highspeed via a `uv run python brainstorm.py` CLI.

The orchestrator (`skills/grfp/SKILL.md`) and command (`commands/brain-jam.md`) keep referencing the bare name `brain-jam` — only the skill body changes. GRFP becomes a thin adapter; m2-brainstorm owns the brainstorming engine.

## Context

- **Current state:** `skills/brain-jam/SKILL.md` documents a Gemini-based brainstorm via the tool-executor's `gemini-brainstorm` tool. The supporting reference is `skills/brain-jam/references/brainstorm-gemini.md`.
- **Runtime gap:** in `~/.claude/settings.json`, `claudikins-tool-executor@claudikins-marketplace` is currently `false`. The existing Gemini path is non-functional until that plugin is re-enabled.
- **Replacement:** `m2-brainstorm:readme-brain-jam` lives at `/Users/kellen/.claude/plugins/marketplaces/m2-deep-research/.claude/plugins/m2-brainstorm/skills/readme-brain-jam/SKILL.md`. It already knows about GRFP staging files (`.claude/grfp/deep-dive.md`, `.claude/grfp/crystal-ball.md`), writes its synthesis to `.claude/grfp/brain-jam.md`, and prompts for `/claudikins-github-readme-for-perfectionists:pen-wielding` at handoff.
- **Plugin enablement:** `m2-brainstorm@m2-deep-research` is not in `enabledPlugins`. Users must enable it once for this swap to function at runtime.

## Goals

1. Stage 4 of the GRFP pipeline produces a `brain-jam.md` synthesis using MiniMax-M2.7-highspeed.
2. The orchestrator state machine (Phases 1–5 in `skills/grfp/SKILL.md`) requires zero changes.
3. The user gets a single clear instruction if the m2-brainstorm plugin is missing — no silent failures.
4. Brainstorm transcripts land in `.claude/grfp/` alongside other staging artifacts.

## Non-Goals

- Removing or modifying `claudikins-tool-executor` integration elsewhere in GRFP.
- Automating `~/.claude/settings.json` edits to enable m2-brainstorm.
- Adding retry logic or error wrapping around the MiniMax CLI — downstream surfaces its own failures.
- Rewriting any other GRFP stage (deep-dive, crystal-ball, think-tank, pen-wielding).

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│  claudikins-grfp plugin                                    │
│                                                            │
│  skills/grfp/SKILL.md (orchestrator)                       │
│         │                                                  │
│         │  Phase 3: "Invoke brain-jam"                     │
│         ▼                                                  │
│  skills/brain-jam/SKILL.md (ADAPTER — rewritten)           │
│    1. Check GRFP staging files exist                       │
│    2. Pre-instruct --output override                       │
│    3. Skill: m2-brainstorm:readme-brain-jam                │
│         │                                                  │
└─────────┼──────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────────┐
│  m2-brainstorm plugin (m2-deep-research marketplace)       │
│                                                            │
│  skills/readme-brain-jam/SKILL.md (ENGINE)                 │
│    1. Sound check (3 GRFP questions)                       │
│    2. Read .claude/grfp/deep-dive.md + crystal-ball.md     │
│    3. Build --claude-thoughts seed                         │
│    4. uv run python brainstorm.py                          │
│         --output .claude/grfp/brainstorm-transcript-*.json │
│    5. Read transcript, synthesise                          │
│    6. Write .claude/grfp/brain-jam.md                      │
│    7. Prompt /…:pen-wielding                               │
└────────────────────────────────────────────────────────────┘
```

Communication contract between the two plugins is **filesystem-based**:
- **GRFP → m2-brainstorm:** existence of `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md`.
- **m2-brainstorm → GRFP:** writing `.claude/grfp/brain-jam.md` (consumed by Phase 5 `pen-wielding`).

## File-Level Changes

### `skills/brain-jam/SKILL.md` (rewrite body)

- Frontmatter `name: brain-jam` preserved; `description` updated to reflect MiniMax-via-m2-brainstorm.
- New sections:
  1. **Preconditions** — verify `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` exist. Halt with phase-specific one-liner if missing.
  2. **Plugin availability** — note that this skill delegates to `m2-brainstorm:readme-brain-jam`. If the Skill invocation errors with "not found," surface the enablement one-liner.
  3. **Delegation contract** — explicit, quoted pre-instruction telling Claude to override readme-brain-jam's step 4 `--output` to `.claude/grfp/brainstorm-transcript-<ISO8601>.json`. State that everything else (sound check, seed building, synthesis, handoff prompt) is the downstream skill's responsibility.
  4. **Handoff** — single sentence noting that `readme-brain-jam` already prompts for `/claudikins-github-readme-for-perfectionists:pen-wielding`.

### `commands/brain-jam.md` (light edit)

- `description` frontmatter: replace "with Gemini" with "with MiniMax."
- Keep `skills: [brain-jam]` and the existing `allowed-tools` list (Skill is already present).
- Leave the "Graph Tool Usage" section as-is — it's an optional fallback and doesn't block the swap.

### `skills/brain-jam/references/brainstorm-gemini.md` (deprecation banner)

Prepend:
```markdown
> **DEPRECATED (2026-05-25).** This document describes the legacy `gemini-brainstorm`
> flow via `claudikins-tool-executor`. The active brain-jam now delegates to
> `m2-brainstorm:readme-brain-jam` (MiniMax-M2.7-highspeed). Kept for historical reference only.
```

### Files NOT changed

- `skills/grfp/SKILL.md` — Phase 3 still says "Invoke `brain-jam`."
- `.claude-plugin/plugin.json` — `gemini` keyword is now misleading but not load-bearing. Optional cleanup, not required.
- `skills/brain-jam/references/` directory layout.
- Any other GRFP skill or command.

## Failure Modes

| Mode | Detection | Behavior |
|------|-----------|----------|
| m2-brainstorm not enabled | Skill tool returns "not found" | Halt with: `m2-brainstorm plugin not enabled. Run /plugin → enable m2-brainstorm@m2-deep-research, then retry /brain-jam.` |
| Missing `.claude/grfp/deep-dive.md` | File-not-found check | Halt with: `Missing deep-dive output. Run /deep-dive first.` |
| Missing `.claude/grfp/crystal-ball.md` | File-not-found check | Halt with: `Missing crystal-ball output. Run /crystal-ball first.` |
| MiniMax CLI failure (network, API key, missing `uv`) | Non-zero exit from `uv run python brainstorm.py` | Adapter does NOT wrap. Downstream `readme-brain-jam` surfaces its own error. |
| `--output` override ignored | Transcript lands in `./.brainstorm/` instead of `.claude/grfp/` | Documentation-clarity bug in the adapter's pre-instruction. Fix by making the override more explicit. |
| Re-running `/brain-jam` | Idempotent | Timestamped transcript filenames avoid collision. `.claude/grfp/brain-jam.md` overwrites — last synthesis wins. |

## Edge Cases Out of Scope

- Migrating from a partial Gemini-era run mid-pipeline.
- Auto-enabling m2-brainstorm via settings.json edits.
- CI / multi-user environments without `uv` installed.

## Testing

Manual verification only — this is a plugin-content swap.

1. **Happy path:** Enable `m2-brainstorm@m2-deep-research`, run `/grfp` from a fresh repo end-to-end. Confirm:
   - `.claude/grfp/brainstorm-transcript-<timestamp>.json` is written by m2-brainstorm.
   - `.claude/grfp/brain-jam.md` is written by m2-brainstorm.
   - User is prompted for `/…:pen-wielding`.
2. **Disabled plugin:** With `m2-brainstorm` not enabled, run `/brain-jam`. Confirm the one-liner appears and no other action is attempted.
3. **Missing staging:** Delete `.claude/grfp/deep-dive.md`, run `/brain-jam`. Confirm the phase-specific one-liner appears.

No automated tests are added — the test surface is the user workflow.

## Open Questions

None. All clarifying decisions captured during brainstorming:
- Integration shape: delegate via Skill tool.
- Precondition handling: detect and instruct.
- Transcript path: `.claude/grfp/brainstorm-transcript-<timestamp>.json`.
- Old reference file: leave in place with deprecation banner.
