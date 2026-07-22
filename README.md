<p align="center">
  <img src="assets/banner.png" alt="claudikins-grfp" width="100%">
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT License"></a>
  <a href="https://github.com/anthropics/claude-code"><img src="https://img.shields.io/badge/Claude_Code-Plugin-5A67D8?style=flat-square" alt="Claude Code Plugin"></a>
  <img src="https://img.shields.io/badge/version-3.1.0-blue?style=flat-square" alt="Version 3.1.0">
  <img src="https://img.shields.io/badge/Critic_Voice-Always--on-9F7AEA?style=flat-square" alt="Critic Voice: Always-on">
  <img src="https://img.shields.io/badge/Delve_Index-0%25-brightgreen?style=flat-square" alt="0% Delve Index">
</p>

# Github Readme for Perfectionists

ESLint for prose. A Claude Code plugin that treats "delve" as a syntax error and will not let you skip to the final draft until five staged artifacts exist.

## Proof this works (or does not)

This README was written by running the plugin's own pipeline on this repository on 2026-06-07. The five stages produced: deep-dive (Reality Report, 132 lines), crystal-ball (9 roadmap candidates), brain-jam (3 angles via Gemini synth + MiniMax pragmatist, 2 rounds), think-tank (6 exemplars scored against a rubric), pen-wielding (this file).

The critic returned round 1 output and failed on round 2 with JSON truncation. The PARTIAL fallback rendered Watch-Outs and Undefended Assumptions from the surviving round and footnoted the gap. The dialogue was not aborted. Transcript: `.claude/grfp/brainstorm-transcript-20260607T171259.json`.

Banned-word count in the source above: 0.

## Before and after

The kind of sentence this plugin exists to delete:

> ~~This library provides a seamless way to delve into documentation generation, unleashing the full potential of your README workflow.~~

The kind of sentence this plugin ships:

> This plugin generates README files. It bans 31 filler words, enforces three sentence patterns that require specifics, and runs a third voice that flags weak claims the writer cannot see.

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

Step 3 is the easy-to-miss one. The `brain-jam` stage shells out to the Chorus CLI via Deno. Stage 4 needs the Chorus symlink, Deno, and **at least one** of `GEMINI_API_KEY` or `MINIMAX_API_KEY`. Both keys give a cross-provider cast (Gemini 3.5 Flash synth + MiniMax-M3 pragmatist/critic); a single key falls back to that provider for all voices. Deno is required - Chorus does not ship a standalone binary.

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
| 5 | Pen Wielding | `:pen-wielding` | `README.md` | Final prose passes Anti-Slop governance |

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
    E -.- E1["Output: README.md"]
```

## The critic third voice (v3.1.0)

Stage 3 runs a 3-voice dialogue: a Gemini 3.5 Flash synth and a MiniMax-M3 pragmatist debate the angle, then an argdown-grounded critic emits structured anti-steelman, undefended assumptions, and an argument map after each round. Two dialogue rounds by default. The critic's output reshapes the next round's speaker prompts. The synthesis surfaces ideas neither voice had alone.

Argdown is plain-text argument notation. `[Claim]:` states a thesis. `+` adds support. `-` raises an attack. The critic uses it the same way a linter reads an AST.

Example of what `brain-jam.md` looks like when the critic succeeds:

````markdown
## Watch-Outs (anti-steelman per voice)

**Where the "deep tech" voice was weakest:**
- Round 2: "The argdown grounding only matters if the reader already
  knows what a Dung extension is; otherwise it reads as jargon proof."

## Undefended Assumptions

- (pragmatist) "Warm traffic dominates; cold readers are the minority."
- (synth) "Exit criteria are visible from the README alone."

## Argument Map (round 2)

```argdown
[Hook]: The README is the proof of the tool
  +> [Recursion]: It was written by the pipeline
  -> [Bloat]: Five stages signals over-engineering
```

**Surviving arguments (IN):** Hook, Recursion
**Defeated arguments (OUT):** Bloat
````

When the critic fails (model timeout, JSON truncation, argdown parse error), the adapter renders Block 1 (Set List) only or PARTIAL blocks from surviving rounds and footnotes which rounds failed. The dialogue is never aborted. Round 2 of this README's brain-jam stage hit JSON truncation; round 1 critique still shaped the draft.

## What pen-wielding enforces

The writing stage applies four governance files plus a pre-write critique check:

- **style-guide.md** - 31 banned words, three sentence patterns (Hook / Hammer / Trust Builder), humanisation rules, spelling consistency
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

## The banned words

| Word | Replacement |
| --- | --- |
| Delve | Analyse, Check, Query |
| Seamless | Compatible, Integrated |
| Unleash | Run, Execute, Enable |
| Robust | Fault-tolerant, Atomic |
| Tapestry | System, Network, Stack |
| Landscape | Delete the sentence |
| Elevate | Improve (with a metric) |
| Testament | Proof, Example |
| Foster | Encourage, Allow |
| Spearhead | Lead, Direct |
| Game-changer | Solves [specific problem] |
| Navigating | Using, Working with |
| Leverage | Use, Apply, Employ |
| Cutting-edge | Modern, Current |
| Empower | Allow, Enable, Let |
| Spine | Core, Backbone, Main path, Structure |
| Substrate | Base layer, Foundation, Underlying layer |
| Load-bearing | Critical, Essential, Required, Core |
| Reality check | Verify, Confirm, Check against [facts] |
| Underscore | Show, Prove, Make clear |
| Pivotal | Key, Central, Required |
| Multifaceted | Complex, Varied, Has N parts |
| Comprehensive | Full, Complete, Covers X/Y/Z |
| Harness | Use, Apply, Run |
| Showcase | Show, Demonstrate, List |
| Realm | Area, Domain, Topic |
| Utilize | Use |
| Unlock | Enable, Open, Expose |
| Holistic | End-to-end, Whole-system, Across X |
| It's important to note | Delete; write the claim |
| Plays a crucial role | Does X / Causes Y / Controls Z |

If any of these appear in the output, the README fails its own rules.

## State of the project (v3.1.0)

| Status | Item |
| --- | --- |
| Shipped | 5-stage pipeline with hard gates between stages |
| Shipped | Chorus brain-jam engine (replaced m2-brainstorm in v3.0.0) |
| Shipped | Gemini 3.5 Flash synth + MiniMax-M3 pragmatist/critic (v3.1.0) |
| Shipped | Single-provider fallbacks when only one API key is set |
| Shipped | 2-round dialogue synthesis (`--max-rounds 2`) |
| Draft | Cast recipe at `skills/brain-jam/recipes/grfp-readme.json` |
| Missing | CI workflows (no `.github/workflows/`) |

## Quality targets

| Metric | Target |
| --- | --- |
| Flesch-Kincaid | Grade 8-10 |
| Time to Joy | 5 commands (honest count, see Quick Start) |
| Visual density | 1 per 300 words |
| Badge count | 5 max |

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

_Delve Index of this README: 0%. Pipeline run: 2026-06-07. Stages: deep-dive (3m), crystal-ball (2m), brain-jam (9m, critic partial - round 2 JSON truncation), think-tank (2m), pen-wielding (4m)._
