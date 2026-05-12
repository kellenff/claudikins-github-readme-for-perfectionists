---
name: deep-dive
description: "Use when starting the README pipeline to extract technical facts from codebase. Identifies project type, tech stack, dependencies, entry points, CI configuration. Uses graph-analysis tools for semantic codebase understanding. Outputs Reality Report."
---

# The Deep Dive

**Goal:** Establish the ground truth of the codebase (what IS) before imagining what could be.

This skill uses **graph-analysis tools** (search_graph, trace_path, get_architecture) as the primary method for understanding codebase structure. Text-based tools (Read, Glob, Grep) are reserved for non-indexed content (config files, lockfiles, manifests, docs).

## Workflow

**Step 0: Load Intelligence**

- Read `references/what-to-extract.md` for the extraction checklist.
- Read `references/graph-analysis.md` (or the shared `references/graph-analysis.md` at the plugin root) for the tool-executor workflow and gotchas.

**Step 1: Verify Graph Index** (Phase 0 from grfp orchestrator)

If invoked via `grfp`, the index is already verified. Otherwise:

```typescript
await execute_code(`
  const status = await serena.index_status({});
  if (!status.indexed) {
    try {
      await serena.index_repository({ path: process.cwd() });
    } catch (e) {
      await workspace.writeJSON("graph-unavailable.json", { reason: e.message });
    }
  }
`);
```

If graph tools are unavailable, fall back to the filename-heuristic table in section 1 below.

1. **Architectural Overview**: Call `get_architecture` for packages, services, entry points.
2. **Identify Configuration**: Read dependency files (`package.json`, `Cargo.toml`, `pyproject.toml`) — these are not in the graph.
3. **Extract Entry Points**: Use `search_graph` + `trace_path` for semantic entry-point discovery.
4. **Context Extraction**: Scan chat history for user struggles/intent (if available).
5. **Synthesize**: Generate the "Codebase Reality Report".

---

## 1. Project Type Detection

### Graph-Based Detection (Primary)

```typescript
await execute_code(`
  const arch = await serena.get_architecture({
    aspects: ["services", "packages", "entry_points", "data_stores"]
  });

  // High-fan-in symbols indicate central code
  const central = await serena.search_graph({
    label: "Function",
    direction: "inbound",
    min_degree: 10,
    limit: 20
  });

  await workspace.writeJSON("deep-dive/architecture.json", { arch, central });
`);
```

**Interpretation:**

| Architecture Signal                                      | Likely Type |
| -------------------------------------------------------- | ----------- |
| `entry_points` includes argparse/commander registrations | CLI tool    |
| `entry_points` includes HTTP route handlers              | API/Server  |
| `services` array empty, `packages` with public exports   | Library     |
| `entry_points` includes main/bootstrap, no HTTP routes   | Application |
| `aspects` returns plugin manifest entries                | Plugin      |

### Filename Fallback (only if graph tools unavailable)

| Files Present                    | Likely Type |
| -------------------------------- | ----------- |
| `bin/`, shebang in main          | CLI tool    |
| `src/index.ts` with exports      | Library     |
| `src/main.ts` or `app.ts`        | Application |
| `plugin.json`, `.claude-plugin/` | Plugin      |
| `src/server.ts`, `app.py`        | API/Server  |
| `Dockerfile` only                | Container   |
| `setup.py` with console_scripts  | Python CLI  |
| `Cargo.toml` with [[bin]]        | Rust CLI    |

---

## 2. Tech Stack Extraction

**Dependency files are not graph-indexed — read them directly.**

### Dependency Files

| File                                 | Ecosystem          |
| ------------------------------------ | ------------------ |
| `package.json`                       | Node.js/JavaScript |
| `requirements.txt`, `pyproject.toml` | Python             |
| `Cargo.toml`                         | Rust               |
| `go.mod`                             | Go                 |
| `Gemfile`                            | Ruby               |
| `pom.xml`, `build.gradle`            | Java               |
| `composer.json`                      | PHP                |

### Framework Detection via Graph

Graph imports reveal frameworks more reliably than reading manifests:

