---
title: How should I handle the missing m2-brainstorm binary
status: accepted
date: '2026-05-28T00:28:09.293Z'
deciders:
  - kellen
snowball:
  schema_version: '1.0'
  source: operator
  confidence: high
  capture_mechanism: ask-user-question
  session_id: 4fa5710a-d464-4be6-8bd4-54aa578a8899
  source_event_id: toolu_01HpadrT4CKs7xvwinqJJGA1
  supersedes: null
  tags:
    - ambient
---

# How should I handle the missing m2-brainstorm binary

## Context and Problem Statement

Question category: Binary.

## Considered Options

- **Install it now** — Run the m2-brainstorm install script before starting so Stage 4 won't halt. I'll show you what command will run before executing.
- **Start anyway** — Begin the pipeline; halt at Stage 4 and install then. Useful if you want to see Stages 1-3 first.
- **I'll install it separately** — You'll handle the install; I should proceed and assume the binary will exist by Stage 4.

## Decision Outcome

Chose **Install it now**. Run the m2-brainstorm install script before starting so Stage 4 won't halt. I'll show you what command will run before executing.
