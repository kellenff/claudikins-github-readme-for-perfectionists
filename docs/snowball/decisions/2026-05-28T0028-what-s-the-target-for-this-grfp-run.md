---
title: What's the target for this GRFP run
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

# What's the target for this GRFP run

## Context and Problem Statement

Question category: Target.

## Considered Options

- **Refresh this repo's README** — Update the existing README.md to reflect v2.0.0 (MiniMax engine) + v2.1.0 (critic voice) and any other drift since May. Most natural use of just-shipped features.
- **Test the new brain-jam wiring end-to-end** — Run the pipeline as a dogfooding exercise on this repo, focused on whether brain-jam.md renders the four blocks correctly. Same target, different intent.
- **Different repo entirely** — Point GRFP at another project. I'll need the path.

## Decision Outcome

Chose **Refresh this repo's README**. Update the existing README.md to reflect v2.0.0 (MiniMax engine) + v2.1.0 (critic voice) and any other drift since May. Most natural use of just-shipped features.
