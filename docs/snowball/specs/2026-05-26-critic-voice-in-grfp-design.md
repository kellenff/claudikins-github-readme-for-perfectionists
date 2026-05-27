# Critic Voice in GRFP Brain-Jam: Always-On Wiring

**Date:** 2026-05-26
**Status:** Design approved, awaiting implementation plan
**Plugin:** `claudikins-grfp` (claudikins-github-readme-for-perfectionists)
**Predecessor:** `2026-05-25-brain-jam-swap-design.md`
**Upstream feature spec:** `m2-deep-research/docs/snowball/specs/2026-05-26-m2-brainstorm-critic-voice-design.md`

## Summary

Wire the `m2-brainstorm` engine's new critic third voice into the GRFP pipeline by:

1. Making `--critique` and `--critic-temperature 0.3` mandatory overrides in the local `brain-jam` adapter's delegation to `m2-brainstorm:readme-brain-jam`.
2. Moving synthesis-and-write responsibility from upstream into the adapter, since upstream's 3-angle Set List format cannot render the critic's structured data.
3. Restructuring `.claude/grfp/brain-jam.md` from a single Set List into four blocks: angles, watch-outs (anti-steelman), undefended assumptions, and an argument map (argdown + Dung extension).
4. Adding a "Pre-write critique check" step to `pen-wielding` that makes the writer *aware* of the critique signal without enforcing rewrites.

Always-on for GRFP. Users who want the cheaper two-voice dialogue can invoke `/m2-brainstorm:readme-brain-jam` directly without the GRFP wrapper.

## Context

- **Current state:** `skills/brain-jam/SKILL.md` is a thin adapter that overrides only `--output` when delegating to `m2-brainstorm:readme-brain-jam`. Upstream runs a 2-voice dialogue (claude-synth + pragmatist) and writes `brain-jam.md` itself using a 3-angle Set List format.
- **New upstream capability:** `m2-brainstorm` v0.2.0+ ships `--critique` and `--critic-temperature` flags. The critic runs after each round, emitting a structured JSON object with `factual_assertions`, `assumptions`, `steelman` / `anti_steelman` pairs, an argdown source, and a Dung extension partition. The critic's output is used to augment the next round's speaker prompts (per-speaker addendums) and surfaces in the transcript as `speaker: "critic"` turns.
- **Shipped engine version:** v0.3.0 (TypeScript port) is installed in `~/.claude/plugins/cache/m2-deep-research/m2-brainstorm/0.3.0`. Source confirms both flags are implemented in `src/brainstorm/cli.ts` and `src/brainstorm/critic.ts`.
- **Drift to ignore:** the local adapter's current text references `uv run python brainstorm.py …`, while the shipped binary is `$HOME/.config/m2-brainstorm/bin/m2-brainstorm`. The adapter's revised text references *flags to override*, not the exact shell invocation, so the drift becomes irrelevant rather than something we have to chase.

## Goals

1. Every GRFP brain-jam run produces a critic-augmented dialogue with zero user configuration.
2. `brain-jam.md` exposes the critic's anti-steelman and undefended-assumption signal to pen-wielding.
3. Pen-wielding becomes *aware* of the critique sections without rigid enforcement rules.
4. Failure modes degrade visibly: partial critique coverage is footnoted; total critique unavailability is announced; CLI failures halt the adapter with the upstream error verbatim.
5. The adapter stays thin: it owns flag overrides and GRFP-specific markdown rendering, nothing else.

## Non-Goals

- Modifying upstream `m2-brainstorm:readme-brain-jam`. The adapter intercepts before upstream's synthesis step; we do not push GRFP knowledge upstream.
- Adding adapter-level args to toggle critique on/off. The pipeline is opinionated.
- Surfacing the critic's `factual_assertions` in `brain-jam.md`. Pen-wielding's job is positioning, not auditing — assertions stay in the transcript JSON for anyone who wants them.
- Building a separate "brain-jam-with-critic" command or skill. The GRFP brain-jam *is* the critic-enabled one now.
- Persisting argdown sources as standalone files. The argdown lives inside `brain-jam.md` and the transcript JSON only.

