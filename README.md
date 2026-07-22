<p align="center">
  <img src="assets/banner.png" alt="claudikins-grfp" width="100%">
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT License"></a>
  <a href="https://github.com/anthropics/claude-code"><img src="https://img.shields.io/badge/Claude_Code-Plugin-5A67D8?style=flat-square" alt="Claude Code Plugin"></a>
  <img src="https://img.shields.io/badge/version-3.1.0-blue?style=flat-square" alt="Version 3.1.0">
  <img src="https://img.shields.io/badge/Critic_Voice-Always--on-9F7AEA?style=flat-square" alt="Critic Voice: Always-on">
  <img src="https://img.shields.io/badge/Delve_Index-A0_B≤1_C≤3_D≤8-brightgreen?style=flat-square" alt="Delve Index graded budgets">
</p>

# Github Readme for Perfectionists

Blank-page READMEs stall. AI READMEs smell. This Claude Code plugin hard-gates five research stages, then grades filler into Tier A-D budgets so the final prose stays specific without sounding scrubbed sterile.

## Proof this works (or does not)

This README was rewritten by `/pen-wielding` on 2026-07-22 against the graded lexicon in `skills/pen-wielding/references/style-guide.md`. Upstream staging from the same-day GRFP run: deep-dive (file-heuristic), crystal-ball (content-level roadmap), brain-jam (`CAST=NO_KEYS` → Critique unavailable), think-tank (6 exemplars).

Staging path: `.claude/grfp/`. Without Gemini/MiniMax keys, Stage 4 still wrote a Set List and later stages continued.

Delve Index for shipping prose below: **A0 B0 C0 D0 (pass)**.

## Before and after

**Tier pile (delete):**

> ~~In today's digital age, it's important to note that this library provides a seamless way to delve into documentation, unleashing potential while harnessing a multifaceted, pivotal approach that plays a crucial role - and furthermore, ultimately, when it comes to READMEs, the synergy is actionable.~~

| Strike fragment | Tier |
| --- | --- |
| In today's digital age / it's important to note / delve / seamless / unleashing / plays a crucial role | **A** (budget 0) |
| harnessing / multifaceted / pivotal | **B** (budget 0-1) |
| _(none left once A/B cut)_ | **C** (budget ≤3) |
| furthermore / ultimately / when it comes to / synergy / actionable | **C/D** (see graded tables) |

**What ships:**

> This plugin writes READMEs. Tier A is a hard ban. Tier B allows one slip. Tier C allows a couple. Tier D caps soft adjectives and leftover ceremony at eight. Three sentence patterns force specifics. A third voice (when keys exist) attacks weak claims before you ship.

## The graded lexicon

Flat zero-tolerance lists make prose sound machine-cleaned. Pen-wielding grades filler instead.

| Tier | Budget | Fail when |
| --- | --- | --- |
| **A** | **0** | Any hit |
| **B** | **0-1** | Count > 1 |
| **C** | **≤3** | Count > 3 |
| **D** | **≤8** | Count > 8 (soft warn > 5) |

**Exempt:** marked mockery (`~~…~~` or a `delete` / `anti-example` / `ai slop` cue), proper nouns and real identifiers (`API key`, `*_KEY`, the `Deep Dive` stage title), Delve Index chrome, and the canonical lexicon tables themselves.

Famous Tier A examples: delve, tapestry, seamless, unleash, "it's important to note", "plays a crucial role", "whether you're a…", "as an AI".

Full token lists, replacements, exemptions, and ripgrep audit recipes live in [`skills/pen-wielding/references/style-guide.md`](skills/pen-wielding/references/style-guide.md). Report format: `Delve Index: A# B# C# D# (pass|fail)`.

## Quick Start (the honest list)

```bash
# 1. Add the Claudikins marketplace
/marketplace add aMilkStack/claudikins-marketplace

# 2. Install this plugin
/plugin install claudikins-grfp

# 3. Install Chorus (brainstorm engine)
ln -sf /path/to/chorus ~/.claude/skills/chorus   # skills-directory plugin symlink
export GEMINI_API_KEY=your-gemini-key-here   # synth (preferred)
export MINIMAX_API_KEY=your-minimax-key-here # pragmatist + critic (preferred)
# Either key alone works - brain-jam falls back to a single-provider cast

# 4. Install Deno (Chorus runtime)
curl -fsSL https://deno.land/install.sh | sh

# 5. Run the pipeline
/claudikins-github-readme-for-perfectionists:grfp
```

