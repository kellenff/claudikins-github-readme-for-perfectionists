# Chorus Brain-Jam Swap — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use snowball:subagent-driven-development (recommended) or snowball:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the m2-brainstorm delegation in Stage 4 with a self-contained GRFP brain-jam skill that invokes the Chorus CLI directly, dropping the m2 marketplace dependency while preserving v2.1.0 critique-aware output.

**Architecture:** Documentation-only change to skill files, one new cast recipe JSON, and README/CHANGELOG updates. The rewritten `skills/brain-jam/SKILL.md` owns sound check, seed building, Chorus CLI invocation, native Chorus transcript parsing, and 4-block `brain-jam.md` rendering. No executable code. Verification is manual readthrough plus optional live `/brain-jam` run.

**Tech Stack:** Markdown skill files (`SKILL.md`), JSON cast config, Chorus CLI (`bin/chorus` → Deno), MiniMax API.

**Spec:** `docs/snowball/specs/2026-06-07-chorus-brain-jam-swap-design.md`

---

## File Structure

**Files to create:**

- `skills/brain-jam/recipes/grfp-readme.json` — draft cast (2 MiniMax personas + critic). Open for iteration post-dogfood.

**Files to modify:**

- `skills/brain-jam/SKILL.md` — full rewrite: self-contained orchestrator (Steps 1–7).
- `commands/brain-jam.md` — description, workflow text, add `Bash` to allowed-tools, remove `Skill`.
- `.claude-plugin/plugin.json` — version `3.0.0`, description references Chorus.
- `README.md` — Quick Start steps 3–4, Requirements section, mermaid C1 label.
- `CHANGELOG.md` — new `[3.0.0]` section (BREAKING).
- `skills/brain-jam/references/brainstorm-gemini.md` — extend deprecation banner.

**Files NOT touched:**

- `skills/grfp/SKILL.md` — orchestrator phases unchanged.
- `skills/pen-wielding/SKILL.md` — consumes same `brain-jam.md` block structure.
- Chorus repo — stays generic.

---

### Task 1: Create `skills/brain-jam/recipes/grfp-readme.json`

**Files:**
- Create: `skills/brain-jam/recipes/grfp-readme.json`

- [ ] **Step 1: Create the recipes directory and cast file**

Run:

```bash
mkdir -p skills/brain-jam/recipes
```

Write `skills/brain-jam/recipes/grfp-readme.json` with this exact content:

```json
{
  "_comment": "DRAFT — open for iteration. Personas, models, and temperatures will be tuned after first GRFP dogfood run.",
  "participants": [
    {
      "name": "synth",
      "model": "MiniMax-M3",
      "persona": "You are a senior engineer whose excitement is technical, not marketing. Build on the prior turn — find what's interesting and raise a new angle; don't just agree.",
      "temperature": 0.8
    },
    {
      "name": "pragmatist",
      "model": "MiniMax-M3",
      "persona": "You are a pragmatist focused on what devs actually need, skeptical of hype. Push back on shallow excitement. Concrete examples only.",
      "temperature": 0.5
    }
  ],
  "critique": true,
  "critic": {
    "name": "critic",
    "model": "MiniMax-M3",
    "persona": "",
    "temperature": 0.3
  }
}
```

Note: JSON does not support comments; if Chorus rejects `_comment`, delete that key — the spec marks the whole file as draft regardless.

- [ ] **Step 2: Validate JSON parses**

Run:

```bash
python3 -m json.tool skills/brain-jam/recipes/grfp-readme.json > /dev/null && echo "OK"
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add skills/brain-jam/recipes/grfp-readme.json
git commit -m "feat(brain-jam): add draft GRFP Chorus cast recipe"
```

---

### Task 2: Rewrite `skills/brain-jam/SKILL.md`

**Files:**
- Modify: `skills/brain-jam/SKILL.md` (replace entire file)

- [ ] **Step 1: Replace the file with the Chorus orchestrator**

Use the Write tool to replace `skills/brain-jam/SKILL.md` entirely with:

````markdown
---
name: brain-jam
description: "Use when determining project tone, voice, and marketing angle after deep-dive and crystal-ball phases. Runs a Chorus CLI dialogue with always-on critic and renders critique-aware brain-jam.md."
---