## Repository layout

```
claudikins-github-readme-for-perfectionists/
├── skills/
│   ├── brain-jam/
│   │   └── SKILL.md                # MODIFIED: override block grows; synthesis-write moves here
│   └── pen-wielding/
│       └── SKILL.md                # MODIFIED: Step 1 + new Step 3.5 + new Step 8 line
└── docs/snowball/specs/
    └── 2026-05-26-critic-voice-in-grfp-design.md   # THIS FILE
```

No new files. No new reference docs. No changes to commands or plugin manifest.

## Architecture: responsibility split

```
┌─────────────────────────────────────────────────────────────────┐
│ claudikins-grfp:brain-jam (adapter, this repo)                  │
│                                                                 │
│  1. Verify GRFP staging files                                   │
│  2. Verify m2-brainstorm plugin available                       │
│  3. Compute timestamp                                            │
│  4. Delegate to readme-brain-jam with overrides:                │
│      --output     → .claude/grfp/brainstorm-transcript-<ts>.json│
│      --critique   → always-on                                   │
│      --critic-temperature 0.3                                   │
│      Halt downstream before its Step 6 (synthesis)              │
│  5. Read transcript, render restructured brain-jam.md           │
│  6. Prompt user for /pen-wielding                               │
└─────────────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────┐
│ m2-brainstorm:readme-brain-jam (upstream, unchanged)            │
│                                                                 │
│  - Three-question Sound Check                                   │
│  - Read deep-dive.md + crystal-ball.md → --claude-thoughts      │
│  - Run m2-brainstorm CLI with given flags                       │
│  - Returns transcript path; halts before its Step 6/7           │
└─────────────────────────────────────────────────────────────────┘
```

**Single responsibility check:**
- Upstream owns: dialogue mechanics, MiniMax invocation, sound-check questions, transcript format.
- Adapter owns: GRFP-specific markdown rendering, `.claude/grfp/` paths, pen-wielding handoff.

The adapter's Step 3 (Delegation contract) gains one new instruction line:

> *"When reading the downstream `readme-brain-jam` skill, treat its Step 6 (Set List synthesis) and Step 7 (write `brain-jam.md`, prompt for pen-wielding) as overridden. Stop following downstream instructions after the CLI returns; the adapter's Step 5 performs the equivalent work."*

The adapter is metacognitive instruction for the agent running the GRFP pipeline — it modifies which parts of the downstream skill the agent executes, the same mechanism by which the current adapter overrides `--output`.

## CLI invocation contract

The adapter's Step 3 (Delegation contract) currently locks one override (`--output`). After this change, it locks three flags and explicitly disowns synthesis-write.

**New override block (verbatim, locked):**

```
Required overrides when invoking m2-brainstorm:readme-brain-jam:

  --output              .claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
  --critique            (no value; flag-only)
  --critic-temperature  0.3

Required behavior override:
  After the upstream CLI invocation succeeds, halt the downstream skill.
  Do NOT execute upstream's Step 6 (Set List synthesis) or Step 7 (write brain-jam.md).
  The adapter performs synthesis and writes brain-jam.md itself.
```

**Why `--critic-temperature 0.3`:** matches the m2-brainstorm critic spec default. The critic's job is analytical extraction (assertions, assumptions, steelman pairs), which favors deterministic output over creative variation. README work needs the critic to be a *consistent reader* across rounds, not a *creative reader*.

**Why we lock these and don't expose them as adapter args:** GRFP is an opinionated pipeline. Adapter args would re-introduce indecision. Users who genuinely need a one-off tweak can invoke `/m2-brainstorm:readme-brain-jam` directly with their own flags.

**Reference form for the override:** the adapter references *what flags to override*, not *how upstream invokes the CLI*. This keeps the adapter insulated from upstream runtime changes (Python → TypeScript → whatever comes next).