Step 3 is the easy-to-miss one. The `brain-jam` stage shells out to the Chorus CLI via Deno. Stage 4 needs the Chorus symlink, Deno, and **at least one** of `GEMINI_API_KEY` or `MINIMAX_API_KEY`. Both keys give a cross-provider cast (Gemini 3.5 Flash synth + MiniMax-M3 pragmatist/critic); a single key falls back to that provider for all voices. Deno is required - Chorus does not ship a standalone binary. Without keys, Stage 4 writes Critique unavailable and later stages still run.

Optional: install `claudikins-tool-executor` or enable `codebase-memory` MCP for graph-analysis tools (`search_graph`, `trace_path`, `get_architecture`) that improve `/deep-dive` and `/crystal-ball` on real codebases. Without either, those stages fall back to file-based heuristics.

No Exa API key or search plugin is required. `/think-tank` uses built-in `WebSearch` and `WebFetch` by default.

## The pipeline

Five sequential stages. Each writes a staging file under `.claude/grfp/`. Later stages read earlier ones. The orchestrator refuses to skip ahead.

| # | Stage | Command suffix | Output artifact | Exit criterion |
| - | ----- | -------------- | --------------- | -------------- |
| 1 | Deep Dive | `:deep-dive` | `.claude/grfp/deep-dive.md` | Reality Report with stack, friction, entry points |
| 2 | Crystal Ball | `:crystal-ball` | `.claude/grfp/crystal-ball.md` | Roadmap Candidates tables |
| 3 | Brain Jam | `:brain-jam` | `.claude/grfp/brain-jam.md` | Three angles + critic blocks (when available) |
| 4 | Think Tank | `:think-tank` | `.claude/grfp/think-tank.md` | Exemplar scores + structural blueprint |
| 5 | Pen Wielding | `:pen-wielding` | `README.md` | Graded lexicon quotas + sentence patterns pass |

You are done when stage 5 writes the file, not when you feel ready.

## How it works

```mermaid
flowchart LR
    A["/deep-dive"] --> B["/crystal-ball"]
    B --> C["/brain-jam"]
    C --> D["/think-tank"]
    D --> E["/pen-wielding"]

    A -.- A1["Facts: stack, structure, friction"]
    B -.- B1["Future: roadmap, tech debt"]
    C -.- C1["Voice: Gemini synth + MiniMax pragmatist (2 rounds)"]
    D -.- D1["Research: exemplar READMEs"]
    E -.- E1["Output: README.md + Delve Index A/B/C/D"]
```

## The critic third voice (v3.1.0)

Stage 3 runs a 3-voice dialogue: a Gemini 3.5 Flash synth and a MiniMax-M3 pragmatist debate the angle, then an argdown-grounded critic emits structured anti-steelman, undefended assumptions, and an argument map after each round. Two dialogue rounds by default. The critic's output reshapes the next round's speaker prompts. The synthesis surfaces ideas neither voice had alone.

Argdown is plain-text argument notation. `[Claim]:` states a thesis. `+` adds support. `-` raises an attack. The critic uses it the same way a linter reads an AST.

Example of what `brain-jam.md` looks like when the critic succeeds:

````markdown
## Watch-Outs (anti-steelman per voice)

**Where the "synth" voice was weakest:**
- Round 1: "Assumes cold readers care about stage counts."

## Undefended Assumptions

- (pragmatist) "Warm traffic dominates; cold readers are the minority."
- (synth) "Exit criteria are visible from the README alone."

## Argument Map (round 2)

```argdown
[Hook]: The README is the proof of the tool
  + Recursion claim with dated artifacts
  - Bloat risk if every stage dumps jargon
```

**Surviving arguments (IN):** Hook, Recursion
**Defeated arguments (OUT):** Bloat
````

When the critic fails (missing API keys, model timeout, JSON truncation, argdown parse error), the adapter renders Block 1 (Set List) only or PARTIAL blocks from surviving rounds and footnotes which rounds failed. The dialogue is never aborted for critic loss alone. This dogfood run hit `NO_KEYS` before any round started; Set List still shaped the draft.

## What pen-wielding enforces

The writing stage applies four governance files plus a pre-write critique check:

