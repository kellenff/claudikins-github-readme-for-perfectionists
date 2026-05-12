---
name: crystal-ball
description: "Use when analysing future opportunities after deep-dive phase. Identifies performance improvements, technical debt, security hardening, and feature gaps. Uses graph-analysis tools for dead-code detection, call-chain reasoning, and complexity hotspots. Outputs roadmap candidates table."
---

# The Crystal Ball

**Goal:** Identify what the codebase **COULD BE**, not what it IS.

This skill uses **graph-analysis tools** to answer structural questions — "what's dead?", "what's deeply nested?", "who calls this deprecated symbol?" — that text-based tools can't reliably answer. Falls back to Read/Grep for content-level questions (TODO comments, version strings, hardcoded secrets).

## Step 1: Load References

- Read `references/roadmap-patterns.md` for presentation patterns and anti-patterns.
- Read the shared `references/graph-analysis.md` at the plugin root for the tool-executor workflow, tool catalogue, and gotchas.

## Step 2: Verify Graph Index

If invoked via `grfp`, the index is already verified. Otherwise check `serena.index_status` before any graph call. If graph tools are unavailable, fall back to the manual analysis patterns at the end of each section.

---

## Analysis Checklist

### 1. Complexity Analysis (Priority)

**Graph-based hotspot detection (primary):**

```typescript
await execute_code(`
  // High fan-in functions = central; high fan-out = orchestrators
  const fanIn = await serena.search_graph({
    label: "Function",
    direction: "inbound",
    min_degree: 15,
    limit: 10
  });

  const fanOut = await serena.search_graph({
    label: "Function",
    direction: "outbound",
    min_degree: 15,
    limit: 10
  });

  // Deep call chains from each hotspot indicate complexity
  const chains = [];
  for (const fn of fanIn.slice(0, 5)) {
    const trace = await serena.trace_path({
      function_name: fn.qualified_name,
      direction: "outbound",
      depth: 5
    });
    chains.push({ name: fn.name, depth: trace.depth, calls: trace.edges?.length });
  }

  await workspace.writeJSON("crystal-ball/complexity.json", { fanIn, fanOut, chains });
`);
```

**Document in plain English:** "This search is O(n²) because it compares every item to every other item. Could be O(n log n) with sorting first."

For each hotspot, fetch the source with `get_code_snippet` and read it for actual complexity (nested loops, recursion).

**Fallback (no graph):** Manual reading of obvious algorithmic code.

### 2. Performance Opportunities

Look for:
- Unnecessary iterations that could be reduced
- Missing caching for expensive operations
- Synchronous operations that could be parallel
- Memory allocations that could be pooled
- Network calls that could be batched

**Graph-based detection of parallelisable chains:**