## `brain-jam.md` restructured format

The new file has four blocks. Per-round critic data is aggregated across rounds (not rendered round-by-round) because pen-wielding cares about cumulative signal.

### Block 1 — Set List (unchanged from current)

```markdown
## Set List

**Option 1: The "Deep Tech" Angle**
_Headline Idea:_ ...
_Focus:_ ...
_Cited from:_ claude_r2, pragmatist_r3

**Option 2: The "Pragmatic Solver" Angle**
_Headline Idea:_ ...
_Focus:_ ...
_Cited from:_ pragmatist_r1, claude_r3

**Option 3: The Synthesis (Recommended)**
_Headline Idea:_ ...
_Tone:_ ...
_Cited from:_ critic_r2 (steelman synthesis)
```

Citation lines may reference `critic_rN` turns where the critic's steelman influenced the synthesis.

### Block 2 — Watch-Outs (anti-steelman per voice) (NEW)

Aggregate of `anti_steelman` fields from all `status: "ok"` critic turns. Each speaker's weakest claim per round, grouped by speaker.

```markdown
## Watch-Outs (anti-steelman per voice)

**Where the "deep tech" voice was weakest:**
- Round 1: "<anti_steelman.claude verbatim>"
- Round 2: "<anti_steelman.claude verbatim>"

**Where the "pragmatist" voice was weakest:**
- Round 1: "<anti_steelman.pragmatist verbatim>"
- Round 2: "<anti_steelman.pragmatist verbatim>"
```

### Block 3 — Undefended Assumptions (NEW)

Aggregate of `assumptions[].premise` where `argued_for == false`, across all `status: "ok"` critic turns. Dedupe by simple case-insensitive string match.

```markdown
## Undefended Assumptions

<!-- USER: write Block 3 header text here (2-3 lines).

The header sits between the "## Undefended Assumptions" heading and the bullet list.
It frames how pen-wielding should treat these premises. Tone-shaping choice:
clinical ("must verify") vs cautionary ("watch out for") vs neutral ("notice these").

Your phrasing here ships verbatim. -->

- (claude) "<premise>"
- (pragmatist) "<premise>"
```

### Block 4 — Argument Map (NEW)

Final round's argdown source (fenced) plus its Dung extension partition. Final round only — earlier rounds' argument labels (`[A]`, `[B]`) can collide with later rounds' labels per the upstream spec.

```markdown
## Argument Map (final round)

\`\`\`argdown
[A]: claim text
  +> [B]: supporting argument
  -> [C]: counter-argument
\`\`\`

**Surviving arguments (IN):** A, B
**Defeated arguments (OUT):** C
**Undecided (UNDEC):** _(none)_
```

### What we explicitly omit

- **`factual_assertions`:** pen-wielding does positioning, not fact-checking. Including them invites over-indexing on individual claims and losing the angle. Available in the transcript JSON for anyone who wants them.
- **Per-round argdown sources:** label collisions across rounds make concatenation meaningless.
- **Round-level synthesis hint:** upstream emits `synthesis_hint` in the transcript; the Set List already encodes this, so we don't duplicate.

## Failure handling

Three layers of failure. Two are already handled by upstream and need no new adapter logic; one is new and the adapter must own it.

### Layer 1 — CLI exit failure (unchanged)

Network outage, missing API key, binary not installed, invalid args. Upstream surfaces the error verbatim; adapter halts. The adapter's existing contract already says "let the downstream skill surface the error. Do not wrap, retry, or swallow it."

### Layer 2 — Sentinel critic turns inside the transcript (NEW)

When `--critique` is on, the engine emits per-round critic turns. A failed critic round produces a sentinel:

```json
{
  "round": 2,
  "speaker": "critic",
  "status": "unavailable",
  "error": "argdown.parse failed at line 3: unexpected token",
  "raw_text": "<final attempt before giving up>"
}
```

