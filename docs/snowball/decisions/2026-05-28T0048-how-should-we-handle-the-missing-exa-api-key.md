---
title: How should we handle the missing EXA_API_KEY
status: accepted
date: '2026-05-28T00:48:07.452Z'
deciders:
  - kellen
snowball:
  schema_version: '1.0'
  source: operator
  confidence: high
  capture_mechanism: ask-user-question
  session_id: 4fa5710a-d464-4be6-8bd4-54aa578a8899
  source_event_id: toolu_01HRpDMiooVkjDuWrqTkzw91
  supersedes: null
  tags:
    - ambient
---

# How should we handle the missing EXA_API_KEY

## Context and Problem Statement

Question category: EXA key.

## Considered Options

- **Set EXA_API_KEY=dummy and continue** — Brainstorm doesn't actually call EXA; the validator just demands the env var exist. Setting a non-empty value satisfies validation and unblocks this run. Also a real upstream feedback item ('brainstorm should not validate research-mode keys').
- **Use a real EXA_API_KEY** — If you have one, paste it (I'll only set it for the single CLI invocation, not persist). Best if you ever plan to use m2-research.
- **Halt here, file upstream issue** — Per the v2.1.0 spec Layer 1 contract, stop and surface upstream's error verbatim. Continue the README work without the critic-voice dogfood demo — we already have deep-dive + crystal-ball.

## Decision Outcome

Chose **Set EXA_API_KEY=dummy and continue**. Brainstorm doesn't actually call EXA; the validator just demands the env var exist. Setting a non-empty value satisfies validation and unblocks this run. Also a real upstream feedback item ('brainstorm should not validate research-mode keys').
