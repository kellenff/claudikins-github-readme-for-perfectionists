---
name: grfp
description: "Use when creating or updating a README. It enforces the 5-step 'Anti-Slop' pipeline and triggers the correct sub-skill for the current phase. Runs a Phase 0 graph-index check before any codebase analysis begins."
---

# README Pipeline Orchestrator

**Goal:** Manage the lifecycle of README generation. Ensure no steps are skipped.

## The 5-Phase Pipeline

You MUST execute these phases in order. Do not skip to drafting until research is complete.

1. **Discovery:** `deep-dive` (Get the Facts)
2. **Audit:** `crystal-ball` (Get the Roadmap)
3. **Strategy:** `brain-jam` (Get the Voice)
4. **Research:** `think-tank` (Get the Patterns)
5. **Execution:** `pen-wielding` (Write the Artifact)

---

## Phase 0: Graph Index Check (runs ONCE before Phase 1)

**Before triggering `deep-dive`, check whether the codebase graph is indexed.**

This amortises the indexing cost across all downstream phases. `deep-dive` and `crystal-ball` both rely on graph tools — they must not pay the indexing cost independently.

```typescript
await execute_code(`
  const status = await serena.index_status({});

  if (!status.indexed) {
    try {
      await serena.index_repository({ path: process.cwd() });
      await workspace.writeJSON("graph-status.json", { available: true });
    } catch (e) {
      // Graceful fallback: downstream skills will use Read/Glob/Grep
      await workspace.writeJSON("graph-status.json", {
        available: false,
        reason: e.message,
        fallback: "Skills will use filename-heuristic analysis"
      });
    }
  } else {
    await workspace.writeJSON("graph-status.json", { available: true });
  }
`);
```

**Failure handling:**

- If `index_repository` throws: write `graph-status.json` with `available: false` and continue.
- Warn the user once: _"Graph index unavailable — analysis will use file-based heuristics instead."_
- Do NOT hard-fail. The pipeline continues with graceful fallback in downstream skills.
- Do NOT re-attempt indexing in `deep-dive` or `crystal-ball` if Phase 0 already ran.

**See `references/graph-analysis.md` (plugin root) for full tool documentation.**

---

## Workflow Logic (The State Machine)

**Before acting, checks the current context to decide which skill to trigger.**

### Phase 1: Missing Facts?

_Condition:_ If you do not have a **"Reality Report"** (from Deep Dive) in the chat history...

_Action:_ **Invoke `deep-dive` immediately.**

_Do not ask:_ "Shall we start?" Just start.

### Phase 2: Missing Roadmap?

_Condition:_ If you have the Reality Report, but lack a **"Roadmap & Future Improvements"** table...

_Action:_ **Invoke `crystal-ball` immediately.**

### Phase 3: Missing Voice?

_Condition:_ If you have Facts + Roadmap, but have not agreed on an **"Angle"** (Deep Tech vs. Pragmatist)...

_Action:_ **Invoke `brain-jam` immediately.**

### Phase 4: Missing Patterns?

_Condition:_ If you have the Angle, but lack a **"Research Results"** audit of similar repos...

_Action:_ **Invoke `think-tank` immediately.**

### Phase 5: Ready to Write?

_Condition:_ If you have Facts + Roadmap + Angle + Patterns...

_Action:_ **Invoke `pen-wielding` to generate the final artifact.**

---

## Handling Handoffs

When a sub-skill completes (e.g., `deep-dive` finishes), do not stop.

1. **Review the output.**
2. **Check the Pipeline Logic** above.
3. **Trigger the next skill** in the chain automatically.

_Example:_ "Deep Dive is complete. I see the Reality Report. Moving to Phase 2: Invoking `crystal-ball`..."