The engine never aborts the dialogue on a sentinel — it just continues without addendum augmentation for the next round. So the transcript may contain a mix of `status: "ok"` and `status: "unavailable"` critic turns.

**Adapter rendering rules for mixed status:**

- **All critic turns `ok`:** render Blocks 2, 3, 4 normally.
- **Some `ok`, some `unavailable`:** render Blocks 2 & 3 using only the `ok` turns. Add a footnote at the end of each block:
  > *Critic was unavailable for round(s) N. Coverage is partial.*

  For Block 4 (Argument Map), use the most recent `ok` critic turn's argdown source — not necessarily the final round. Label it: `## Argument Map (round N — final critic-ok round)`.
- **All critic turns `unavailable`:** render Block 1 (Set List) only. Replace Blocks 2-4 with a single block:

  ```markdown
  ## Critique unavailable

  <!-- USER: write the 'all critic unavailable' message here (~3 lines).

  This message must convey two things at once:
    (1) this is NOT a pipeline failure — the dialogue ran fine;
    (2) pen-wielding should know it's writing without the third-voice safety net.

  Include the per-round errors at the end of your message:
    "Errors were: round 1: <error>; round 2: <error>; round 3: <error>."

  Your phrasing here ships verbatim. -->
  ```

  This is a *successful* run from GRFP's perspective — the dialogue completed, just without the third voice. Do not halt.

### Layer 3 — Transcript shape unexpected (defensive)

The adapter parses the transcript JSON. If required fields are missing (no `turns` array, no `synthesis_hint`, no `critique_aggregate` when expected, etc.), the adapter halts with:

> Transcript shape invalid: <field> missing. The m2-brainstorm engine may have changed contract. Run `/m2-brainstorm:readme-brain-jam` directly to verify, then report upstream.

This guards against future engine drift. The adapter's only job at this boundary is to turn loose JSON into a typed handoff for the markdown renderer — parse-don't-validate.

### No silent degradation

A user looking at `brain-jam.md` always knows whether they got full critique signal or partial. Footnotes do that visibly. We never produce a "best effort" file that *looks* complete but isn't.

## Pen-wielding integration

Three precise touchpoints in `skills/pen-wielding/SKILL.md`. Loose coupling (Aware posture): pen-wielding becomes *aware* of the critique sections without rigid rewrite rules.

### Touchpoint 1 — Step 1 (Ingest Context), item 3

**Current:**
> *3. **Voice:** `brain-jam` decision (The chosen angle: Deep Tech vs Pragmatic)*

**New:**
> *3. **Voice + Watch-Outs:** `brain-jam` decision (chosen angle, anti-steelman watch-outs, undefended assumptions, and argument map)*

That's the only Step-1 change. Pen-wielding still treats `brain-jam.md` as a single ingest artifact, just a richer one.

### Touchpoint 2 — NEW Step 3.5 (Pre-write critique check)

Inserted between current Step 3 (Writing Rules) and Step 4 (Section Templates):

```markdown
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
```

### Touchpoint 3 — New verification lines in Step 8 (Brain-Jam Verification)

Append two lines to the existing checklist under "Brain-Jam Verification":

```markdown
- [ ] Undefended assumptions from brain-jam.md are either substantiated or absent from the README
- [ ] No README claim aligns with an OUT argument from the argument map
```

### What we explicitly don't add

- No defensive rewriting loop. (User picked Aware, not Defensive or Adversarial.)
- No quoting of critic text verbatim in the README. The critique is *upstream signal*, not README content.
- No new reference file. The Step 3.5 prose lives inline because it's short and tied to one specific artifact.

### Skill description

Unchanged. Pen-wielding still synthesises four phase outputs — brain-jam just got richer.

## Testing

This is a documentation-only change (skill markdown). No code, no automated tests. Verification is manual:

1. **Adapter syntax check:** read the modified `skills/brain-jam/SKILL.md` end-to-end. The override block, halt instruction, and rendering steps must be unambiguous.
2. **Live run with critique:** in a real GRFP session, complete deep-dive and crystal-ball, then run `/brain-jam`. Verify:
   - The upstream CLI is invoked with `--critique` and `--critic-temperature 0.3`.
   - The transcript JSON contains critic turns.
   - `brain-jam.md` has all four blocks rendered correctly.
