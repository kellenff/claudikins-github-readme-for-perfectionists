# Graph-Analysis Tools — Shared Reference

**Plugin-wide reference for all `claudikins-grfp` skills.**

This document covers the codebase-memory graph-analysis tools available via the `claudikins-tool-executor` plugin. Skills in this pipeline use these tools for semantic codebase understanding — symbol relationships, call chains, architectural overview — that text-based tools (Read, Grep, Glob) cannot answer.

---

## When to Reach for Graph Tools

| Question                                           | Tool                                                             |
| -------------------------------------------------- | ---------------------------------------------------------------- |
| "What are the entry points of this codebase?"      | `search_graph` (label: Function, direction: inbound, min_degree) |
| "What does this function call, and what calls it?" | `trace_path` (direction: both)                                   |
| "What is the architectural shape?"                 | `get_architecture`                                               |
| "Is this symbol dead code?"                        | `search_graph` (max_degree: 0, direction: inbound)               |
| "Read the source of this exact function"           | `get_code_snippet`                                               |
| "Run a custom structural query"                    | `query_graph` (Cypher)                                           |
| "Text/regex search across indexed code"            | `search_code`                                                    |

**Prefer graph tools over Read/Grep/Glob when** the question is about _symbol relationships_, _call structure_, _entry points_, or _dead-code detection_.

**Prefer Read/Grep/Glob over graph tools when** the question is about _file content_, _config values_, _string constants_, or _unindexed files_ (docs, manifests, lockfiles).

---

## Access Pattern: Tool-Executor 3-Step Workflow

Graph tools are **not** called directly. They are accessed via the `claudikins-tool-executor` plugin using the canonical 3-step workflow:

```
1. search_tools(query)       → discover available tools
2. get_tool_schema(name)     → fetch exact input schema
3. execute_code(code)        → run the tool inside the sandbox
```

Inside `execute_code`, graph tools are exposed via the `serena` client (or codebase-memory client when wired separately).

### Example

```typescript
// Step 1 — discover (only needed if you don't know the tool name)
const tools = await search_tools("search_graph");

// Step 2 — schema (only needed for unfamiliar parameter shapes)
const schema = await get_tool_schema("search_graph");

// Step 3 — execute
await execute_code(`
  const hits = await serena.search_graph({
    label: "Function",
    name_pattern: ".*[Hh]andler.*",
    direction: "inbound",
    min_degree: 5,
    limit: 25
  });
  await workspace.writeJSON("research/handlers.json", hits);
`);
```

---

## Standard Opener (Phase 0 from grfp orchestrator)

Before any skill performs codebase analysis, the `grfp` orchestrator runs a Phase 0 check:

```typescript
await execute_code(`
  // 1. Check if the repo is indexed
  const status = await serena.index_status({});

  // 2. Auto-index if missing
  if (!status.indexed) {
    try {
      await serena.index_repository({ path: process.cwd() });
    } catch (e) {
      // Graceful fallback — skills should detect this and fall back to Read/Glob/Grep
      await workspace.writeJSON("graph-unavailable.json", { reason: e.message });
      return;
    }
  }

  // 3. Cache architecture overview for downstream skills
  const arch = await serena.get_architecture({
    aspects: ["services", "packages", "entry_points"]
  });
  await workspace.writeJSON("research/architecture.json", arch);
`);
```

---

## Graceful Degradation

**If `claudikins-tool-executor` is not installed, or indexing fails, skills MUST fall back to basic tools.**

Detection pattern:

```typescript
// At the top of any skill workflow step using graph tools:
let graphAvailable = false;
try {
  const status = await execute_code(`await serena.index_status({})`);
  graphAvailable = status?.indexed === true;
} catch {
  graphAvailable = false;
}

if (graphAvailable) {
  // Use search_graph, trace_path, etc.
} else {
  // Fall back to Read, Glob, Grep with existing heuristics
}
```

Each skill documents its specific fallback path in its own `## Graph Tools` section.

---

## Tool Catalogue

### `search_graph`

Find nodes (functions, classes, files) by name pattern, label, or graph-degree filters.

**Key parameters:**

- `name_pattern` (regex) — match against symbol names
- `label` — `Function`, `Class`, `File`, etc.
- `qn_pattern` (regex) — match against qualified names
- `min_degree` / `max_degree` — filter by graph centrality
- `relationship` — `CALLS`, `IMPORTS`, `DEFINES`, etc.
- `direction` — `inbound | outbound | both`
- `exclude_entry_points`, `offset`, `limit`

**Best for:** Discovering entry points, controllers, handlers, hot symbols, dead code.

**Example:**

```typescript
await serena.search_graph({
  label: "Function",
  name_pattern: ".*Handler.*",
  direction: "inbound",
  min_degree: 5,
  limit: 25,
});
```