```typescript
await execute_code(`
  // Find which external packages are imported most heavily
  const imports = await serena.query_graph({
    query: \`
      MATCH (f:File)-[:IMPORTS]->(p:Package)
      WHERE p.external = true
      RETURN p.name, count(f) as usage_count
      ORDER BY usage_count DESC
      LIMIT 20
    \`
  });
  await workspace.writeJSON("deep-dive/frameworks.json", imports);
`);
```

### Build Tool Detection

| Tool       | Files                         |
| ---------- | ----------------------------- |
| TypeScript | `tsconfig.json`               |
| Webpack    | `webpack.config.js`           |
| Vite       | `vite.config.ts`              |
| esbuild    | `esbuild` in scripts          |
| Rollup     | `rollup.config.js`            |
| Babel      | `.babelrc`, `babel.config.js` |

---

## 3. Dependencies & Prerequisites

### Runtime Requirements

**From package.json:**

```json
{ "engines": { "node": ">=18.0.0", "npm": ">=9.0.0" } }
```

**From pyproject.toml:**

```toml
[project]
requires-python = ">=3.10"
```

**From Cargo.toml:**

```toml
[package]
rust-version = "1.70"
```

### System Dependencies

**Check these files:**

- `Dockerfile` - RUN apt-get install commands
- `.github/workflows/*.yml` - setup steps
- `Makefile` - prerequisite checks
- `INSTALL.md` or `docs/installation.md`

**Common system deps:**

- `libssl-dev` - cryptography
- `libpq-dev` - PostgreSQL
- `ffmpeg` - media processing
- `graphviz` - diagram generation

---

## 4. Entry Points & API Surface

**This section uses graph tools as primary — filename heuristics are unreliable.**

### Find Entry Points Semantically

```typescript
await execute_code(`
  // 1. Inbound-degree-0 functions are roots (entry points)
  const roots = await serena.search_graph({
    label: "Function",
    direction: "inbound",
    max_degree: 0,
    limit: 50
  });

  // 2. From each root, trace outbound calls (the "main flow")
  const flows = [];
  for (const root of roots.slice(0, 5)) {
    const trace = await serena.trace_path({
      function_name: root.qualified_name,
      direction: "outbound",
      depth: 3
    });
    flows.push({ root: root.name, trace });
  }

  await workspace.writeJSON("deep-dive/entry-points.json", { roots, flows });
`);
```

### Map the Public API Surface

```typescript
await execute_code(`
  // Exported symbols only — qn_pattern matches public namespace
  const publicApi = await serena.search_graph({
    label: "Function",
    qn_pattern: ".*\\\\.(index|exports|public)\\\\..*",
    limit: 100
  });
  await workspace.writeJSON("deep-dive/public-api.json", publicApi);
`);
```

### Read Source of Key Entry Points

For each identified entry point, fetch the exact source:

```typescript
await execute_code(`
  const snippet = await serena.get_code_snippet({
    qualified_name: "src.cli.main"  // or whatever the graph returned
  });
  await workspace.writeJSON("deep-dive/entry-source.json", snippet);
`);
```

**Value proposition sources** (still read from files — not in graph):

- `package.json` description field
- Module-level docstrings
- JSDoc on main exports
- Existing README fragments

---

## 5. CI/Automation Detection

**CI files are not graph-indexed — read directly.**

### GitHub Actions

**Badge sources from workflows:**

- Workflow status - Build badge
- `codecov/codecov-action` - Coverage badge
- Release workflow - Version badge

### Other CI

| Service   | Config File            |
| --------- | ---------------------- |
| CircleCI  | `.circleci/config.yml` |
| Travis    | `.travis.yml`          |
| GitLab CI | `.gitlab-ci.yml`       |
| Jenkins   | `Jenkinsfile`          |

---

## 6. Chat History Context

**Source:** `~/.claude/history.jsonl`

Filter entries where `project` matches the current project path. (Chat history is not graph-indexed.)

### Pattern Extraction

**Repeated questions (knowledge gaps):**

- "How do I..." patterns
- "Why isn't..." patterns
- "What's the difference between..." patterns

**Problem categories:**

- Configuration/Setup
- Debugging/Errors
- Feature implementation
- Performance
- Testing
- Deployment

### README Implications

| Pattern Found             | README Implication             |
| ------------------------- | ------------------------------ |
| Repeated config questions | Detailed configuration section |
| Debugging struggles       | Troubleshooting section        |
| "How do I test" questions | Testing section with examples  |
| Performance questions     | Performance section            |
| Deployment questions      | Deployment guides              |

---

## Output Format

```markdown
# Deep Dive Findings

## Project Overview

- **Type:** [CLI/library/framework/plugin/app/API]
- **Tech Stack:** [languages, frameworks]
- **Value Proposition:** [1-2 sentence problem/solution]
- **Entry Point:** [main file/command]
- **Architecture (from get_architecture):** [packages, services, layers]

## Dependencies

- **Runtime:** [Node 18+, Python 3.10+, etc.]
- **System:** [any system dependencies]
- **Dev:** [dev dependencies of note]
- **Most-imported packages (from graph):** [top 5 by usage_count]

## Entry Points (from search_graph + trace_path)

- [qualified_name] - [what it does]
- [main flow trace: A → B → C]

## CI/Automation

- **Build:** [workflow files]
- **Badge Sources:** [CI, coverage, package manager]

## User Context (from Chat History)

- **Common Struggles:** [repeated questions]
- **Decisions Made:** [key decisions from history]
- **Focus Areas:** [topics frequently discussed]

## Missing Information (Blockers)

- [ ] [List anything you need but couldn't find]

## Graph Index State

- **Indexed:** [Yes/No]
- **Method used:** [graph / filename-fallback]
```

## Important

- Focus on FACTS, not improvements (that's crystal-ball's job)
- Be specific with qualified-name or file:line references
- Note confidence levels:
  - **High:** Extracted from graph or config files
  - **Medium:** Inferred from import patterns or filename heuristics
  - **Low:** Guessed from structure
- If graph tools were unavailable, note this in the "Missing Information" section

## Graph Tools Used by This Skill

- `get_architecture` — packages/services/entry points overview
- `search_graph` — entry-point discovery (max_degree filters), public API mapping (qn_pattern), heavily-used symbols (min_degree)
- `trace_path` — "main flow" reconstruction from entry points
- `get_code_snippet` — reading source of graph-identified symbols
- `query_graph` — import-frequency analysis for framework detection

See `references/graph-analysis.md` for full tool documentation and the tool-executor 3-step workflow.

## Handoff

**After presenting the Reality Report:**

1. **Ask:** "Does this accurately represent the codebase? Any missing information we need to clarify?"
2. **Wait** for user confirmation.
3. **Transition:**
   - If changes needed: Refine the Deep Dive.
   - If approved: **"Proceeding to crystal-ball to predict what this project COULD become."**
