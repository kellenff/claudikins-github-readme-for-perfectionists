# Gemini Synth Cast + Provider Fallback

**Date:** 2026-06-07  
**Status:** Approved  
**Plugin:** `claudikins-grfp` (claudikins-github-readme-for-perfectionists)  
**Predecessor:** `2026-06-07-chorus-brain-jam-swap-design.md`

## Summary

Upgrade the GRFP brain-jam Chorus cast to use **Gemini 3.5 Flash** for the `synth` voice while keeping MiniMax for `pragmatist` and `critic`. Reduce dialogue from **3 rounds to 2**. Add **pre-flight provider fallback**: if only one API key is available, the missing provider's roles are absorbed by the available model.

Chorus stays generic; GRFP owns cast recipes and selection logic in `skills/brain-jam/SKILL.md`.

## Goals

1. Cross-provider diversity in the default cast (Gemini synth + MiniMax pragmatist/critic).
2. Brain-jam runs when the operator has **either** `GEMINI_API_KEY` or `MINIMAX_API_KEY` (both preferred).
3. Drop round count to 2 — round 1 explores; round 2 responds to critic pressure (proven value in chorus gemini dogfood; round 3 diminishing returns on the narrow README-angle prompt).
4. Participant names unchanged (`synth`, `pragmatist`, `critic`) so the renderer needs no structural changes.

## Non-Goals

- Mid-run API failure fallback (rate limits, model-not-found during dialogue). Requires Chorus engine changes.
- Modifying the Chorus repo.
- Adding a third speaker participant.
- Renaming `synth` to `gemini-synth` (keeps renderer stable; model identity is in the transcript stderr/progress stream).

## Cast recipes

Three JSON files under `skills/brain-jam/recipes/`. Personas and temperatures identical; only `model` fields differ.

### Primary — `grfp-readme.json`

| Participant | Model |
| --- | --- |
| `synth` | `gemini-3.5-flash` |
| `pragmatist` | `MiniMax-M3` |
| `critic` | `MiniMax-M3` |

Requires both `GEMINI_API_KEY` and `MINIMAX_API_KEY`.

### Fallback — `grfp-readme.fallback-minimax.json`

All three on `MiniMax-M3`. Used when `GEMINI_API_KEY` is unset.

### Fallback — `grfp-readme.fallback-gemini.json`

All three on `gemini-3.5-flash`. Used when `MINIMAX_API_KEY` is unset.

## Cast selection (Step 4.5)

Insert between seed building (Step 4) and Chorus invocation (Step 5) in `skills/brain-jam/SKILL.md`:

```bash
if [ -n "$GEMINI_API_KEY" ] && [ -n "$MINIMAX_API_KEY" ]; then
  CAST_CONFIG="skills/brain-jam/recipes/grfp-readme.json"
  CAST_NOTE=""
elif [ -n "$MINIMAX_API_KEY" ]; then
  CAST_CONFIG="skills/brain-jam/recipes/grfp-readme.fallback-minimax.json"
  CAST_NOTE="Only MINIMAX_API_KEY found — running MiniMax for all voices."
elif [ -n "$GEMINI_API_KEY" ]; then
  CAST_CONFIG="skills/brain-jam/recipes/grfp-readme.fallback-gemini.json"
  CAST_NOTE="Only GEMINI_API_KEY found — running Gemini for all voices."
else
  echo "NO_KEYS"
fi
```

If output is `NO_KEYS`, halt with:

> Need at least one API key: set GEMINI_API_KEY and/or MINIMAX_API_KEY (both preferred for cross-provider cast), then retry /brain-jam.

If `CAST_NOTE` is non-empty, tell the operator before running Chorus.

Chorus validates keys for the **selected** cast only — mixed cast is never attempted when a key is missing.

## Chorus invocation changes

| Flag | Was | New |
| --- | --- | --- |
| `--config` | `grfp-readme.json` (fixed) | `$CAST_CONFIG` from Step 4.5 |
| `--max-rounds` | `3` | `2` |

All other locked flags unchanged.

## Renderer impact

Minimal. Participant names unchanged.

- **Block 1 (Set List):** unchanged.
- **Block 2 (Watch-Outs):** at most 2 rounds per voice (was 3).
- **Block 4 (Argument Map):** uses last ok critic turn (now round 2 at latest).

**Optional footnote:** When a fallback cast was used, prepend to `brain-jam.md`:

```markdown
> *Cast fallback: <MiniMax-only | Gemini-only> (<missing key> not set).*
```

## Failure modes

| Mode | Detection | Behavior |
| --- | --- | --- |
| Neither key set | Step 4.5 | Halt with install hint |
| Only one key | Step 4.5 | Fallback recipe + operator notice |
| Chorus CLI error mid-run | Non-zero exit | Surface stderr verbatim (unchanged) |
| Critic unavailable | Status classification | FULL / PARTIAL / NO-CRITIQUE (unchanged) |

## File-level changes

| File | Change |
| --- | --- |
| `skills/brain-jam/recipes/grfp-readme.json` | `synth` model → `gemini-3.5-flash` |
| `skills/brain-jam/recipes/grfp-readme.fallback-minimax.json` | **New** — all MiniMax |
| `skills/brain-jam/recipes/grfp-readme.fallback-gemini.json` | **New** — all Gemini |
| `skills/brain-jam/SKILL.md` | Step 4.5 cast selection; `--max-rounds 2`; optional fallback footnote in Step 6 |
| `README.md` | API key requirements; pipeline diagram; 2-round note |
| `CHANGELOG.md` | Entry under next version |
| `commands/brain-jam.md` | Optional one-line cast note |

**Not changed:** `skills/grfp/SKILL.md`, `skills/pen-wielding/SKILL.md`, Chorus repo.

## Testing

Manual verification:

1. **Both keys:** `/brain-jam` uses primary cast; stderr shows `google/gemini-3.5-flash` for synth.
2. **MiniMax only:** fallback-minimax recipe; operator sees notice; run completes.
3. **Gemini only:** fallback-gemini recipe; operator sees notice; run completes.
4. **Neither key:** halt before Chorus invoked.
5. **Transcript shape:** 2 critic rounds, 4 speak turns (+ seed), 4-block `brain-jam.md`.

## Rationale: 2 rounds

- Round 1 + critic establishes baseline angles from a rich seed (deep-dive + crystal-ball + sound check).
- Round 2 + critic injects critic addendum (anti-steelman, undefended assumptions) — demonstrated new-axis value in chorus gemini dogfood.
- Round 3 often rephrases rather than discovers on the narrow README-angle prompt; saves ~33% API cost/latency.
