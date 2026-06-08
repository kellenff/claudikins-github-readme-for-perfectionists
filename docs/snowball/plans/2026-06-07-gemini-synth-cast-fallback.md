# Gemini Synth Cast + Provider Fallback Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use snowball:subagent-driven-development (recommended) or snowball:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Gemini 3.5 Flash as the default synth voice, reduce brain-jam to 2 rounds, and select a fallback cast when only one provider API key is available.

**Architecture:** Three Chorus recipe JSON files (primary heterogeneous + two homogeneous fallbacks). The brain-jam skill gains Step 4.5 to probe `GEMINI_API_KEY` / `MINIMAX_API_KEY` and pick the recipe before shelling out to Chorus. Participant names stay fixed so the transcript renderer is unchanged except for an optional fallback footnote.

**Tech Stack:** GRFP skill markdown, Chorus cast JSON, bash key probing. No Chorus repo changes.

**Spec:** `docs/snowball/specs/2026-06-07-gemini-synth-cast-fallback-design.md`

---

## File Structure

| File | Action |
| --- | --- |
| `skills/brain-jam/recipes/grfp-readme.json` | Modify — Gemini synth |
| `skills/brain-jam/recipes/grfp-readme.fallback-minimax.json` | Create |
| `skills/brain-jam/recipes/grfp-readme.fallback-gemini.json` | Create |
| `skills/brain-jam/SKILL.md` | Modify — Step 4.5, max-rounds 2, fallback footnote |
| `README.md` | Modify — keys, diagram, rounds |
| `CHANGELOG.md` | Modify — 3.1.0 entry |
| `commands/brain-jam.md` | Modify — one-line cast note |

---

### Task 1: Primary cast — Gemini synth

**Files:**
- Modify: `skills/brain-jam/recipes/grfp-readme.json`

- [ ] **Step 1: Update synth model to Gemini**

Replace the file contents with:

```json
{
  "participants": [
    {
      "name": "synth",
      "model": "gemini-3.5-flash",
      "persona": "You are a senior engineer whose excitement is technical, not marketing. Build on the prior turn — find what's interesting and raise a new angle; don't just agree.",
      "temperature": 0.8
    },
    {
      "name": "pragmatist",
      "model": "MiniMax-M3",
      "persona": "You are a pragmatist focused on what devs actually need, skeptical of hype. Push back on shallow excitement. Concrete examples only.",
      "temperature": 0.5
    }
  ],
  "critique": true,
  "critic": {
    "name": "critic",
    "model": "MiniMax-M3",
    "persona": "",
    "temperature": 0.3
  }
}
```

- [ ] **Step 2: Validate JSON parses**

Run:

```bash
python3 -m json.tool skills/brain-jam/recipes/grfp-readme.json > /dev/null && echo OK
```

Expected: `OK`

---

### Task 2: Fallback cast recipes

**Files:**
- Create: `skills/brain-jam/recipes/grfp-readme.fallback-minimax.json`
- Create: `skills/brain-jam/recipes/grfp-readme.fallback-gemini.json`

- [ ] **Step 1: Create MiniMax-only fallback**

Write `skills/brain-jam/recipes/grfp-readme.fallback-minimax.json`:

```json
{
  "participants": [
    {
      "name": "synth",
      "model": "MiniMax-M3",
      "persona": "You are a senior engineer whose excitement is technical, not marketing. Build on the prior turn — find what's interesting and raise a new angle; don't just agree.",
      "temperature": 0.8
    },
    {
      "name": "pragmatist",
      "model": "MiniMax-M3",
      "persona": "You are a pragmatist focused on what devs actually need, skeptical of hype. Push back on shallow excitement. Concrete examples only.",
      "temperature": 0.5
    }
  ],
  "critique": true,
  "critic": {
    "name": "critic",
    "model": "MiniMax-M3",
    "persona": "",
    "temperature": 0.3
  }
}
```

- [ ] **Step 2: Create Gemini-only fallback**

Write `skills/brain-jam/recipes/grfp-readme.fallback-gemini.json`:

