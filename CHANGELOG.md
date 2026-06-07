# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

## [2.0.0] - 2026-05-26

### Changed

- **BREAKING — brain-jam engine swap**: Stage 4 (`brain-jam`) now delegates to `m2-brainstorm:readme-brain-jam` (MiniMax-M2.7-highspeed) instead of running a Gemini brainstorm via `claudikins-tool-executor`. Users must enable `m2-brainstorm@m2-deep-research` for the pipeline to complete; the skill prints a one-line installation hint if the plugin is unavailable.
- `skills/brain-jam/SKILL.md` rewritten as a thin adapter — validates GRFP staging files, computes a transcript path under `.claude/grfp/`, and delegates the conversation to the downstream skill. The orchestrator (`skills/grfp/SKILL.md`) and Phase 3 state machine are unchanged; the skill name `brain-jam` is preserved so existing pipeline wiring still works
- `commands/brain-jam.md` description updated to reflect the MiniMax engine
- Brainstorm transcripts now land in `.claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json` alongside other GRFP staging artifacts (overriding the downstream skill's default of `./.brainstorm/…`)
- `plugin.json` description and keywords updated to reflect the MiniMax engine (`gemini` keyword replaced with `minimax`)

### Deprecated

- `skills/brain-jam/references/brainstorm-gemini.md` — kept as historical reference with a deprecation banner; no longer part of the active workflow

### Migration

Enable the new brainstorm engine before running Stage 4:

```
/plugin → enable m2-brainstorm@m2-deep-research
```

The legacy Gemini path was already non-functional whenever `claudikins-tool-executor` was disabled; this release replaces the engine entirely.

Design spec: `docs/snowball/specs/2026-05-25-brain-jam-swap-design.md`
Implementation plan: `docs/snowball/plans/2026-05-25-brain-jam-swap.md`

---

## [1.2.0] - 2026-05-11

### Added

- **Graph-analysis tool integration** via `claudikins-tool-executor` plugin
- **`references/graph-analysis.md`**: Shared plugin-wide reference for the 8 codebase-memory graph tools (`search_graph`, `trace_path`, `get_architecture`, `get_code_snippet`, `query_graph`, `search_code`, `index_repository`, `index_status`), the tool-executor 3-step workflow, standard opener sequence, and gotchas
- **grfp orchestrator**: Phase 0 graph-index check (`index_status` → auto-index → graceful fallback) runs once before any codebase-analysis phase, amortising indexing cost across the pipeline
- **deep-dive skill**: Workflow rewritten to use `get_architecture` for architectural overview, `search_graph` (with `max_degree: 0`) for semantic entry-point detection, `trace_path` for "main flow" reconstruction, and `get_code_snippet` for reading source of graph-identified symbols; filename-heuristic table retained as fallback
- **crystal-ball skill**: Workflow rewritten with graph-based dead-code detection (orphaned-symbol queries), complexity hotspot identification (fan-in/fan-out via `min_degree`), deprecated-API caller enumeration, and attack-surface tracing from public entry points
- **pen-wielding skill**: Optional architecture diagram step in Visual Engineering — calls `get_architecture` and converts result to Mermaid flowchart syntax for the README
- **brain-jam and think-tank skills**: Reference link to shared graph-analysis docs for on-demand architectural queries during voice-strategy and external-research phases (no workflow changes)
- All 6 command frontmatters updated with `mcp__plugin_claudikins-tool-executor_tool-executor__*` allowed-tools entries

### Changed

- All skills now document a graceful-degradation path: if `claudikins-tool-executor` is not installed or indexing fails, skills fall back to existing Read/Glob/Grep behaviour with a single user warning
- README requirements section now describes graph-analysis capability alongside Gemini integration

---

## [1.1.1] - 2026-01-20

### Fixed

- All 5 skill descriptions CSO-optimized (now start with "Use when...")
- Standardized plugin.json metadata format

---

## [1.1.0] - 2026-01-19

### Added

- **style-guide.md**: Humanisation Patterns section with tone/voice, confidence framing, and narrative techniques
- **style-guide.md**: Reusable Phrase Templates section with intro, transition, conclusion, and warning phrases
- **style-guide.md**: "Instead of X, Write Y" table for practical rewrites
- **structural-templates.md**: Results Table, Learnings Table, and Executive Summary Block templates
- **structural-templates.md**: Risk Table and Timeline Table templates
- **structural-templates.md**: Feature-Benefit-HowTo Table template
- **structural-templates.md**: Do/Don't Patterns section with standard and table formats
- **visual-engineering.md**: ASCII Visualisations section with status indicators, progress bars, and funnels
- **anti-patterns.md**: Writing Anti-Patterns section (passive voice, hedging, corporate-speak, etc.)
- **anti-patterns.md**: Research Anti-Patterns section (assuming needs, skipping testing, etc.)
- **anti-patterns.md**: Extended Quality Checklist with 4 new items
- **think-tank/SKILL.md**: Validation Methodology section with SMART Check and Insight-to-Action linking
- **deep-dive/references/what-to-extract.md**: Research Parameters Template
- **deep-dive/references/what-to-extract.md**: Insight-to-Action Linking table

### Changed

- **pen-wielding/SKILL.md**: Updated Step 2 to reference new content in governance files
- **pen-wielding/SKILL.md**: Added archive reference note for influencer-skills-synthesis.md

### Archived

- **influencer-skills-synthesis.md**: Marked as archived with integration mapping header

## [1.0.0] - 2026-01-14

### Added

- Initial release
- Five-skill GRFP pipeline: deep-dive, crystal-ball, brain-jam, think-tank, pen-wielding
- Anti-Slop style guide with banned words
- Structural templates for CLI, library, and framework projects
- Visual engineering guidelines
- Anti-patterns documentation
- Gemini integration for brain-jam sessions
