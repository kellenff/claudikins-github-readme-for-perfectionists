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