```json
{
  "participants": [
    {
      "name": "synth",
      "model": "gemini-3.5-flash",
      "persona": "You are a senior engineer whose excitement is technical, not marketing. Build on the prior turn — find what's interesting and raise a new angle; don't just agree.",
      "temperature": 0.8
    },
    {
      "name": "pragmatist",
      "model": "gemini-3.5-flash",
      "persona": "You are a pragmatist focused on what devs actually need, skeptical of hype. Push back on shallow excitement. Concrete examples only.",
      "temperature": 0.5
    }
  ],
  "critique": true,
  "critic": {
    "name": "critic",
    "model": "gemini-3.5-flash",
    "persona": "",
    "temperature": 0.3
  }
}
```

- [ ] **Step 3: Validate both parse**

Run:

```bash
python3 -m json.tool skills/brain-jam/recipes/grfp-readme.fallback-minimax.json > /dev/null && \
python3 -m json.tool skills/brain-jam/recipes/grfp-readme.fallback-gemini.json > /dev/null && \
echo OK
```

Expected: `OK`

---

### Task 3: brain-jam skill — cast selection and 2 rounds

**Files:**
- Modify: `skills/brain-jam/SKILL.md`

- [ ] **Step 1: Insert Step 4.5 after Step 4 (Build seed)**

Add this section between Step 4 and the current Step 5:

```markdown
---

## Step 4.5: Resolve cast config

Probe API keys and pick the Chorus recipe. Run via Bash:

```bash
if [ -n "$GEMINI_API_KEY" ] && [ -n "$MINIMAX_API_KEY" ]; then
  echo "skills/brain-jam/recipes/grfp-readme.json"
elif [ -n "$MINIMAX_API_KEY" ]; then
  echo "skills/brain-jam/recipes/grfp-readme.fallback-minimax.json"
elif [ -n "$GEMINI_API_KEY" ]; then
  echo "skills/brain-jam/recipes/grfp-readme.fallback-gemini.json"
else
  echo "NO_KEYS"
fi
```

If the output is `NO_KEYS`, halt with:

> Need at least one API key: set GEMINI_API_KEY and/or MINIMAX_API_KEY (both preferred for cross-provider cast), then retry /brain-jam.

Store the output path as `CAST_CONFIG`.

If the path contains `fallback-minimax`, tell the operator before Step 5:

> Only MINIMAX_API_KEY found — running MiniMax for all voices (Gemini synth unavailable).

If the path contains `fallback-gemini`, tell the operator:

> Only GEMINI_API_KEY found — running Gemini for all voices (MiniMax pragmatist/critic unavailable).

Record whether a fallback was used (`FALLBACK=none|minimax|gemini`) for Step 6.3.

---
```

- [ ] **Step 2: Update Step 5 (Run Chorus CLI)**

Change the bash block to use `$CAST_CONFIG` and `--max-rounds 2`:

```bash
"$CHORUS" \
  --config "$CAST_CONFIG" \
  --prompt "What's the right angle for this README — tone, hook, and positioning?" \
  --seed "<seed from Step 4>" \
  --critique \
  --critic-temperature 0.3 \
  --max-rounds 2 \
  --argdown-mode lightweight \
  --output .claude/grfp/brainstorm-transcript-<YYYYMMDDTHHMMSS>.json
```

Renumber is not required if Step 4.5 is inserted and Step 5 content is updated in place.

- [ ] **Step 3: Add fallback footnote to Step 6.3**

After the Block 1 Set List template opening, add:

```markdown
If Step 4.5 used a fallback cast (`FALLBACK` is `minimax` or `gemini`), prepend this line before `## Set List`:

> *Cast fallback: MiniMax-only (GEMINI_API_KEY not set).* — or — *Cast fallback: Gemini-only (MINIMAX_API_KEY not set).*
```

- [ ] **Step 4: Verify no stale references**

Run:

```bash
grep -n 'max-rounds 3\|grfp-readme.json' skills/brain-jam/SKILL.md
```

Expected: `grfp-readme.json` appears only in Step 4.5 selection logic, not as a hardcoded Step 5 path. No `max-rounds 3`.

---

### Task 4: README and command docs

**Files:**
- Modify: `README.md`
- Modify: `commands/brain-jam.md`

- [ ] **Step 1: Update README Quick Start keys (around line 46)**

Replace the single-key export with:

```bash
export GEMINI_API_KEY=your-gemini-key-here   # synth (preferred)
export MINIMAX_API_KEY=your-minimax-key-here # pragmatist + critic (preferred)
# Either key alone works — brain-jam falls back to a single-provider cast
```

- [ ] **Step 2: Update Step 3 paragraph (around line 55)**

Replace the MINIMAX-only halt sentence with:

> Stage 4 needs the Chorus symlink, Deno, and **at least one** of `GEMINI_API_KEY` or `MINIMAX_API_KEY`. Both keys give a cross-provider cast (Gemini synth + MiniMax pragmatist/critic); a single key falls back to that provider for all voices.

- [ ] **Step 3: Update mermaid annotation (line 72)**

Change:

```
C -.- C1["Voice: Chorus synth + pragmatist + critic"]
```

to:

```
C -.- C1["Voice: Gemini synth + MiniMax pragmatist/critic (2 rounds)"]
```

- [ ] **Step 4: Update critic section intro (line 81)**

Change "Claude-synth and a MiniMax-pragmatist" to "Gemini synth and a MiniMax pragmatist". Add: "Two dialogue rounds by default."

- [ ] **Step 5: Update Requirements section (around line 210)**

Replace MINIMAX-only requirement with:

```markdown
- At least one of `GEMINI_API_KEY` or `MINIMAX_API_KEY` in environment or `~/.claude/skills/chorus/.env` (both preferred)
```

- [ ] **Step 6: Add one line to commands/brain-jam.md**

After the description in the body, add:

```markdown
Default cast: Gemini synth + MiniMax pragmatist/critic (2 rounds). Falls back to a single-provider cast when only one API key is set.
```

---

### Task 5: CHANGELOG

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Add 3.1.0 section at top (below header, above 3.0.1)**

```markdown
## [3.1.0] - 2026-06-07

### Changed

- **brain-jam cast**: Default synth voice is now Gemini 3.5 Flash (`gemini-3.5-flash`); pragmatist and critic remain MiniMax-M3.
- **brain-jam rounds**: Reduced from 3 to 2 (`--max-rounds 2`).
- **Provider fallback**: Brain-jam runs with either `GEMINI_API_KEY` or `MINIMAX_API_KEY`; missing provider roles fall back to the available model. Both keys preferred for cross-provider diversity.
- New recipes: `grfp-readme.fallback-minimax.json`, `grfp-readme.fallback-gemini.json`.

Design spec: `docs/snowball/specs/2026-06-07-gemini-synth-cast-fallback-design.md`
Implementation plan: `docs/snowball/plans/2026-06-07-gemini-synth-cast-fallback.md`

---
```

---

### Task 6: Manual verification

- [ ] **Step 1: Recipe parse via Chorus (no network if keys missing)**

If Chorus is installed:

```bash
CHORUS="$HOME/.claude/skills/chorus/bin/chorus"
for f in skills/brain-jam/recipes/grfp-readme*.json; do
  echo "=== $f ==="
  deno run --allow-read "$HOME/.claude/skills/chorus/src/run_spec.ts" 2>/dev/null || true
done
python3 -m json.tool skills/brain-jam/recipes/grfp-readme.json > /dev/null && echo "all recipes valid JSON"
```

- [ ] **Step 2: Key selection logic dry run**

```bash
GEMINI_API_KEY=x MINIMAX_API_KEY=y bash -c '
if [ -n "$GEMINI_API_KEY" ] && [ -n "$MINIMAX_API_KEY" ]; then echo primary
elif [ -n "$MINIMAX_API_KEY" ]; then echo minimax-fallback
elif [ -n "$GEMINI_API_KEY" ]; then echo gemini-fallback
else echo NO_KEYS; fi'
```

Expected: `primary`

```bash
unset GEMINI_API_KEY; MINIMAX_API_KEY=y bash -c '...'
```

Expected: `minimax-fallback`

- [ ] **Step 3: Live end-to-end (when keys available)**

Run `/brain-jam` with both keys. Confirm stderr shows `google/gemini-3.5-flash` for synth in round 1, exactly 2 critic rounds in transcript, and 4-block `brain-jam.md`.

- [ ] **Step 4: Commit**

```bash
git add skills/brain-jam/recipes/ skills/brain-jam/SKILL.md README.md CHANGELOG.md commands/brain-jam.md docs/snowball/specs/2026-06-07-gemini-synth-cast-fallback-design.md docs/snowball/plans/2026-06-07-gemini-synth-cast-fallback.md
git commit -m "$(cat <<'EOF'
feat(brain-jam): Gemini synth cast with provider fallback

Cross-provider default (Gemini synth + MiniMax prag/critic), 2-round
dialogue, and single-key fallback recipes when only one API key is set.
EOF
)"
```