```typescript
await execute_code(`
  // Find sequential await chains (often parallelisable with Promise.all)
  const chains = await serena.query_graph({
    query: \`
      MATCH (a:Function)-[:ASYNC_CALLS*2..4]->(b:Function)
      WHERE NOT (a)-[:CALLS {parallel: true}]->()
      RETURN a.qualified_name, length((a)-[:ASYNC_CALLS*]->(b)) as chain_depth
      ORDER BY chain_depth DESC LIMIT 20
    \`
  });
  await workspace.writeJSON("crystal-ball/async-chains.json", chains);
`);
```

### 3. Code Quality Issues

**Graph-based dead-code detection (primary):**

```typescript
await execute_code(`
  // Orphaned symbols: defined but never called/imported
  const dead = await serena.search_graph({
    label: "Function",
    direction: "inbound",
    max_degree: 0,
    exclude_entry_points: true,
    limit: 50
  });

  // Same for classes
  const deadClasses = await serena.search_graph({
    label: "Class",
    direction: "inbound",
    max_degree: 0,
    limit: 20
  });

  await workspace.writeJSON("crystal-ball/dead-code.json", { functions: dead, classes: deadClasses });
`);
```

**Then read each candidate** with `get_code_snippet` to confirm it's truly dead (not just dynamically called).

Other content-level checks (Read/Grep):

- Outdated dependencies (check package.json versions)
- TODO/FIXME comments (extract and list)
- Inconsistent patterns (mixed async styles, naming conventions)

**Fallback (no graph):** Grep for `export` keyword, then grep for callers.

### 4. Security Hardening

**Graph-based attack-surface tracing:**

```typescript
await execute_code(`
  // From every public entry point, trace outbound paths to dangerous sinks
  const entries = await serena.search_graph({
    label: "Function",
    relationship: "HANDLES",
    direction: "inbound",
    limit: 50
  });

  const paths = [];
  for (const entry of entries) {
    const trace = await serena.trace_path({
      function_name: entry.qualified_name,
      direction: "outbound",
      depth: 4,
      risk_labels: true
    });
    if (trace.has_risk_label) paths.push({ entry: entry.name, trace });
  }

  await workspace.writeJSON("crystal-ball/attack-surface.json", paths);
`);
```

Content-level (Read/Grep):

- Hardcoded secrets (API keys, passwords in code)
- Unsafe dependencies (check for known vulnerabilities)
- Missing rate limiting on public endpoints

### 5. Technical Debt

**Find callers of deprecated APIs:**

```typescript
await execute_code(`
  // Locate deprecated symbols then enumerate their callers
  const deprecated = await serena.search_graph({
    label: "Function",
    name_pattern: ".*[Dd]eprecated.*",  // or use comment markers if indexed
    limit: 50
  });

  const callers = [];
  for (const sym of deprecated) {
    const trace = await serena.trace_path({
      function_name: sym.qualified_name,
      direction: "inbound",
      depth: 1
    });
    callers.push({ deprecated: sym.name, callers: trace.nodes });
  }

  await workspace.writeJSON("crystal-ball/deprecated-callers.json", callers);
`);
```

Other content-level checks:

- Inconsistent patterns across codebase
- Missing tests for critical paths
- Documentation gaps (CONTRIBUTING.md, examples/)
- Workarounds marked with comments

### 6. Feature Gaps

Based on project type (from deep-dive's `get_architecture`), identify:

- Common features for this type that are missing
- User-requested features (check GitHub issues if accessible)
- Competitive features (what similar tools have)

## Output Format

**REQUIRED:** Return the analysis in this specific markdown format so `pen-wielding` can read it.

```markdown
# Roadmap Candidates

## Performance Opportunities

| Location           | Current              | Opportunity             | Impact |
| ------------------ | -------------------- | ----------------------- | ------ |
| `src/search.ts:45` | O(n²) nested loops   | Use Map for O(1) lookup | High   |
| `src/api.ts:120`   | Sequential API calls | Batch with Promise.all  | Medium |

## Technical Debt

| Location          | Issue                  | Suggested Fix       |
| ----------------- | ---------------------- | ------------------- |
| `src/utils.ts:30` | TODO from 6 months ago | Implement or remove |
| `package.json`    | lodash@3.x outdated    | Upgrade to 4.x      |

## Feature Gaps

| Feature             | Why It Matters                 | Effort |
| ------------------- | ------------------------------ | ------ |
| Config file support | Users want persistent settings | Medium |
| Fluent data access  | Stop two-step retrieval dance  | Low    |

## Security Hardening

| Location        | Risk                  | Recommendation      |
| --------------- | --------------------- | ------------------- |
| `src/cli.ts:15` | No input sanitisation | Validate before use |

## Complexity Notes

| Function        | Complexity | Plain English                              |
| --------------- | ---------- | ------------------------------------------ |
| `findMatches()` | O(n²)      | Compares every item - slow for large lists |

## Dead Code (from graph)

| Symbol          | Location               | Confidence             |
| --------------- | ---------------------- | ---------------------- |
| `legacyParseV1` | `src/parser/legacy.ts` | High (0 inbound edges) |

## Community Help Wanted

| Task      | Context                                    |
| --------- | ------------------------------------------ |
| Add Tests | `src/core/` has 0% coverage                |
| Examples  | Create `examples/` folder with basic usage |
```

## Important

- Focus on README-worthy improvements (things users/contributors care about)
- Be specific with file:line OR qualified-name references
- Explain WHY something is an opportunity, not just WHAT
- Prioritise by impact (High/Medium/Low)
- Don't overwhelm — top 5-10 items per category max
- For dead-code findings, always confirm with `get_code_snippet` before listing (avoid false positives from dynamic dispatch)

## Graph Tools Used by This Skill

- `search_graph` — hotspot detection (min_degree), dead-code detection (max_degree: 0), deprecated-symbol discovery
- `trace_path` — call-chain depth, attack-surface paths from entry points, caller enumeration for deprecated APIs
- `query_graph` — async-chain detection, custom structural queries
- `get_code_snippet` — confirming dead-code candidates, reading complexity hotspots

See `references/graph-analysis.md` at the plugin root for full tool documentation.

## Handoff

**Goal:** Transition from technical analysis to creative brainstorming.

1. **Present Findings:** Show the "Roadmap Candidates" tables.
2. **Verify:** Ask the user:
   > "The Crystal Ball has spoken. Happy with these insights, or want to explore more? Ready for a brain-jam with Gemini?"
3. **Transition:**
   - If user wants changes: Refine the analysis.
   - If user is satisfied: **"Proceeding to brain-jam."**