# The Brain-Jam (GRFP Stage 4)

Stage 4 of the GRFP pipeline. This skill is a **self-contained orchestrator**: it validates staging files, resolves the Chorus CLI, runs the three-question Sound Check, builds a seed from staging context, invokes Chorus, parses the transcript, renders `.claude/grfp/brain-jam.md`, and hands off to pen-wielding.

Always-on critic. Users who want a cheaper two-voice dialogue without GRFP staging can invoke the Chorus skill directly.

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

## Step 2: Resolve Chorus CLI

Run via the Bash tool:

```bash
CHORUS="$(command -v chorus 2>/dev/null || echo "$HOME/.claude/skills/chorus/bin/chorus")"
if [ ! -x "$CHORUS" ]; then echo "MISSING"; else echo "$CHORUS"; fi
```

If the output is `MISSING`, halt with:

> Chorus not found. Symlink the chorus plugin to ~/.claude/skills/chorus (see README), then retry /brain-jam.

If the launcher fails with `deno: command not found`, halt with:

> Deno required to run Chorus. Install Deno (https://deno.land), then retry /brain-jam.

Do NOT attempt to edit `~/.claude/settings.json` or install Chorus automatically.

---

## Step 3: Sound check

Ask the user, **one question at a time** (use AskUserQuestion or equivalent):

1. **The "Killer" Feature:** What implementation detail are you proudest of?
2. **The "Pain" Point:** What 2 AM frustration does this solve?
3. **The Vibe:** Do you want "Technical Clarity" or "Organised Chaos"?

Record all three answers before proceeding.

---

## Step 4: Build seed

Read `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` with the Read tool.

Compose the Chorus `--seed` string as: tech-stack summary (from deep-dive) + roadmap tension (from crystal-ball) + killer feature + pain point + vibe preference (from Sound Check). Aim for 4–6 sentences with at least one concrete claim and one tension.

If staging files are thin despite passing Step 1, ask the user inline for 2–3 sentences about what the project does.

---

## Step 5: Run Chorus CLI

Compute the timestamp NOW: run `date +%Y%m%dT%H%M%S` via Bash. Transcript path:

```
.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

Run Chorus with the resolved binary from Step 2:

```bash
"$CHORUS" \
  --config skills/brain-jam/recipes/grfp-readme.json \
  --prompt "What's the right angle for this README — tone, hook, and positioning?" \
  --seed "<seed from Step 4>" \
  --critique \
  --critic-temperature 0.3 \
  --max-rounds 3 \
  --argdown-mode lightweight \
  --output .claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

Substitute the actual timestamp and seed. Chorus prints the transcript path to stdout on success.

If the CLI exits non-zero, surface stderr verbatim. Do not wrap, retry, or swallow errors.

---

## Step 6: Read transcript and render brain-jam.md

When the CLI returns exit code 0, read the transcript JSON at the path from Step 5.

### Step 6.1: Parse and validate transcript shape

The transcript JSON must contain:

- `turns`: array of turn objects with `round`, `participant`, and `kind` fields
- `critiqueAggregate`: object — always required (Step 5 unconditionally passes `--critique`)

If any required top-level field is missing, halt with:

> Transcript shape invalid: <field> missing. Chorus engine may have changed contract.

Do not attempt to render a partial file.

### Step 6.2: Classify critic status

Iterate `turns` filtering by `kind == "critique"` and `participant == "critic"`. Each critic turn has `status` of `"ok"` or `"unavailable"`.

- If **all** critic turns have `status == "ok"`, set rendering mode to **FULL**.
- If **some** are `"ok"` and **some** are `"unavailable"`, set rendering mode to **PARTIAL**.
- If **all** critic turns have `status == "unavailable"`, set rendering mode to **NO-CRITIQUE**.

### Step 6.3: Render brain-jam.md based on mode

Write to `.claude/grfp/brain-jam.md` using the Write tool. Block 1 always renders; Blocks 2–4 depend on mode.

#### Block 1 — Set List (always rendered)

Synthesize three angles from speak turns (`kind == "speak"`). Citation turn ids use Chorus participant names: `synth_rN`, `pragmatist_rN`, `critic_rN`.

````markdown
## Set List

**Option 1: The "Deep Tech" Angle**
_Headline Idea:_ <one sentence>
_Focus:_ <one sentence>
_Cited from:_ <comma-separated turn ids like synth_r2, pragmatist_r3>

**Option 2: The "Pragmatic Solver" Angle**
_Headline Idea:_ <one sentence>
_Focus:_ <one sentence>
_Cited from:_ <comma-separated turn ids>

**Option 3: The Synthesis (Recommended)**
_Headline Idea:_ <one sentence>
_Tone:_ <one sentence>
_Cited from:_ <comma-separated turn ids; may include critic_rN>
````

**Quality test:** Option 3 must reference at least one idea that appears in the transcript but is in neither Option 1 nor Option 2.

#### Block 2 — Watch-Outs (FULL or PARTIAL mode only)

Aggregate `antiSteelman` from all `status == "ok"` critic turns. Group by participant key (`synth`, `pragmatist`); preserve round order.

````markdown
## Watch-Outs (anti-steelman per voice)

**Where the "synth" voice was weakest:**
- Round 1: "<antiSteelman.synth verbatim>"
- Round 2: "<antiSteelman.synth verbatim>"

**Where the "pragmatist" voice was weakest:**
- Round 1: "<antiSteelman.pragmatist verbatim>"
- Round 2: "<antiSteelman.pragmatist verbatim>"
````

In **PARTIAL** mode, append:

> *Critic was unavailable for round(s) <comma-separated round numbers>. Coverage is partial.*

#### Block 3 — Undefended Assumptions (FULL or PARTIAL mode only)

Aggregate `assumptions[].premise` where `argued_for == false` from all `status == "ok"` critic turns. Deduplicate case-insensitively; preserve first-occurrence order. Prefix with `participant`.

````markdown
## Undefended Assumptions

Consider the following hidden assumptions in the research:

- (synth) "<premise>"
- (pragmatist) "<premise>"
````

In **PARTIAL** mode, append the same partial-coverage footnote as Block 2.

#### Block 4 — Argument Map (FULL or PARTIAL mode only)

Use the **last** `status == "ok"` critic turn's `argdown` source and `dungExtension` partition.

**FULL mode template:**

`````markdown
## Argument Map (round N)

```argdown
<argdown source verbatim>
```

**Surviving arguments (IN):** <comma-separated labels from dungExtension.in>
**Defeated arguments (OUT):** <comma-separated labels from dungExtension.out>
**Undecided (UNDEC):** <comma-separated labels from dungExtension.undec, or "_(none)_">
`````

**PARTIAL mode template:** same structure with heading `## Argument Map (round N — final critic-ok round)`.

Substitute `N` with the actual round number.

#### NO-CRITIQUE mode replacement

Skip Blocks 2–4. Append after Block 1:

````markdown
## Critique unavailable

The critic didn't return any output. This is NOT a terminating error.

Errors were: <semicolon-separated "round N: <error>" per critic turn>.
````

### Step 6.4: Confirm file written

Run `ls -la .claude/grfp/brain-jam.md` via Bash. Do not Read it back unless rendering failed.

---

## Step 7: Handoff

Prompt the user:

> brain-jam.md written to .claude/grfp/. Next stage: run `/claudikins-github-readme-for-perfectionists:pen-wielding`.

This skill is done.
````

- [ ] **Step 2: Verify no m2 references remain**

Run:

```bash
grep -n 'm2-brainstorm\|readme-brain-jam\|synthesis_hint\|anti_steelman\|dung_extension\|critique_aggregate\|speaker ==' skills/brain-jam/SKILL.md && echo "FAIL" || echo "OK"
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add skills/brain-jam/SKILL.md
git commit -m "feat(brain-jam): rewrite as self-contained Chorus orchestrator"
```

---

### Task 3: Update `commands/brain-jam.md`

**Files:**
- Modify: `commands/brain-jam.md`

- [ ] **Step 1: Update frontmatter and workflow body**

Replace the entire file with:

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add commands/brain-jam.md
git commit -m "docs(brain-jam): update command for Chorus engine"
```

---

### Task 4: Bump version in `.claude-plugin/plugin.json`

**Files:**
- Modify: `.claude-plugin/plugin.json`

- [ ] **Step 1: Update version and description**

Change `"version": "2.1.0"` to `"version": "3.0.0"`.

Change the description string to:

```
Five rituals for README perfection: /deep-dive the codebase, /crystal-ball the roadmap, /brain-jam with Chorus, enter the /think-tank, then /pen-wielding to write.
```

- [ ] **Step 2: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "chore: release v3.0.0 (Chorus brain-jam engine)"
```

---

### Task 5: Update `README.md`

**Files:**
- Modify: `README.md` (Quick Start, mermaid label, Requirements)

- [ ] **Step 1: Replace Quick Start steps 3–5 and explanatory paragraph**

Replace lines 44–54 (from `# 3. Enable the brainstorm engine` through the Step 4 paragraph) with:

````markdown
# 3. Install Chorus (brainstorm engine)
ln -sf /path/to/chorus ~/.claude/skills/chorus   # skills-directory plugin symlink
export MINIMAX_API_KEY=your-key-here             # or set in ~/.claude/skills/chorus/.env

# 4. Install Deno (Chorus runtime)
curl -fsSL https://deno.land/install.sh | sh

# 5. Run the pipeline
/claudikins-github-readme-for-perfectionists:grfp
````

Replace the old Step 4 paragraph with:

```markdown
Step 3 is the easy-to-miss one. The `brain-jam` stage shells out to the Chorus CLI via Deno. Without the symlink and `MINIMAX_API_KEY`, Stage 4 halts with a one-line install hint. Deno is required — Chorus does not ship a standalone binary.
```

- [ ] **Step 2: Update mermaid Stage 3 label**

Change line 69 from:

```
C -.- C1["Voice: Claude + MiniMax + critic"]
```

to:

```
C -.- C1["Voice: Chorus synth + pragmatist + critic"]
```

- [ ] **Step 3: Replace Requirements section (lines 202–209)**

Replace with:

```markdown
## Requirements

- Claude Code 1.0 or later
- Chorus plugin symlinked at `~/.claude/skills/chorus` (required for `/brain-jam`)
- Deno (required — Chorus launcher shells out to `deno run`)
- `MINIMAX_API_KEY` in environment or `~/.claude/skills/chorus/.env`
- `claudikins-tool-executor` plugin (optional; enables graph-analysis tools for `/deep-dive` and `/crystal-ball`)
```

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): update Quick Start and requirements for Chorus"
```

---

### Task 6: Add `[3.0.0]` to `CHANGELOG.md`

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Insert new section after line 7 (before `[2.0.0]`)**

```markdown
## [3.0.0] - 2026-06-07

