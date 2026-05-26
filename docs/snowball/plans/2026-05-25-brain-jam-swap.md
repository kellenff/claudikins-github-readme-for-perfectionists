# Brain-Jam Engine Swap Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use snowball:subagent-driven-development (recommended) or snowball:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert the GRFP plugin's `brain-jam` skill into a thin adapter that delegates to `m2-brainstorm:readme-brain-jam` (MiniMax-M2.7-highspeed) instead of calling Gemini through `claudikins-tool-executor`.

**Architecture:** GRFP keeps its Phase 3 orchestrator wiring untouched. The local `skills/brain-jam/SKILL.md` becomes an adapter that (1) checks for `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md`, (2) pre-instructs an `--output` path override pointing at `.claude/grfp/`, and (3) invokes `m2-brainstorm:readme-brain-jam` via the Skill tool. The two plugins communicate through filesystem artifacts only — no direct coupling.

**Tech Stack:** Markdown skill files, YAML frontmatter, Claude Code Skill tool delegation. No code compilation or unit-test suite — verification is manual (per the spec).

**Spec:** `docs/snowball/specs/2026-05-25-brain-jam-swap-design.md`

---

### Task 1: Rewrite `skills/brain-jam/SKILL.md` as the adapter

**Files:**
- Modify: `skills/brain-jam/SKILL.md` (full rewrite of body; preserve frontmatter `name`)

- [ ] **Step 1: Replace the entire file with the adapter body**

Write the following content. Note that the `name:` field stays as `brain-jam` (the orchestrator depends on this exact name); only the description and body change.

````markdown
---
name: brain-jam
description: "Use when determining project tone, voice, and marketing angle after deep-dive and crystal-ball phases. Delegates to m2-brainstorm:readme-brain-jam (MiniMax-M2.7-highspeed)."
---

# The Brain-Jam (GRFP adapter)

Stage 4 of the GRFP pipeline. This skill is a **thin adapter**: it validates GRFP staging files, pre-instructs an output-path override, then delegates the actual brainstorm to `m2-brainstorm:readme-brain-jam`.

The downstream engine runs MiniMax-M2.7-highspeed via `uv run python brainstorm.py`. It already knows about GRFP staging files, writes synthesis to `.claude/grfp/brain-jam.md`, and prompts for `/claudikins-github-readme-for-perfectionists:pen-wielding` at handoff. Do not duplicate that work here.

---

## Step 1: Verify GRFP staging files

Both files must exist in the current working directory:

- `.claude/grfp/deep-dive.md`
- `.claude/grfp/crystal-ball.md`

If `deep-dive.md` is missing, halt with:

> Missing deep-dive output. Run /deep-dive first.

If `crystal-ball.md` is missing, halt with:

> Missing crystal-ball output. Run /crystal-ball first.

Do not attempt to improvise context or skip these checks.

---

## Step 2: Verify the m2-brainstorm plugin is available

This skill delegates to `m2-brainstorm:readme-brain-jam`. If the Skill tool returns "skill not found" or an equivalent unavailable-skill error when you attempt to invoke it in Step 3, halt with:

> m2-brainstorm plugin not enabled. Run /plugin → enable m2-brainstorm@m2-deep-research, then retry /brain-jam.

Do NOT attempt to edit `~/.claude/settings.json` automatically. Plugin enablement is user-driven.

---

## Step 3: Delegation contract — invoke readme-brain-jam with an output override

Before invoking the downstream skill, lock in the transcript path. Compute an ISO8601 timestamp now and use it for the override:

```
.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

When you invoke `m2-brainstorm:readme-brain-jam` via the Skill tool, you MUST instruct the downstream execution to use that exact path for its `--output` flag in step 4 of `readme-brain-jam`. The readme-brain-jam default of `./.brainstorm/readme-angle-<timestamp>.json` is NOT acceptable for GRFP — staging artifacts must live under `.claude/grfp/`.

Everything else is the downstream skill's responsibility:

- The three-question Sound Check
- Reading `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` to build `--claude-thoughts`
- Running the MiniMax CLI
- Reading the transcript and synthesising
- Writing `.claude/grfp/brain-jam.md`
- Prompting the user for `/claudikins-github-readme-for-perfectionists:pen-wielding`

If the MiniMax CLI fails (network, API key, missing `uv`, missing `brainstorm.py`), let the downstream skill surface the error. Do not wrap, retry, or swallow it.

---

## Step 4: Handoff

`readme-brain-jam` already ends by prompting for `/claudikins-github-readme-for-perfectionists:pen-wielding`. Once delegation returns, this adapter is done — do not re-prompt or duplicate the handoff.
````

- [ ] **Step 2: Verify the rewritten file is well-formed**

Run: `head -3 skills/brain-jam/SKILL.md`
Expected: shows the YAML frontmatter opening (`---`, `name: brain-jam`, etc.)

Run: `grep -n "^## Step" skills/brain-jam/SKILL.md`
Expected: four `## Step` headings (Step 1 through Step 4).

- [ ] **Step 3: Commit**

```bash
git add skills/brain-jam/SKILL.md
git commit -m "refactor(brain-jam): rewrite as adapter delegating to m2-brainstorm

Skill body now validates GRFP staging files, instructs an --output
override into .claude/grfp/, then delegates to
m2-brainstorm:readme-brain-jam. Frontmatter name preserved so the
orchestrator's Phase 3 wiring is unchanged."
```

---

### Task 2: Update the `brain-jam` command description

**Files:**
- Modify: `commands/brain-jam.md` (line 3 only — the `description` frontmatter)

- [ ] **Step 1: Replace the description string**

Use Edit to replace exactly:

Old:
```
description: Stage 4 - Collaborate with Gemini to find the perfect README angle (voice/strategy phase; graph tools available on-demand for architectural questions)
```