3. **Failure-path simulation:** run a brain-jam where the m2-brainstorm CLI is unavailable (rename the binary). Adapter must halt with upstream's error.
4. **Mixed-status simulation:** edit a real transcript to inject a `status: "unavailable"` critic turn, re-run the adapter's Step 5 (render). Verify footnote appears and Argument Map references the correct round.
5. **All-unavailable simulation:** edit all critic turns to `status: "unavailable"`. Verify the "Critique unavailable" block replaces Blocks 2-4.

The implementation plan should sequence these so manual verification follows logically from the diff.

## Out of scope (YAGNI)

- **Adapter-level critique toggle.** Pipeline is always-on.
- **Cross-model critic.** Adapter inherits upstream's default (same MiniMax generator for critic and speakers).
- **Critic-confabulation detection.** Per upstream's critic spec, the critic can invent assertions or assumptions not in the speaker turns. Pen-wielding's "Aware" posture inherits this risk; mitigations are future work.
- **Argdown rendering as a graphical diagram.** Plain fenced text only. GitHub doesn't render argdown natively; an image pipeline is a separate spec.
- **Updating `commands/brain-jam.md`.** The command file calls the skill by name; the new behavior is invisible at the command surface.
- **Changes to deep-dive, crystal-ball, think-tank, or grfp skills.** They produce no inputs to the critic and consume no outputs from it.

## Open questions (resolved during brainstorming)

| Q | Resolution |
|---|---|
| Always-on, opt-out, or opt-in critique? | **Always-on.** |
| Surface critique data to pen-wielding? | **Full restructure** of `brain-jam.md`. |
| Update pen-wielding too, or just brain-jam? | **Both, in this spec.** |
| How aggressively should pen-wielding consume the critique? | **Aware** (loose coupling). |
| Adapter approach — pure flag injection, fork, or push upstream? | **Pure flag injection (Approach A).** |
| Reference the binary path or stay invocation-agnostic? | **Invocation-agnostic** — adapter references flags only. |

## Risks

1. **Upstream contract drift.** If m2-brainstorm changes the transcript JSON shape (renames `critic` to `judge`, restructures `dung_extension`, etc.), the adapter's Layer 3 halt fires. This is intentional: visible breakage beats silent garbage. Mitigation: the spec author maintains upstream and downstream — drift will be caught early.
2. **Critic latency.** Always-on means every brain-jam run incurs the extra ~50% wallclock from N additional critic API calls. Users on slow networks may notice. Acceptable cost for the always-on opinionated pipeline.
3. **Markdown-renderer complexity in the adapter.** The adapter is now responsible for parsing JSON and rendering four conditional blocks. This is more logic than a "thin adapter" usually carries. Mitigation: keep the rendering rules in the skill markdown declarative (templates), not procedural.
4. **Pen-wielding ignoring the new blocks.** The Aware posture relies on writer discipline. A future tightening to Defensive remains available without breaking this spec.

## Acceptance criteria

- [ ] `skills/brain-jam/SKILL.md` documents the new three-flag override block.
- [ ] `skills/brain-jam/SKILL.md` includes the four-block `brain-jam.md` template with the two `<!-- USER: -->` placeholders resolved.
- [ ] `skills/brain-jam/SKILL.md` documents Layer 2 mixed-status and all-unavailable rendering rules.
- [ ] `skills/pen-wielding/SKILL.md` Step 1 item 3 is updated.
- [ ] `skills/pen-wielding/SKILL.md` has a new Step 3.5 (Pre-write critique check).
- [ ] `skills/pen-wielding/SKILL.md` Step 8 has two new verification lines.
- [ ] No changes to commands, plugin manifest, or other skills.
- [ ] Spec committed; implementation plan generated as a follow-up.