### Changed

- **BREAKING — brain-jam engine swap**: Stage 4 (`brain-jam`) now invokes the Chorus CLI directly instead of delegating to `m2-brainstorm:readme-brain-jam`. Users must install the Chorus skills-directory plugin and set `MINIMAX_API_KEY`; the m2-brainstorm marketplace plugin, 75MB binary, and `EXA_API_KEY` workaround are no longer required.
- `skills/brain-jam/SKILL.md` rewritten as a self-contained orchestrator — Sound Check, seed building, Chorus CLI invocation, native transcript parsing, and critique-aware `brain-jam.md` rendering all live in GRFP.
- New cast recipe at `skills/brain-jam/recipes/grfp-readme.json` (draft — open for iteration).
- `commands/brain-jam.md` updated: Chorus description, `Bash`/`Write` in allowed-tools, `Skill` removed.
- README Quick Start and Requirements updated for Chorus + Deno prerequisites.

### Removed

- All runtime dependency on `m2-brainstorm@m2-deep-research` and `~/.config/m2-brainstorm/bin/m2-brainstorm`.

### Migration

Install Chorus before running Stage 4:

```bash
ln -sf /path/to/chorus ~/.claude/skills/chorus
export MINIMAX_API_KEY=your-key-here
```

Design spec: `docs/snowball/specs/2026-06-07-chorus-brain-jam-swap-design.md`
Implementation plan: `docs/snowball/plans/2026-06-07-chorus-brain-jam-swap.md`

