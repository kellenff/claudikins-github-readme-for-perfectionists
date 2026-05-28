# Critic Voice in GRFP Brain-Jam — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use snowball:subagent-driven-development (recommended) or snowball:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire the m2-brainstorm critic third voice into the GRFP pipeline as an always-on capability, restructure `brain-jam.md` to surface critique signal, and make pen-wielding aware of the new sections.

**Architecture:** Documentation-only change to two skill files. The local `brain-jam` adapter gains two flag overrides and takes over the synthesis-and-write step from upstream; `pen-wielding` gains a pre-write critique check. No new files, no new reference docs, no automated tests — verification is manual readthrough plus optional live run.

**Tech Stack:** Markdown skill files (`SKILL.md`). Frontmatter unchanged. The skills are interpreted by Claude Code at runtime; they are not executed code.

**Spec:** `docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md`

---

## File Structure

**Files to modify:**

- `skills/brain-jam/SKILL.md` — adapter; Step 3 (Delegation contract) grows, new Step 4 (Render brain-jam.md) inserted, existing Step 4 (Handoff) renumbered to Step 5.
- `skills/pen-wielding/SKILL.md` — Step 1 item 3 reworded, new Step 3.5 inserted, two new lines appended to Step 8 Brain-Jam Verification.

**Files NOT touched:**

- `commands/brain-jam.md` — the command calls the skill by name; new behavior is invisible at the command surface.
- `.claude-plugin/plugin.json` — no version bump in this spec (the behavior change is a documentation refresh, not a breaking interface).
- All other skills (`deep-dive`, `crystal-ball`, `think-tank`, `grfp`) — they produce no inputs to the critic and consume no outputs from it.
- `skills/brain-jam/references/brainstorm-gemini.md` — already marked deprecated in a prior commit; leave as-is.

**Why no `references/` files for new content:** the new Step 3.5 in pen-wielding is short and tied to one specific artifact (`brain-jam.md`); inlining keeps the cognitive load on the writer at one file. The new Step 4 in brain-jam contains template literals that the agent instantiates at runtime — those belong in the SKILL.md itself, not a reference.

---

## Task 1: Replace `skills/brain-jam/SKILL.md` Step 3 (Delegation contract)

**Files:**
- Modify: `skills/brain-jam/SKILL.md` (Step 3 section, lines 45-66 in current file)

**Context:** Step 3 currently locks one override (`--output`). After this task it locks three flags and disowns the downstream's synthesis step.

- [ ] **Step 1: Read the current Step 3 verbatim**

Run: `sed -n '45,66p' skills/brain-jam/SKILL.md`

Expected output: the current "Step 3: Delegation contract" section starting with `## Step 3: Delegation contract — invoke readme-brain-jam with an output override`.

- [ ] **Step 2: Replace Step 3 with the new version**

Use the Edit tool. Replace the entire current Step 3 (from the `## Step 3:` heading through the closing `---` separator before "## Step 4: Handoff") with:

```markdown
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
- Running the MiniMax CLI with the four flags above
- Returning the transcript path

If the MiniMax CLI fails (network, API key, missing binary), let the downstream skill surface the error. Do not wrap, retry, or swallow it.

---
```

- [ ] **Step 3: Verify the edit applied cleanly**

Run: `sed -n '45,90p' skills/brain-jam/SKILL.md`

Expected: the new "Step 3: Delegation contract — invoke readme-brain-jam with locked overrides" content above. Confirm the table renders three rows (output, critique, critic-temperature) and the "Required behavior override" paragraph is present.

- [ ] **Step 4: Commit**

