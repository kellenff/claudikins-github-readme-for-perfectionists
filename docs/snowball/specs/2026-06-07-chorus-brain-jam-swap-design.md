# Brain-Jam Engine Swap: m2-brainstorm → Chorus

**Date:** 2026-06-07
**Status:** Design approved, awaiting implementation plan
**Plugin:** `claudikins-grfp` (claudikins-github-readme-for-perfectionists)
**Predecessor:** `2026-05-26-critic-voice-in-grfp-design.md`

## Summary

Replace the GRFP pipeline's Stage 4 brain-jam engine. The local `brain-jam` skill currently delegates to `m2-brainstorm:readme-brain-jam` via the Skill tool. Rewrite it as a self-contained orchestrator that invokes the Chorus CLI directly.

Chorus stays generic; GRFP owns README-specific flow (sound check, seed building, cast config, transcript parsing, `brain-jam.md` rendering). This drops the `m2-brainstorm` marketplace dependency (plugin toggle, 75MB binary, `EXA_API_KEY` workaround) and consolidates on Chorus as the one brainstorm engine across plugins.

The orchestrator (`skills/grfp/SKILL.md`) keeps referencing the bare name `brain-jam` — only the skill body and supporting files change.

## Context

- **Current state:** `skills/brain-jam/SKILL.md` is a thin adapter that validates staging files, locks `--critique` / `--critic-temperature` / `--output` overrides, delegates to `m2-brainstorm:readme-brain-jam` via the Skill tool, then renders critique-aware `.claude/grfp/brain-jam.md` from the m2 transcript schema (`speaker`, snake_case critic fields, `synthesis_hint`).
- **Replacement:** Chorus plugin at `~/.claude/skills/chorus` (skills-directory symlink). CLI: `bin/chorus` → `deno run chorus.ts`. Transcript schema uses `participant`, `kind` (`speak` | `critique`), camelCase critic fields (`antiSteelman`, `dungExtension`, `critiqueAggregate`). No `synthesis_hint` field.
- **v2.1.0 behavior preserved:** Always-on critic third voice, 4-block `brain-jam.md` (Set List, Watch-Outs, Undefended Assumptions, Argument Map), FULL / PARTIAL / NO-CRITIQUE rendering modes, pen-wielding critique awareness.

## Goals

1. Stage 4 runs without `m2-brainstorm` marketplace plugin, binary, or `EXA_API_KEY` workaround.
2. Chorus is the single brainstorm engine across the operator's plugins.
3. Preserve v2.1.0 critique-aware output and pen-wielding integration.
4. Simpler Quick Start: Chorus symlink + `MINIMAX_API_KEY` + Deno (no 75MB binary install).

## Non-Goals

- Adding a `readme-brain-jam` skill upstream in Chorus.
- Multi-provider cast (MiniMax-only for now; cast file is swappable later).
- Automated Chorus plugin installation or settings.json edits.
- Backward compatibility with m2 transcript JSON.
- Modifying `skills/grfp/SKILL.md`, `skills/pen-wielding/SKILL.md`, or other GRFP stages beyond docs references.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  claudikins-grfp                                            │
│                                                             │
│  skills/grfp/SKILL.md (orchestrator — unchanged)            │
│         │  Phase 3: "Invoke brain-jam"                      │
│         ▼                                                   │
│  skills/brain-jam/SKILL.md (FULL ORCHESTRATOR — rewritten)  │
│    1. Verify staging files                                  │
│    2. Resolve Chorus CLI                                    │
│    3. Sound check (3 questions, one at a time)              │
│    4. Read deep-dive + crystal-ball → build --seed          │
│    5. Shell out to Chorus CLI                               │
│    6. Parse transcript → render brain-jam.md                │
│    7. Handoff to pen-wielding                               │
│                                                             │
│  skills/brain-jam/recipes/grfp-readme.json (NEW, iterable)  │
└──────────────────────────┬──────────────────────────────────┘
                           │ Bash: chorus --config … --seed …
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  chorus plugin (generic — no GRFP knowledge)                │
│    bin/chorus → deno run chorus.ts                          │
│    Writes transcript JSON to --output path                  │
└─────────────────────────────────────────────────────────────┘
```

Communication contract is **filesystem-based** (unchanged):

- **Input:** `.claude/grfp/deep-dive.md`, `.claude/grfp/crystal-ball.md`
- **Output:** `.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json`, `.claude/grfp/brain-jam.md`
- **Consumer:** `pen-wielding`

## Workflow steps

### Step 1: Preconditions

Both files must exist:

- `.claude/grfp/deep-dive.md`
- `.claude/grfp/crystal-ball.md`

Halt with the same phase-specific one-liners as today if either is missing.

### Step 2: Resolve Chorus CLI

```bash
CHORUS="$(command -v chorus 2>/dev/null || echo "$HOME/.claude/skills/chorus/bin/chorus")"
```

If `$CHORUS` is not executable, halt with:

> Chorus not found. Symlink the chorus plugin to ~/.claude/skills/chorus (see README), then retry /brain-jam.

Do not attempt to edit `~/.claude/settings.json` or install Chorus automatically.

### Step 3: Sound check

Ask the user, one question at a time:

1. **The "Killer" Feature:** What implementation detail are you proudest of?
2. **The "Pain" Point:** What 2 AM frustration does this solve?
3. **The Vibe:** Do you want "Technical Clarity" or "Organised Chaos"?

### Step 4: Build seed

Read `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md`. Compose `--seed` as: tech-stack summary + killer feature + pain point + vibe preference. Aim for 4–6 sentences with at least one concrete claim and one tension.

If staging files are missing content despite passing Step 1, ask the user inline for 2–3 sentences about what the project does.

### Step 5: Run Chorus CLI

Compute timestamp: `date +%Y%m%dT%H%M%S`. Transcript path:

```
.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

