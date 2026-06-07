# Web Research Reference (Primary)

**Plugin-wide reference for think-tank exemplar README research.**

This phase researches external repositories on the web. **No third-party search API is required.** The pipeline must complete using only built-in `WebSearch` and `WebFetch` tools.

---

## Tool Priority (use first match available)

| Priority | Tool | When to use |
| --- | --- | --- |
| 1 | `WebSearch` | Discover candidate repos, awesome lists, comparisons |
| 2 | `WebFetch` | Read raw README content from `https://raw.githubusercontent.com/...` |
| 3 | Exa MCP (`web_search_exa`, `web_fetch_exa`) | Optional enhancement if installed — never required |
| 4 | Gemini deep research via `claudikins-tool-executor` | Optional legacy path — see `gemini-deep-research.md` |

**If Exa is unavailable:** continue with `WebSearch` + `WebFetch`. Do not halt the pipeline.

**If all web tools fail:** use the candidate list below and score from cached knowledge, marking confidence as Low in the output.

---

## Standard workflow

### Step 1: Discover candidates

```
WebSearch: "best [project-type] github README examples"
WebSearch: "awesome [technology] documentation"
WebSearch: "[similar-tool] alternatives github stars"
```

Find 10-15 candidates, narrow to 5-7 by stars and initial quality scan.

### Step 2: Fetch READMEs directly

Prefer raw GitHub URLs — no API key, no JavaScript rendering:

```
https://raw.githubusercontent.com/{owner}/{repo}/{branch}/README.md
```

Use `WebFetch` on each exemplar. If fetch fails, try the repo's default branch or `main`/`master`.

### Step 3: Score and extract

Apply `scoring-rubric.md`. Extract patterns per `think-tank` Step 4.

---

## Known exemplar seeds (documentation / linter tools)

Useful starting points when search is slow or unavailable:

| Repo | Why study it |
| --- | --- |
| [astral-sh/ruff](https://github.com/astral-sh/ruff) | Proof-first hero, benchmark chart, testimonials with numbers |
| [get-alex/alex](https://github.com/get-alex/alex) | Opinionated prose linter, clear qualification signals |
| [prettier/prettier](https://github.com/prettier/prettier) | Mature OSS README structure |
| [eli64s/readme-ai](https://github.com/eli64s/readme-ai) | Direct competitor — anti-pattern source (vague emoji bullets) |

---

## Graceful degradation

```markdown
Detection:
- Exa MCP not in allowed-tools or calls fail → use WebSearch/WebFetch only
- WebSearch unavailable → fetch known exemplar URLs directly via WebFetch
- WebFetch blocked → score from prior knowledge, note "Low confidence — web tools unavailable"
```

Warn the user once if web tools are degraded. Do not hard-fail think-tank.

---

## What is NOT required

- `EXA_API_KEY` — never required by GRFP (removed with v3.0.0 Chorus swap)
- Exa Cursor plugin / MCP server
- `claudikins-tool-executor` Gemini deep research
- Any paid search API