```bash
git add skills/brain-jam/SKILL.md
git commit -m "feat(brain-jam): lock --critique and --critic-temperature 0.3 overrides

Step 3 of the GRFP brain-jam adapter now mandates the m2-brainstorm
critic third voice on every run, and instructs the agent to halt
downstream synthesis (Step 6/7) so the adapter can render brain-jam.md
with critique signal in Task 2.

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

---

## Task 2: Insert `skills/brain-jam/SKILL.md` Step 4 (Render brain-jam.md)

**Files:**
- Modify: `skills/brain-jam/SKILL.md` (new section inserted after Step 3, before existing "Step 4: Handoff")

**Context:** This task introduces the largest single addition: the rendering rules for the restructured `brain-jam.md`. The section is declarative — it shows the agent the four block templates and the conditional logic for partial / total critic unavailability.

- [ ] **Step 1: Read the current end of Step 3 and start of "Step 4: Handoff"**

Run: `grep -n "^## Step" skills/brain-jam/SKILL.md`

Expected: confirms Step 3 ends and "Step 4: Handoff" begins immediately after. Note the line number of "## Step 4: Handoff".

- [ ] **Step 2: Insert the new Step 4 (Render) BEFORE the existing "Step 4: Handoff"**

Use the Edit tool to insert the following section. The `old_string` should be the literal line `## Step 4: Handoff` (the existing heading), and `new_string` should be the entire new Step 4 below followed by `## Step 5: Handoff` (the renamed old heading).

New Step 4 content (paste verbatim):

```markdown
## Step 4: Read transcript and render brain-jam.md

When the downstream CLI returns with exit code 0, the transcript JSON exists at the path you computed in Step 3. Read it with the Read tool.

### Step 4.1: Parse and validate transcript shape

The transcript JSON must contain:

- `turns`: array of turn objects, each with `round`, `speaker`, and content fields
- `synthesis_hint`: string (used as a synthesis seed)
- `critique_aggregate`: object (present when `--critique` was passed)

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

```markdown
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
```

**Quality test:** Option 3 must reference at least one idea that appears in the transcript but is in neither Option 1 nor Option 2.

#### Block 2 — Watch-Outs (FULL or PARTIAL mode only)

Aggregate `anti_steelman` fields from all `status == "ok"` critic turns. Group by speaker; preserve round order.

```markdown
## Watch-Outs (anti-steelman per voice)

**Where the "deep tech" voice was weakest:**
- Round 1: "<anti_steelman.claude verbatim>"
- Round 2: "<anti_steelman.claude verbatim>"
- ... (one bullet per critic turn that has the claude anti-steelman)

**Where the "pragmatist" voice was weakest:**
- Round 1: "<anti_steelman.pragmatist verbatim>"
- Round 2: "<anti_steelman.pragmatist verbatim>"
- ... (one bullet per critic turn that has the pragmatist anti-steelman)
```

In **PARTIAL** mode, append this footnote line at the end of Block 2:

> *Critic was unavailable for round(s) <comma-separated round numbers>. Coverage is partial.*

#### Block 3 — Undefended Assumptions (FULL or PARTIAL mode only)

Aggregate `assumptions[].premise` where `argued_for == false`, across all `status == "ok"` critic turns. Deduplicate by case-insensitive string match; preserve first-occurrence order.

```markdown
## Undefended Assumptions

Consider the following hidden assumptions in the research:

- (claude) "<premise>"
- (pragmatist) "<premise>"
- ... (one bullet per unique undefended premise, prefixed with speaker)
```

In **PARTIAL** mode, append this footnote line at the end of Block 3:

> *Critic was unavailable for round(s) <comma-separated round numbers>. Coverage is partial.*

#### Block 4 — Argument Map (FULL or PARTIAL mode only)

Use the **last** `status == "ok"` critic turn's `argdown` source. In FULL mode this is the final round; in PARTIAL mode it may be an earlier round.

```markdown
## Argument Map (round N — final critic-ok round)

\`\`\`argdown
<argdown source verbatim from critic_rN.argdown>
\`\`\`

**Surviving arguments (IN):** <comma-separated argument labels from dung_extension.in>
**Defeated arguments (OUT):** <comma-separated argument labels from dung_extension.out>
**Undecided (UNDEC):** <comma-separated labels from dung_extension.undec, or "_(none)_">
```

In **FULL** mode, the heading is `## Argument Map (round N)` where N is the final round.
In **PARTIAL** mode, the heading is `## Argument Map (round N — final critic-ok round)`.

#### NO-CRITIQUE mode replacement

In **NO-CRITIQUE** mode, skip Blocks 2, 3, and 4 entirely. Append this single block after Block 1:

```markdown
## Critique unavailable

The critic didn't return any output. This is NOT a terminating error.

Errors were: round 1: <error from critic_r1.error>; round 2: <error from critic_r2.error>; round 3: <error from critic_r3.error>.
```