**Locked flags:**

| Flag | Value |
| --- | --- |
| `--config` | `skills/brain-jam/recipes/grfp-readme.json` |
| `--prompt` | `"What's the right angle for this README — tone, hook, and positioning?"` |
| `--seed` | Seed from Step 4 |
| `--critique` | (flag-only) |
| `--critic-temperature` | `0.3` |
| `--max-rounds` | `3` |
| `--output` | `.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json` |
| `--argdown-mode` | `lightweight` |

Use `--argdown-mode lightweight` to avoid Deno/JSR fetch failures that caused all-critic-unavailable runs in v2.1.0 dogfood (m2 era).

If the CLI fails (non-zero exit), surface stderr verbatim. Do not wrap, retry, or swallow.

### Step 6: Render brain-jam.md

Parse Chorus JSON natively. Required top-level fields: `turns`, `critiqueAggregate`. If missing after exit 0, halt with:

> Transcript shape invalid: \<field\> missing. Chorus engine may have changed contract.

**Schema mapping (Chorus → renderer):**

| m2 field (old) | Chorus field (new) |
| --- | --- |
| `speaker: "claude"` | `participant: "synth"`, `kind: "speak"` |
| `speaker: "pragmatist"` | `participant: "pragmatist"`, `kind: "speak"` |
| `speaker: "critic"` | `participant: "critic"`, `kind: "critique"` |
| `anti_steelman` | `antiSteelman` |
| `dung_extension` | `dungExtension` |
| `critique_aggregate` | `critiqueAggregate` |
| `assumptions[].speaker` | `assumptions[].participant` |
| `synthesis_hint` | *(absent)* — embed fixed instruction: "Option 3 MUST reference at least one idea that appears in the transcript but is in neither Option 1 nor Option 2." |

**Critic status classification** (unchanged logic):

- Filter `turns` where `kind == "critique"` and `participant == "critic"`.
- All `status == "ok"` → **FULL**
- Mix of `"ok"` and `"unavailable"` → **PARTIAL**
- All `"unavailable"` → **NO-CRITIQUE**

Render the same four blocks as v2.1.0 (Set List always; Watch-Outs / Undefended Assumptions / Argument Map in FULL or PARTIAL; NO-CRITIQUE replacement block otherwise). Update Watch-Outs section headers to match cast participant names (`synth` / `pragmatist`). Citation turn ids use Chorus participant names (e.g. `synth_r2`, `pragmatist_r3`, `critic_r2`).

### Step 7: Handoff

Prompt:

> brain-jam.md written to .claude/grfp/. Next stage: run `/claudikins-github-readme-for-perfectionists:pen-wielding`.

## Cast config (open for iteration)

**New file:** `skills/brain-jam/recipes/grfp-readme.json`

Ship a starting point structurally aligned with Chorus's `critic-on.json`: two MiniMax personas + critic, `critique: true`. Exact personas, model IDs, and temperatures are **DRAFT — open for iteration** after the first GRFP dogfood run. Do not treat as frozen.

Implementation seeds the file from `critic-on.json` as a baseline; model IDs and persona text may change without a spec revision.

## File-level changes

| File | Change |
| --- | --- |
| `skills/brain-jam/SKILL.md` | Full rewrite (Steps 1–7 above) |
| `skills/brain-jam/recipes/grfp-readme.json` | **New** — draft cast |
| `commands/brain-jam.md` | Description: Chorus instead of m2-brainstorm |
| `.claude-plugin/plugin.json` | Description string update |
| `README.md` | Quick Start + Requirements: Chorus install, drop m2 steps |
| `CHANGELOG.md` | BREAKING: m2 → Chorus |
| `skills/brain-jam/references/brainstorm-gemini.md` | Extend deprecation banner |

**Not changed:** `skills/grfp/SKILL.md`, `skills/pen-wielding/SKILL.md`, Chorus repo.

## Failure modes

| Mode | Detection | Behavior |
| --- | --- | --- |
| Chorus CLI not found | `$CHORUS` not executable | Install one-liner (Step 2) |
| Missing staging file | Step 1 file check | Phase-specific halt (unchanged) |
| `MINIMAX_API_KEY` missing | Chorus exit 1 | Surface Chorus stderr verbatim |
| Deno not installed | Launcher fails | Halt: Deno required (https://deno.land) |
| Chorus CLI non-zero | Network, API, config | Surface stderr verbatim |
| Transcript shape invalid | Missing required fields after exit 0 | Defensive halt (Step 6) |
| Critic partial / unavailable | Status classification | FULL / PARTIAL / NO-CRITIQUE (unchanged) |
| Re-run `/brain-jam` | Idempotent | Timestamped transcripts; `brain-jam.md` overwrites |

## Edge cases out of scope

- Migrating mid-pipeline from an m2-era transcript.
- Auto-installing Chorus.
- CI without Deno or MiniMax API key.
- Multi-provider cast.

## Testing

Manual verification only:

1. **Happy path:** Chorus installed, staging present, `/brain-jam` end-to-end → transcript + 4-block `brain-jam.md` + pen-wielding handoff.
2. **Missing Chorus:** Confirm install one-liner, no CLI attempted.
3. **Missing staging:** Confirm phase-specific halt.
4. **Sound check:** Three questions, one at a time, before CLI runs.

## Open questions

None blocking implementation. Cast config (`grfp-readme.json`) is explicitly marked open for iteration post-dogfood.