---

### `trace_path`

Trace call chains, data flow, or cross-service edges starting from a named function.

**Key parameters:**

- `function_name` — exact name (discover via `search_graph` first)
- `direction` — `inbound | outbound | both`
- `depth` — default ~3
- `mode` — `calls | data_flow | cross_service`
- `risk_labels` — bool

**Best for:** Mapping how requests flow through the app, finding which subsystems are central, identifying validation gaps from entry → sink.

**Example:**

```typescript
await serena.trace_path({
  function_name: "handleRequest",
  direction: "both",
  depth: 3,
  mode: "calls",
});
```

---

### `get_code_snippet`

Read the source of a node by qualified name.

**Key parameters:**

- `qualified_name` — e.g. `"src.server.bootstrap"`

**Best for:** Reading the exact source of a graph-discovered symbol without grepping or full-file reads.

**Example:**

```typescript
await serena.get_code_snippet({
  qualified_name: "src.routes.user.createUser",
});
```

---

### `query_graph`

Run arbitrary Cypher against the knowledge graph.

**Key parameters:**

- `query` — Cypher string (200-row result cap)

**Best for:** Custom structural questions — counting HTTP endpoints, listing cross-service edges, finding files that change together.

**Example:**

```typescript
await serena.query_graph({
  query:
    "MATCH (a)-[r:HTTP_CALLS]->(b) RETURN a.name, b.name, r.url_path LIMIT 20",
});
```

---

### `get_architecture`

Return a high-level architectural view: packages, services, layers, entry points.

**Key parameters:**

- `aspects` — array: `services`, `packages`, `layers`, `entry_points`, `data_stores`

**Best for:** Single-call architectural overview. Ideal first call when writing the "Architecture" section of a README, or generating a Mermaid module diagram.

**Example:**

```typescript
await serena.get_architecture({
  aspects: ["services", "packages", "entry_points"],
});
```

---

### `search_code`

Text/regex search across the indexed corpus.

**Key parameters:**

- `pattern` — regex
- optional path/file filters

**Best for:** Finding config keys, env var names, log strings, error messages, version constants. Use only when graph queries can't answer the question.

**Example:**

```typescript
await serena.search_code({ pattern: "process\\.env\\.[A-Z_]+" });
```

---

### `index_repository`

Build (or rebuild) the knowledge graph for a repo.

**Key parameters:** repo path / project identifier.

**Best for:** Required first step before any graph tool returns results. Run once per fresh repo.

---

### `index_status`

Report whether the index exists and is current.

**Key parameters:** project identifier (optional).

**Best for:** Phase 0 check at the top of `grfp` orchestrator to decide whether to auto-index.

---

### `detect_changes`

Map current git diff to affected symbols in the graph.

**Best for:** "What's new in this PR" sections; less central for README authoring.

---

## Edge Types (for `query_graph` / `trace_path` filters)

```
CALLS, HTTP_CALLS, ASYNC_CALLS, IMPORTS, DEFINES, DEFINES_METHOD,
HANDLES, IMPLEMENTS, OVERRIDE, USAGE, FILE_CHANGES_WITH,
CONTAINS_FILE, CONTAINS_FOLDER, CONTAINS_PACKAGE
```

---

## Gotchas

1. **`trace_path` requires exact names** — always call `search_graph(name_pattern=...)` first to discover qualified names.
2. **`query_graph` caps at 200 rows** — use `search_graph` with degree filters for counting.
3. **`search_graph(relationship="HTTP_CALLS")` filters nodes by degree, not edges** — use `query_graph` if you need actual edge data.
4. **`direction="outbound"` misses cross-service callers** — prefer `direction="both"` for completeness.
5. **Default page size is 10** — check `has_more` and use `offset` for pagination.
6. **Index can be stale** — call `detect_changes` after recent git activity to validate.

---

## Per-Skill Usage Patterns

Each skill's `SKILL.md` documents how that specific phase uses graph tools. See:

- `skills/deep-dive/SKILL.md` — entry-point detection, project-type analysis, API surface mapping
- `skills/crystal-ball/SKILL.md` — dead-code detection, complexity hotspots, deprecated API callers
- `skills/grfp/SKILL.md` — Phase 0 index check with graceful fallback
- `skills/pen-wielding/SKILL.md` — auto-generated architecture diagrams
- `skills/brain-jam/SKILL.md` — reference link only (voice-strategy phase, no codebase introspection)
- `skills/think-tank/SKILL.md` — reference link only (external research phase, no codebase introspection)

---

## Related Reading

- `/Users/kellen/.claude/skills/codebase-memory/SKILL.md` — canonical tool reference (all 14 tools)
- `claudikins-tool-executor` plugin: `te-guide/SKILL.md` — 3-tool workflow + workspace API