(Substitute the actual per-round errors from each critic turn's `error` field.)

### Step 4.4: Confirm file written

Run `ls -la .claude/grfp/brain-jam.md` via the Bash tool to confirm the file exists. Do not Read it back unless rendering failed.

---
```

Then immediately after the closing `---` of the new Step 4, the existing "Step 4: Handoff" heading must be renamed to "Step 5: Handoff" and its body updated to reflect that brain-jam.md is now adapter-owned.

Replace the existing handoff section (currently lines ~70-72 of the file) with:

```markdown
## Step 5: Handoff

The adapter has written `.claude/grfp/brain-jam.md` itself in Step 4. Do NOT re-invoke any rendering. Prompt the user:

> brain-jam.md written to .claude/grfp/. Next stage: run `/claudikins-github-readme-for-perfectionists:pen-wielding`.

This adapter is done.
```

- [ ] **Step 3: Verify the file structure is correct**

Run: `grep -n "^## Step" skills/brain-jam/SKILL.md`

Expected output (in order):

```
## Step 1: Verify GRFP staging files
## Step 2: Verify the m2-brainstorm plugin is available
## Step 3: Delegation contract — invoke readme-brain-jam with locked overrides
## Step 4: Read transcript and render brain-jam.md
## Step 5: Handoff
```

Plus the substep headings (`### Step 4.1`, `### Step 4.2`, etc.) and block subsection headings (`#### Block 1`, `#### Block 2`, etc.).

- [ ] **Step 4: Verify the four-block templates are present**

Run:

```bash
grep -c "## Set List" skills/brain-jam/SKILL.md
grep -c "## Watch-Outs (anti-steelman per voice)" skills/brain-jam/SKILL.md
grep -c "## Undefended Assumptions" skills/brain-jam/SKILL.md
grep -c "## Argument Map" skills/brain-jam/SKILL.md
grep -c "## Critique unavailable" skills/brain-jam/SKILL.md
```

Each should return `1`.

- [ ] **Step 5: Commit**

```bash
git add skills/brain-jam/SKILL.md
git commit -m "feat(brain-jam): adapter renders critique-aware brain-jam.md

Adds Step 4 (Read transcript and render brain-jam.md) with four-block
template (Set List, Watch-Outs, Undefended Assumptions, Argument Map)
and conditional rendering for FULL / PARTIAL / NO-CRITIQUE modes.
Existing Step 4 (Handoff) renumbered to Step 5.

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

---

## Task 3: Update `skills/pen-wielding/SKILL.md` Step 1 item 3

**Files:**
- Modify: `skills/pen-wielding/SKILL.md` (line 16 in current file)

**Context:** Pen-wielding's Step 1 (Ingest Context) lists four phase outputs. Item 3 currently labels brain-jam as just "Voice". After this task it acknowledges the new sections.

- [ ] **Step 1: Read the current Step 1 item 3**

Run: `sed -n '10,20p' skills/pen-wielding/SKILL.md`

Expected: line 16 reads `3. **Voice:** \`brain-jam\` decision (The chosen angle: Deep Tech vs Pragmatic)`.

- [ ] **Step 2: Replace the line via Edit tool**

`old_string`:

```
3. **Voice:** `brain-jam` decision (The chosen angle: Deep Tech vs Pragmatic)
```

`new_string`:

```
3. **Voice + Watch-Outs:** `brain-jam` decision (chosen angle, anti-steelman watch-outs, undefended assumptions, and argument map)
```

- [ ] **Step 3: Verify**

Run: `sed -n '16p' skills/pen-wielding/SKILL.md`

Expected: the new "Voice + Watch-Outs" line.

- [ ] **Step 4: Commit**

```bash
git add skills/pen-wielding/SKILL.md
git commit -m "docs(pen-wielding): acknowledge new brain-jam.md sections in Step 1

The brain-jam adapter now produces a four-block brain-jam.md (angles,
watch-outs, assumptions, argument map). Step 1 item 3 reflects the
richer artifact.

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

---

## Task 4: Insert `skills/pen-wielding/SKILL.md` Step 3.5 (Pre-write critique check)

**Files:**
- Modify: `skills/pen-wielding/SKILL.md` (insert new section between current Step 3 end and Step 4 start)

**Context:** Step 3 ("Writing Rules") ends with the "Spelling Consistency" subsection at line ~62. Step 4 ("Section Templates") begins at line ~64 with `## Step 4: Section Templates`. We insert Step 3.5 between them. The horizontal rule (`---`) after Step 3 stays; we add a new section + new `---` before Step 4.

- [ ] **Step 1: Confirm the insertion point**

Run: `grep -n "^## Step\|^---" skills/pen-wielding/SKILL.md | head -15`

Expected: a line for `## Step 3: Writing Rules`, then a `---` separator, then `## Step 4: Section Templates`.

- [ ] **Step 2: Insert Step 3.5 via Edit tool**

`old_string`:

```
## Step 4: Section Templates
```

`new_string`:

```
## Step 3.5: Pre-write critique check

Before writing any section, scan `brain-jam.md` for the three NEW blocks:

1. **Watch-Outs (anti-steelman):** the weakest claims each voice made during the dialogue.
   Be aware as you write — these are the angles a hostile reader would attack first.
2. **Undefended Assumptions:** premises the dialogue relied on without arguing for them.
   For each one, either find supporting evidence in `deep-dive.md`, or avoid the claim entirely.
3. **Argument Map:** the final round's argument structure. Arguments in the IN set survived
   the dialogue's own scrutiny; OUT arguments did not. Prefer claims aligned with IN arguments.

If `brain-jam.md` shows "Critique unavailable", proceed normally — the dialogue produced
useful angles even without third-voice moderation.

---

## Step 4: Section Templates
```

- [ ] **Step 3: Verify the section order**

Run: `grep -n "^## Step" skills/pen-wielding/SKILL.md`

Expected output (in order):

```
## Step 1: Ingest Context
## Step 2: Load Standards
## Step 3: Writing Rules (Always Enforced)
## Step 3.5: Pre-write critique check
## Step 4: Section Templates
## Step 5: Visual Engineering
## Step 6: Anti-Patterns (Never Do)
## Step 7: Quality Metrics
## Step 8: Verification Against Previous Steps
## Step 9: The Final Audit
```

- [ ] **Step 4: Commit**

```bash
git add skills/pen-wielding/SKILL.md
git commit -m "feat(pen-wielding): add Step 3.5 pre-write critique check

Loose-coupling Aware posture: pen-wielding scans brain-jam.md's new
critique blocks before writing each section but does not enforce
rewrites. Watch-Outs, Undefended Assumptions, and Argument Map become
inputs to writer awareness, not enforcement gates.

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

---

## Task 5: Append two lines to `skills/pen-wielding/SKILL.md` Step 8 Brain-Jam Verification

**Files:**
- Modify: `skills/pen-wielding/SKILL.md` (Step 8 Brain-Jam Verification checklist)

**Context:** Step 8 has a "Brain-Jam Verification" subsection with four checklist items. We append two more.

- [ ] **Step 1: Read the current Brain-Jam Verification checklist**

Run: `grep -n -A 6 "### Brain-Jam Verification" skills/pen-wielding/SKILL.md`

Expected: the four existing checklist lines ending with "Target audience language used appropriately".

- [ ] **Step 2: Append two lines via Edit tool**

`old_string`:

```
- [ ] Tone matches the chosen angle (Deep Tech vs Pragmatic vs etc.)
- [ ] Voice is consistent throughout
- [ ] Marketing angle applied to hero section and description
- [ ] Target audience language used appropriately
```

`new_string`:

```
- [ ] Tone matches the chosen angle (Deep Tech vs Pragmatic vs etc.)
- [ ] Voice is consistent throughout
- [ ] Marketing angle applied to hero section and description
- [ ] Target audience language used appropriately
- [ ] Undefended assumptions from brain-jam.md are either substantiated or absent from the README
- [ ] No README claim aligns with an OUT argument from the argument map
```

- [ ] **Step 3: Verify**

Run: `grep -n -A 8 "### Brain-Jam Verification" skills/pen-wielding/SKILL.md`

Expected: six checklist items, the last two being the new "Undefended assumptions" and "No README claim aligns with an OUT argument" lines.

- [ ] **Step 4: Commit**

```bash
git add skills/pen-wielding/SKILL.md
git commit -m "feat(pen-wielding): add critique-aware verification lines to Step 8

Final-audit checklist now confirms undefended assumptions are
substantiated or omitted, and that no README claim aligns with an
OUT argument from the argdown map.

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

---

## Task 6: End-to-end readthrough verification

**Files:**
- Read-only: `skills/brain-jam/SKILL.md`, `skills/pen-wielding/SKILL.md`, `docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md`

**Context:** This is the manual verification step from the spec's Testing section, item 1 ("Adapter syntax check"). Items 2-5 (live runs, simulated failures) are optional follow-ups outside this plan's scope because they require a running GRFP session with staging files.

- [ ] **Step 1: Read `skills/brain-jam/SKILL.md` end-to-end**

Run the Read tool on `skills/brain-jam/SKILL.md` (the whole file).

Check:
- Steps 1-5 numbered correctly.
- Step 3 override table has three rows (output, critique, critic-temperature).
- Step 4 has substeps 4.1 through 4.4.
- Step 4 has all four block templates (Set List, Watch-Outs, Undefended Assumptions, Argument Map) and the NO-CRITIQUE replacement block.
- Step 5 (Handoff) does not re-invoke any rendering.

- [ ] **Step 2: Read `skills/pen-wielding/SKILL.md` end-to-end**

Run the Read tool on `skills/pen-wielding/SKILL.md` (the whole file).

Check:
- Step 1 item 3 reads "Voice + Watch-Outs".
- Step 3.5 is present between Step 3 and Step 4 with the three-bullet critique check.
- Step 8 Brain-Jam Verification has six checklist items.

- [ ] **Step 3: Cross-check against the spec's Acceptance criteria**

Open `docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md` to the "Acceptance criteria" section (near end of file). Mentally tick each box against what is now in the codebase:

- `skills/brain-jam/SKILL.md` documents the new three-flag override block. → Task 1
- `skills/brain-jam/SKILL.md` includes the four-block `brain-jam.md` template with the two `<!-- USER: -->` placeholders resolved. → Task 2 (placeholders were resolved inline in the spec; templates carry the resolved text)
- `skills/brain-jam/SKILL.md` documents Layer 2 mixed-status and all-unavailable rendering rules. → Task 2
- `skills/pen-wielding/SKILL.md` Step 1 item 3 is updated. → Task 3
- `skills/pen-wielding/SKILL.md` has a new Step 3.5 (Pre-write critique check). → Task 4
- `skills/pen-wielding/SKILL.md` Step 8 has two new verification lines. → Task 5
- No changes to commands, plugin manifest, or other skills. → confirm via `git diff --stat main..HEAD` — only two files changed.

- [ ] **Step 4: Confirm scope did not creep**

Run:

```bash
git diff --stat main..HEAD -- 'commands/*' '.claude-plugin/*' 'skills/deep-dive/*' 'skills/crystal-ball/*' 'skills/think-tank/*' 'skills/grfp/*'
```

Expected: empty output (no files changed in these paths).

- [ ] **Step 5: Final commit if any cleanup needed**

If Step 3 or Step 4 surfaced any drift, fix it and commit:

```bash
git add <files>
git commit -m "docs: fix drift surfaced in readthrough verification

Refs: docs/snowball/specs/2026-05-26-critic-voice-in-grfp-design.md"
```

If no drift, this step is a no-op — skip the commit.

---

## Out of plan (deferred follow-ups)

These appear in the spec's Testing section but are outside this plan:

- **Live run with critique** (Testing item 2) — requires a real GRFP session with deep-dive.md and crystal-ball.md staged. Run manually once the spec author has a target repo.
- **Failure-path simulation** (Testing item 3) — rename the m2-brainstorm binary and run `/brain-jam`. Confirms Layer 1 halt.
- **Mixed-status simulation** (Testing item 4) — hand-edit a transcript to inject `status: "unavailable"` on one critic turn, then trigger Step 4 rendering only. Confirms PARTIAL mode footnotes.
- **All-unavailable simulation** (Testing item 5) — edit all critic turns to `status: "unavailable"`, render. Confirms NO-CRITIQUE block.

These can be exercised opportunistically the next time the GRFP pipeline runs against a real codebase.