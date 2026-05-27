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

## Step 4: Handoff

`readme-brain-jam` already ends by prompting for `/claudikins-github-readme-for-perfectionists:pen-wielding`. Once delegation returns, this adapter is done — do not re-prompt or duplicate the handoff.
