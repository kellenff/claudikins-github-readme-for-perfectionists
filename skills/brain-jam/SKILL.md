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