New:
```
description: Stage 4 - Collaborate with MiniMax (via m2-brainstorm) to find the perfect README angle (voice/strategy phase; graph tools available on-demand for architectural questions)
```

Do NOT touch the `allowed-tools` list, the `skills` declaration, the body, or the "Graph Tool Usage" section. The `Skill` tool is already in `allowed-tools` and is what the rewritten skill needs.

- [ ] **Step 2: Verify only the description line changed**

Run: `git diff commands/brain-jam.md`
Expected: a single-line change on the `description:` frontmatter field, nothing else.

- [ ] **Step 3: Commit**

```bash
git add commands/brain-jam.md
git commit -m "docs(brain-jam): update command description to reflect MiniMax engine"
```

---

### Task 3: Prepend deprecation banner to the legacy Gemini reference

**Files:**
- Modify: `skills/brain-jam/references/brainstorm-gemini.md` (prepend, do not delete or replace existing content)

- [ ] **Step 1: Prepend the deprecation banner**

Use Edit. The current file's first line is `# Gemini Brainstorm Reference`. Replace exactly:

Old:
```
# Gemini Brainstorm Reference
```

New:
```
> **DEPRECATED (2026-05-25).** This document describes the legacy `gemini-brainstorm`
> flow via `claudikins-tool-executor`. The active brain-jam now delegates to
> `m2-brainstorm:readme-brain-jam` (MiniMax-M2.7-highspeed). Kept for historical reference only.

# Gemini Brainstorm Reference
```

This preserves the original heading and all content beneath it. Only the banner is added.

- [ ] **Step 2: Verify the banner is at the top**

Run: `head -5 skills/brain-jam/references/brainstorm-gemini.md`
Expected: the first line begins with `> **DEPRECATED (2026-05-25).**`, followed by the rest of the banner, a blank line, then `# Gemini Brainstorm Reference`.

- [ ] **Step 3: Commit**

```bash
git add skills/brain-jam/references/brainstorm-gemini.md
git commit -m "docs(brain-jam): mark Gemini reference deprecated after engine swap"
```

---

### Task 4: Manual verification

This task is verification, not code. There are no automated tests for a content swap — the spec mandates manual checks across three paths.

**Files:** none (test-of-record only)

- [ ] **Step 1: Happy-path verification**

Pre-conditions:
- Enable `m2-brainstorm@m2-deep-research` in `~/.claude/settings.json` (via the `/plugin` command).
- Confirm `~/.claude/plugins/marketplaces/m2-deep-research/brainstorm.py` exists.
- Have a fresh repo where `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` already exist (run `/deep-dive` and `/crystal-ball` first if not).

Run `/brain-jam` (or `/grfp` and let the state machine route to Phase 3).

Confirm all four of:
1. The Sound Check questions appear (from readme-brain-jam, not from the old gemini skill).
2. A file `.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json` is created.
3. A file `.claude/grfp/brain-jam.md` is created with the synthesis.
4. The session ends with a prompt for `/claudikins-github-readme-for-perfectionists:pen-wielding`.

If any one of these fails, capture the failure and fix the adapter before continuing. Common failure: transcript landing in `./.brainstorm/` instead of `.claude/grfp/` — this means the Step 3 override pre-instruction in Task 1 was not explicit enough.

- [ ] **Step 2: Disabled-plugin verification**

Disable `m2-brainstorm@m2-deep-research` (via `/plugin`). With `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` still present, run `/brain-jam`.

Expected output (verbatim):

> m2-brainstorm plugin not enabled. Run /plugin → enable m2-brainstorm@m2-deep-research, then retry /brain-jam.

No other action should be attempted. No file should be created.

Re-enable the plugin after this check.

- [ ] **Step 3: Missing-staging verification**

With `m2-brainstorm` enabled, delete (or rename) `.claude/grfp/deep-dive.md`. Run `/brain-jam`.

Expected output (verbatim):

> Missing deep-dive output. Run /deep-dive first.

No other action should be attempted. Restore the file after the check.

Repeat with `.claude/grfp/crystal-ball.md` removed; expected:

> Missing crystal-ball output. Run /crystal-ball first.

- [ ] **Step 4: Document verification results**

Append a verification note to the spec at the bottom (a new `## Verification Log` section), one line per check, with date and pass/fail. This closes the loop on the spec's `## Testing` section.

Example:
```markdown
## Verification Log

- 2026-05-26 — Happy path: PASS
- 2026-05-26 — Disabled plugin: PASS
- 2026-05-26 — Missing staging (deep-dive): PASS
- 2026-05-26 — Missing staging (crystal-ball): PASS
```

- [ ] **Step 5: Commit verification log**

```bash
git add docs/snowball/specs/2026-05-25-brain-jam-swap-design.md
git commit -m "docs(brain-jam): record manual verification results"
```

---

## Notes for the implementer

- **Do not modify** `skills/grfp/SKILL.md`, `.claude-plugin/plugin.json`, or any other GRFP skill/command. The spec lists them as out of scope.
- **Do not delete** `skills/brain-jam/references/brainstorm-gemini.md`. The user chose to keep it as historical context (with banner).
- **Do not enable** the m2-brainstorm plugin programmatically via settings.json edits. Plugin enablement is user-driven.
- The downstream `readme-brain-jam` skill source lives at `/Users/kellen/.claude/plugins/marketplaces/m2-deep-research/.claude/plugins/m2-brainstorm/skills/readme-brain-jam/SKILL.md` — useful to read when Task 4 Step 1 fails and you need to debug the integration.
- If you find the Step 3 override in Task 1 is being ignored at runtime (transcript lands in the default path), make the override pre-instruction more emphatic — e.g., bold the path, repeat it in two adjacent sentences, or put it in a fenced quote block.
