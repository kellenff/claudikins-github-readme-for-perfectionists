---
name: brain-jam
description: "Use when determining project tone, voice, and marketing angle after deep-dive and crystal-ball phases. Delegates to m2-brainstorm:readme-brain-jam (MiniMax-M2.7-highspeed)."
---

# The Brain-Jam (GRFP adapter)

Stage 4 of the GRFP pipeline. This skill is a **thin adapter**: it validates GRFP staging files, pre-instructs an output-path override, then delegates the actual brainstorm to `m2-brainstorm:readme-brain-jam`.

The downstream engine runs MiniMax-M2.7-highspeed. It knows about GRFP staging files and writes the transcript JSON to the path the adapter computes. This adapter takes over the synthesis-and-write step: it reads the transcript, renders the critique-aware `.claude/grfp/brain-jam.md`, and prompts for `/claudikins-github-readme-for-perfectionists:pen-wielding` itself.

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

*(This step defines the halt response if the m2-brainstorm plugin is unavailable. The actual detection happens during the Skill tool invocation in Step 3.)*

This skill delegates to `m2-brainstorm:readme-brain-jam`. If the Skill tool returns "skill not found" or an equivalent unavailable-skill error, halt with:

> m2-brainstorm plugin not enabled. Run /plugin → enable m2-brainstorm@m2-deep-research, then retry /brain-jam.

Do NOT attempt to edit `~/.claude/settings.json` automatically. Plugin enablement is user-driven.

---

## Step 3: Delegation contract — invoke readme-brain-jam with locked overrides

Compute the timestamp NOW (before invoking the Skill tool): run `date +%Y%m%dT%H%M%S` via the Bash tool. For example, a run at 2:23pm on 26 May 2026 yields `20260526T142300`. The full transcript path is then `.claude/grfp/brainstorm-transcript-20260526T142300.json` (substitute your computed timestamp).

The locked-in transcript path format is:

```
.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

**Required overrides when invoking `m2-brainstorm:readme-brain-jam`:**

| Flag                   | Value                                                  | Why locked                                                              |
| ---------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------- |
| `--output`             | `.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json` | GRFP staging artifacts must live under `.claude/grfp/`.             |
| `--critique`           | (flag-only, no value)                                  | GRFP always uses the third voice. Users who want the cheaper two-voice dialogue can invoke `/m2-brainstorm:readme-brain-jam` directly. |
| `--critic-temperature` | `0.3`                                                  | Matches m2-brainstorm spec default; analytical extraction favors deterministic output over creative variation. |

**Required behavior override:**

When reading the downstream `readme-brain-jam` skill, treat its Step 6 (Set List synthesis) and Step 7 (write `brain-jam.md`, prompt for `/pen-wielding`) as overridden. Stop following downstream instructions after the CLI returns; this adapter's Step 4 performs the equivalent work with critique-aware rendering.

Everything else is the downstream skill's responsibility:

- The three-question Sound Check
- Reading `.claude/grfp/deep-dive.md` and `.claude/grfp/crystal-ball.md` to build `--claude-thoughts`
- Running the MiniMax CLI with the locked overrides above plus its own `--prompt`, `--claude-thoughts`, and `--max-rounds` arguments
- Returning the transcript path

If the MiniMax CLI fails (network, API key, missing binary), let the downstream skill surface the error. Do not wrap, retry, or swallow it.

---

## Step 4: Read transcript and render brain-jam.md

When the downstream CLI returns with exit code 0, the transcript JSON exists at the path you computed in Step 3. Read it with the Read tool.

### Step 4.1: Parse and validate transcript shape

The transcript JSON must contain:

- `turns`: array of turn objects, each with `round`, `speaker`, and content fields
- `synthesis_hint`: string (used as a synthesis seed)
- `critique_aggregate`: object — always required (the adapter unconditionally passes `--critique`)

If any of these top-level fields are missing, halt with:

> Transcript shape invalid: <field> missing. The m2-brainstorm engine may have changed contract. Run `/m2-brainstorm:readme-brain-jam` directly to verify, then report upstream.

This is the Layer 3 defensive halt. Do not attempt to render a partial file.

### Step 4.2: Classify critic status

Iterate `turns` filtering by `speaker == "critic"`. Each critic turn has a `status` field of `"ok"` or `"unavailable"`.

- If **all** critic turns have `status == "ok"`, set rendering mode to **FULL**.
- If **some** critic turns are `"ok"` and **some** are `"unavailable"`, set rendering mode to **PARTIAL**.
- If **all** critic turns have `status == "unavailable"`, set rendering mode to **NO-CRITIQUE**.

### Step 4.3: Render brain-jam.md based on mode

Write the rendered markdown to `.claude/grfp/brain-jam.md` using the Write tool. The file always starts with Block 1 (Set List). Blocks 2-4 depend on mode.

#### Block 1 — Set List (always rendered)

Synthesize three angles from the dialogue. Each angle cites the turns it emerged from; the synthesis option may cite a `critic_rN` turn when the critic's steelman influenced it. Template:

````markdown
## Set List

**Option 1: The "Deep Tech" Angle**
_Headline Idea:_ <one sentence>
_Focus:_ <one sentence>
_Cited from:_ <comma-separated turn ids like claude_r2, pragmatist_r3>

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

Aggregate `anti_steelman` fields from all `status == "ok"` critic turns. Group by speaker; preserve round order.

````markdown
## Watch-Outs (anti-steelman per voice)

**Where the "deep tech" voice was weakest:**
- Round 1: "<anti_steelman.claude verbatim>"
- Round 2: "<anti_steelman.claude verbatim>"
- ... (one bullet per critic turn that has the claude anti-steelman)

**Where the "pragmatist" voice was weakest:**
- Round 1: "<anti_steelman.pragmatist verbatim>"
- Round 2: "<anti_steelman.pragmatist verbatim>"
- ... (one bullet per critic turn that has the pragmatist anti-steelman)
````

In **PARTIAL** mode, append this footnote line at the end of Block 2:

> *Critic was unavailable for round(s) <comma-separated round numbers>. Coverage is partial.*

#### Block 3 — Undefended Assumptions (FULL or PARTIAL mode only)

Aggregate `assumptions[].premise` where `argued_for == false`, across all `status == "ok"` critic turns. Deduplicate by case-insensitive string match; preserve first-occurrence order.

````markdown
## Undefended Assumptions

Consider the following hidden assumptions in the research:

- (claude) "<premise>"
- (pragmatist) "<premise>"
- ... (one bullet per unique undefended premise, prefixed with speaker)
````

In **PARTIAL** mode, append this footnote line at the end of Block 3:

> *Critic was unavailable for round(s) <comma-separated round numbers>. Coverage is partial.*

#### Block 4 — Argument Map (FULL or PARTIAL mode only)

Use the **last** `status == "ok"` critic turn's `argdown` source. In FULL mode this is the final round; in PARTIAL mode it may be an earlier round.

**FULL mode template:**

`````markdown
## Argument Map (round N)

```argdown
<argdown source verbatim from critic_rN.argdown>
```

**Surviving arguments (IN):** <comma-separated argument labels from dung_extension.in>
**Defeated arguments (OUT):** <comma-separated argument labels from dung_extension.out>
**Undecided (UNDEC):** <comma-separated labels from dung_extension.undec, or "_(none)_">
`````

**PARTIAL mode template:**

`````markdown
## Argument Map (round N — final critic-ok round)

```argdown
<argdown source verbatim from critic_rN.argdown>
```

**Surviving arguments (IN):** <comma-separated argument labels from dung_extension.in>
**Defeated arguments (OUT):** <comma-separated argument labels from dung_extension.out>
**Undecided (UNDEC):** <comma-separated labels from dung_extension.undec, or "_(none)_">
`````

Substitute `N` with the actual round number.

#### NO-CRITIQUE mode replacement

In **NO-CRITIQUE** mode, skip Blocks 2, 3, and 4 entirely. Append this single block after Block 1:

````markdown
## Critique unavailable

The critic didn't return any output. This is NOT a terminating error.

Errors were: <semicolon-separated, one entry per critic turn formatted as "round N: \<error from critic_rN.error\>">.
````

(Substitute the actual per-round errors from each critic turn's `error` field.)

### Step 4.4: Confirm file written

Run `ls -la .claude/grfp/brain-jam.md` via the Bash tool to confirm the file exists. Do not Read it back unless rendering failed.

---

## Step 5: Handoff

The adapter has written `.claude/grfp/brain-jam.md` itself in Step 4. Do NOT re-invoke any rendering. Prompt the user:

> brain-jam.md written to .claude/grfp/. Next stage: run `/claudikins-github-readme-for-perfectionists:pen-wielding`.

This adapter is done.