- **style-guide.md** - graded lexicon (A0 / B≤1 / C≤3 / D≤8), three sentence patterns (Hook / Hammer / Trust Builder), humanisation rules, spelling consistency
- **structural-templates.md** - section ordering by project type, results tables, do/don't patterns
- **visual-engineering.md** - mermaid diagrams, ASCII progress bars, visual density floor of 1 per 300 words
- **anti-patterns.md** - writing anti-patterns (passive voice, hedging, generic headers), research anti-patterns, quality checklist
- **Step 3.5 (v3.1.0)** - scan `brain-jam.md`'s Watch-Outs, Undefended Assumptions, and Argument Map before writing. Be aware as you write. Substantiate the assumptions from `deep-dive.md` or cut the claim.

<details>
<summary><strong>The three sentence patterns</strong></summary>

You must vary between these three. Mixing them is the whole job.

### Pattern A: The Hook (Pain plus Solution)

**Use in:** Description, Intro.

| Before | After |
| --- | --- |
| "This library helps with JSON parsing issues." | "JSON logs are unreadable. Kinesis parses them instantly." |

### Pattern B: The Hammer (Fact plus Metric)

**Use in:** Features, Performance.

| Before | After |
| --- | --- |
| "The system is designed to be incredibly fast." | "Builds finish in under 200ms." |

### Pattern C: The Trust Builder (Constraint plus Fallback)

**Use in:** Usage, Technical sections.

| Before | After |
| --- | --- |
| "Works perfectly on all platforms." | "Uses `epoll` on Linux; falls back to `kqueue` on macOS." |

</details>

<details>
<summary><strong>Humanisation patterns</strong></summary>

Write TO readers, not AT them.

| Weak | Strong | Why |
| --- | --- | --- |
| "Users can optionally configure..." | "Configure your preferences in..." | Direct address over passive |
| "The system enables..." | "This helps you..." | Human over system |
| "Consider implementing..." | "Implement..." | Commands over suggestions |
| "We achieved 2.4M reach" | "We reached 2.4M people, 20% above target" | Context makes numbers meaningful |

</details>

## State of the project (v3.1.0)

| Status | Item |
| --- | --- |
| Shipped | 5-stage pipeline with hard gates between stages |
| Shipped | Chorus brain-jam engine (replaced m2-brainstorm in v3.0.0) |
| Shipped | Gemini 3.5 Flash synth + MiniMax-M3 pragmatist/critic (v3.1.0) |
| Shipped | Single-provider fallbacks when only one API key is set |
| Shipped | 2-round dialogue synthesis (`--max-rounds 2`) |
| Shipped | Graded Anti-Slop lexicon (Tier A-D budgets) |
| Draft | Cast recipe at `skills/brain-jam/recipes/grfp-readme.json` |
| Missing | CI workflows that enforce Delve Index quotas |
| Missing | Offline / no-key brain-jam path beyond Critique unavailable |

## Quality targets

| Metric | Target |
| --- | --- |
| Flesch-Kincaid | Grade 8-10 |
| Time to Joy | 5 commands (honest count, see Quick Start) |
| Visual density | 1 per 300 words |
| Badge count | 5-7 |
| Delve Index | A0; B≤1; C≤3; D≤8 |

## When NOT to use this

- **You want full creative control.** The pipeline enforces structure. It will fight you.
- **Your project is trivial.** A 20-line script does not need a 5-phase pipeline.
- **You need speed.** Each phase takes minutes. The brain-jam stage alone is 3-9 minutes wallclock with critique on.
- **You hate opinionated tools.** The opinions are baked in.
- **You want a general-purpose AI conversation.** Ask Claude or GPT directly. The plugin's value is exit criteria, not chat fluency.

## Requirements

- Claude Code 1.0 or later
- Chorus plugin symlinked at `~/.claude/skills/chorus` (required for `/brain-jam`)
- Deno (required - Chorus launcher shells out to `deno run`)
- At least one of `GEMINI_API_KEY` or `MINIMAX_API_KEY` in environment or `~/.claude/skills/chorus/.env` (both preferred)
- `claudikins-tool-executor` plugin or `codebase-memory` MCP (optional; enables graph-analysis tools for `/deep-dive` and `/crystal-ball`)

## License

[MIT](LICENSE)

---

_Delve Index: A0 B0 C0 D0 (pass). Pen-wielding rerun: 2026-07-22 (graded lexicon surfaced). Prior stages: deep-dive (file-heuristic), crystal-ball (content-level), brain-jam (NO_KEYS), think-tank (WebSearch)._
