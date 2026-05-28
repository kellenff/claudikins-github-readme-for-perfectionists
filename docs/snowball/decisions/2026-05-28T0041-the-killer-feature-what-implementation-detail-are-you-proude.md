---
title: 'The ''Killer'' Feature: what implementation detail are you proudest of in this plugin'
status: accepted
date: '2026-05-28T00:41:33.502Z'
deciders:
  - kellen
snowball:
  schema_version: '1.0'
  source: operator
  confidence: high
  capture_mechanism: ask-user-question
  session_id: 4fa5710a-d464-4be6-8bd4-54aa578a8899
  source_event_id: toolu_012XKEGtdqNJfQEPTmS6uX2L
  supersedes: null
  tags:
    - ambient
---

# The 'Killer' Feature: what implementation detail are you proudest of in this plugin

## Context and Problem Statement

Question category: Killer feature.

## Considered Options

- **The Anti-Slop banned-words list** — 15 specific filler words (delve, seamless, unleash, …) the pipeline refuses to ship. Lexical opinion as a feature.
- **The 5-stage pipeline structure** — Each stage produces a discrete artifact; later stages consume earlier ones. No skip-ahead allowed by the orchestrator.
- **The new critic third voice (v2.1.0)** — argdown-backed critique catches assumptions and weak claims the writer can't see. The signal a hostile reader would attack first.
- **The opinionated tone enforcement** — Sentence patterns (Hook / Hammer / Trust Builder) + humanisation rules + Flesch-Kincaid targets. Style baked in, not optional.

## Decision Outcome

Chose **The new critic third voice (v2.1.0)**. argdown-backed critique catches assumptions and weak claims the writer can't see. The signal a hostile reader would attack first.
