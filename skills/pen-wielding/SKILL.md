---
name: pen-wielding
description: "Use when writing the final README after all phases complete. Synthesises outputs from deep-dive, crystal-ball, brain-jam, and think-tank. Applies Anti-Slop style guide. Uses get_architecture graph tool to auto-generate Mermaid architecture diagrams."
---

# Pen-Wielding Protocol

**Goal:** Write a production-ready GitHub README that feels authentically human.

## Step 1: Ingest Context

Read the outputs from the preparatory skills:

1. **Facts:** `deep-dive` report (Stack, friction points, install complexity)
2. **Future:** `crystal-ball` analysis (Roadmap table, technical debt)
3. **Voice + Watch-Outs:** `brain-jam` decision (chosen angle, anti-steelman watch-outs, undefended assumptions, and argument map)
4. **Plan:** `think-tank` findings (Selected badges, sections, and visual strategy)

**Do not proceed until all four reports are available.**

---

## Step 2: Load Standards

Read the governance files to establish the form:

1. `references/style-guide.md` (Anti-Slop rules, Graded Lexicon A–D, Humanisation Patterns, Reusable Phrases)
2. `references/structural-templates.md` (Section templates, Results/Metrics tables, Do/Don't patterns)
3. `references/visual-engineering.md` (Density rules, diagrams, ASCII visualisations)
4. `references/anti-patterns.md` (Final checklist, Writing/Research anti-patterns)

**Note:** The reference files include patterns integrated from the influencer-skills-synthesis research (archived at `../grfp/reference/influencer-skills-synthesis.md`).

---

## Step 3: Writing Rules (Always Enforced)

### Style

- Clear and concise, active voice
- Important information first
- **No M dash (—) EVER** - use hyphens or restructure
- Technical only - no marketing fluff
- No paragraph > 4 sentences

### The "Anti-Slop" Filter

- **Check every sentence** against the graded lexicon in `references/style-guide.md`
  - **Tier A budget 0** (hard fail on delve, seamless, unleash, …)
  - **Tier B budget 0–1**, **Tier C ≤3**, **Tier D ≤5** (total hits per tier)
- **Mockery exemption:** lexicon hits inside a *marked* anti-example do not consume budgets
  - Markers: strikethrough `~~…~~`, or a cue line (`delete` / `anti-example` / `ai slop` / `tier pile` / `exists to delete` / `do not ship` / `bad:`), plus annotation tables that only catalogue that quote
  - Unlabeled blockquotes and scare-quote irony still count
  - Prose around the mock quote still counts
- **Proper-noun exemption:** real product/company/person/place/work titles and code identifiers that contain a lexicon token do not consume budgets
  - Prefer canonical casing, links, backticks, or a type noun on first mention
  - Generic reuse next to the name still counts (`Foster Inc.` exempt; `foster collaboration` counts)
  - Fake brands invented to smuggle filler are not exempt
- Report `Delve Index: A# B# C# D# (pass|fail)` in the README proof footer (quotas exclude exempt mockery and proper nouns)
- **Never use "The Problem / The Solution" framing** - it's lazy and cliched
- **Enforce Patterns:**
  - Use **"The Hook"** (Pattern A) in the _Description_
  - Use **"The Hammer"** (Pattern B) in the _Features_ list
  - Use **"The Trust Builder"** (Pattern C) in _Usage/Configuration_

### Spelling Consistency

Pick one and stick to it:

- **British:** colour, optimise, analyse, behaviour, centre, finalising
- **American:** color, optimize, analyze, behavior, center, finalizing

---

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

Write sections in this order:

### 1. Hero Section

```markdown
<p align="center"><img src="banner.png" alt="Project Name" width="600"></p>
<p align="center">
  <a href="..."><img src="badge1" alt="Build"></a>
  <!-- 5-7 badges max: build, coverage, version, license, downloads -->
</p>

# Project Name - Core Function Description

> One sentence value proposition (what it does, why you'd use it)
```

### 2. Description

```markdown
## What is [Project]?

[Project] does X for Y. Instead of [tedious manual approach], you [simple action] and get [result].

**Key Features:**

- [Feature 1]: [Benefit]
- [Feature 2]: [Benefit]
```

### 3. Installation

```markdown
## Installation

### Prerequisites

- Node.js 18+ (LTS recommended)

### Package Manager

\`\`\`bash
npm install project-name
\`\`\`

### Docker (if Time-to-Joy > 3 commands)

\`\`\`bash
docker run -it project-name
\`\`\`
```

### 4. Quick Start

```markdown
## Quick Start

\`\`\`typescript
import { thing } from 'project-name';
const result = thing.doSomething();
\`\`\`

**Output:**
\`\`\`
Expected output here
\`\`\`
```

### 5. Usage (with collapsible advanced)

```markdown
## Usage

### Basic Usage

[Code example with explanation]

<details>
<summary>Advanced Usage</summary>

[Advanced configuration, options, edge cases]

</details>
```

### 6. Configuration (table format)

```markdown
| Option    | Type   | Default     | Description  |
| --------- | ------ | ----------- | ------------ |
| `option1` | string | `"default"` | What it does |
```

---

## Step 5: Visual Engineering

### When to Use What

| Visual Type  | Use Case                | Tool          |
| ------------ | ----------------------- | ------------- |
| Terminal GIF | CLI tools, workflows    | VHS           |
| Diagram      | Architecture, data flow | Mermaid       |
| Screenshot   | UI apps, dashboards     | With alt text |

### Visual Density

**Target:** 1 visual per 300 words. If exceeded, add code block, diagram, or break into subsections.

### Mermaid Diagrams (GitHub-native)

```markdown
\`\`\`mermaid
flowchart LR
A[Input] --> B[Process] --> C[Output]
\`\`\`
```

### Graph-Derived Architecture Diagrams (Optional)

When `claudikins-tool-executor` is available and the codebase was indexed in Phase 0, use `get_architecture` to auto-generate a Mermaid module map instead of writing it by hand:

```typescript
await execute_code(`
  const arch = await serena.get_architecture({
    aspects: ["packages", "services", "entry_points"]
  });

  // Convert to Mermaid flowchart
  const nodes = arch.packages?.map(p => \`  \${p.id}[\${p.name}]\`).join("\\n") ?? "";
  const edges = arch.dependencies?.map(d => \`  \${d.from} --> \${d.to}\`).join("\\n") ?? "";
  const diagram = \`flowchart LR\\n\${nodes}\\n\${edges}\`;

  await workspace.writeJSON("pen-wielding/arch-diagram.json", { arch, diagram });
`);
```

Then embed the generated `diagram` string in the README's Mermaid block. If graph tools are unavailable, hand-write the diagram from the `deep-dive` Reality Report.

**See `references/graph-analysis.md` at the plugin root for the tool-executor workflow and `get_architecture` parameter details.**

### Visual Placeholders

Insert specific placeholders for visuals identified in `think-tank`:

```markdown
> **Visual:** [Description from think-tank] (Use tool: VHS/Mermaid)
```

### Accessibility Requirements

- **Alt text:** Always descriptive, never empty
- Bad: `![](image.png)`
- Good: `![Architecture diagram showing client-server flow](arch.png)`

---

## Step 6: Anti-Patterns (Never Do)

| Anti-Pattern                 | Problem            | Fix                                                 |
| ---------------------------- | ------------------ | --------------------------------------------------- |
| Wall of text                 | Unreadable         | Max 4 sentences per paragraph                       |
| Badge overload               | Cluttered          | Max 5-7 essential badges                            |
| Generic headers              | Poor SEO           | Semantic: "High-Performance Parsing" not "Features" |
| Missing prerequisites        | Broken installs    | Explicit versions: "Node.js 18+"                    |
| No output shown              | Confusing          | Always show expected result                         |
| M dash (—)                   | Rendering issues   | Use hyphens or restructure                          |
| Buried quick start           | User abandonment   | Within first screen view                            |
| Copy-paste friction          | User errors        | Working defaults, note what to change               |
| Click here links             | Poor accessibility | Descriptive link text                               |
| H1 just project name         | Poor SEO           | Include core function                               |
| "The Problem / The Solution" | Cliched, lazy      | Show value directly                                 |

---

## Step 7: Quality Metrics

| Metric                 | Target          | Action if Violated     |
| ---------------------- | --------------- | ---------------------- |
| Flesch-Kincaid         | Grade 8-10      | Simplify sentences     |
| Time to Joy            | <=3 commands    | Add Docker/Makefile    |
| Visual density         | 1 per 300 words | Add code block/diagram |
| Badge count            | 5-7             | Curate to essentials   |
| Quick start visibility | < 30 seconds    | Move above the fold    |
| Delve Index            | A0; B≤1; C≤3; D≤5 | Trim the over-budget tier |

---

## Step 8: Verification Against Previous Steps

**Before finalising, cross-check against each preparatory step:**

### Deep Dive Verification

- [ ] Tech stack mentioned matches deep-dive findings
- [ ] Prerequisites include all system dependencies identified
- [ ] Entry points documented match identified entry points
- [ ] Installation steps work for the detected ecosystem

### Crystal Ball Verification

- [ ] Roadmap section included (if opportunities identified)
- [ ] Future features mentioned align with crystal-ball analysis
- [ ] Technical debt items acknowledged where relevant
- [ ] Version/maturity signals match project state

### Brain-Jam Verification

- [ ] Tone matches the chosen angle (Deep Tech vs Pragmatic vs etc.)
- [ ] Voice is consistent throughout
- [ ] Marketing angle applied to hero section and description
- [ ] Target audience language used appropriately
- [ ] Undefended assumptions from brain-jam.md are either substantiated or absent from the README
- [ ] No README claim aligns with an OUT argument from the argument map

### Think-Tank Verification

- [ ] Badge selection matches think-tank recommendations
- [ ] Section order follows the researched blueprint
- [ ] Visual strategy implemented as planned
- [ ] Patterns borrowed from exemplars applied correctly
- [ ] Anti-patterns from research avoided

---

## Step 9: The Final Audit

Before outputting, review against `references/anti-patterns.md`:

- [ ] Are paragraphs < 4 sentences?
- [ ] Are M-dashes (—) removed?
- [ ] Is the "Delve" index 0?
- [ ] Are headers semantic (e.g., "Zero-Config Build" vs "Features")?
- [ ] Does the Quick Start work in < 30 seconds?
- [ ] No "The Problem / The Solution" framing used?

---

## Output

Present the final README in a single, copy-pasteable Markdown block.

**Ask:** "README complete. Want to refine anything, or ship it?"

- If refine -> make edits
- If ship -> write to README.md, commit, done