---
```

- [ ] **Step 2: Commit**

```bash
git add CHANGELOG.md
git commit -m "docs(changelog): add v3.0.0 Chorus brain-jam swap release notes"
```

---

### Task 7: Extend deprecation banner in `brainstorm-gemini.md`

**Files:**
- Modify: `skills/brain-jam/references/brainstorm-gemini.md` (lines 1–3)

- [ ] **Step 1: Replace the deprecation banner**

Replace lines 1–3 with:

```markdown
> **DEPRECATED (2026-05-25).** This document describes the legacy `gemini-brainstorm`
> flow via `claudikins-tool-executor`. Superseded first by `m2-brainstorm:readme-brain-jam`
> (2026-05-26), then by Chorus CLI invocation from GRFP `brain-jam` (2026-06-07).
> Kept for historical reference only.
```

- [ ] **Step 2: Commit**

```bash
git add skills/brain-jam/references/brainstorm-gemini.md
git commit -m "docs(brain-jam): extend gemini reference deprecation for Chorus"
```

---

### Task 8: Manual verification

**Files:** none (readthrough + optional live run)

- [ ] **Step 1: Grep sweep for stale m2 references**

Run:

```bash
grep -rn 'm2-brainstorm\|readme-brain-jam\|m2-deep-research' \
  skills/ commands/ README.md CHANGELOG.md .claude-plugin/ \
  --include='*.md' --include='*.json' \
  | grep -v 'docs/snowball/' \
  | grep -v 'brainstorm-gemini.md' \
  | grep -v CHANGELOG.md \
  || echo "OK"
```

Expected: only CHANGELOG historical mentions and brainstorm-gemini.md historical text. Any hits in active skill/command files are failures to fix before shipping.

- [ ] **Step 2: Readthrough checklist**

Confirm by reading the modified files:

1. `skills/brain-jam/SKILL.md` — Steps 1–7 match spec; Chorus schema field names (`antiSteelman`, `dungExtension`, `critiqueAggregate`, `kind`, `participant`).
2. `skills/brain-jam/recipes/grfp-readme.json` — valid JSON, two personas + critic.
3. `commands/brain-jam.md` — `Bash` and `Write` present; no `Skill` tool.
4. `README.md` — no m2 install steps in Quick Start or Requirements.
5. `.claude-plugin/plugin.json` — version `3.0.0`.

- [ ] **Step 3: Optional live happy-path run**

Prerequisites: Chorus symlinked, `MINIMAX_API_KEY` set, Deno installed, `.claude/grfp/deep-dive.md` and `crystal-ball.md` present.

Run `/claudikins-github-readme-for-perfectionists:brain-jam` (or `/brain-jam`).

Expected:

- Three Sound Check questions, one at a time.
- Transcript at `.claude/grfp/brainstorm-transcript-<timestamp>.json`.
- `brain-jam.md` with Set List (+ critique blocks if critic succeeded).
- Handoff prompt for pen-wielding.

- [ ] **Step 4: Optional missing-Chorus test**

Temporarily rename `~/.claude/skills/chorus/bin/chorus`, run `/brain-jam`.

Expected: install one-liner halt; no CLI attempted. Restore binary afterward.

- [ ] **Step 5: Final commit if any fixups from verification**

```bash
git status
# If clean, verification complete.
```

---

## Spec Coverage (self-review)

| Spec requirement | Task |
| --- | --- |
| Drop m2 dependency | Tasks 2, 5, 6 |
| Self-contained GRFP orchestrator | Task 2 |
| Sound check (3 questions) | Task 2 Step 1 (Step 3 in skill) |
| CLI resolution order | Task 2 (Step 2 in skill) |
| GRFP cast file (iterable) | Task 1 |
| Locked CLI flags incl. `--argdown-mode lightweight` | Task 2 (Step 5 in skill) |
| Native Chorus schema parsing | Task 2 (Step 6 in skill) |
| 4-block brain-jam.md / FULL-PARTIAL-NO-CRITIQUE | Task 2 (Step 6.3) |
| README Quick Start update | Task 5 |
| CHANGELOG BREAKING entry | Task 6 |
| command + plugin.json updates | Tasks 3, 4 |
| Manual testing | Task 8 |
| pen-wielding unchanged | (no task — by design) |

No placeholders. All tasks have exact file paths and content.
