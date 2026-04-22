---
schema_version: "1"
uuid: 67ddd890-a17c-4070-91a8-f21a389981d9
title: "Add pr-review Haiku early-exit gate for drafts, closed PRs, and already-reviewed PRs"
status: ready
priority: low
type: feature
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
parent: 3
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

Runs before the triage stage. Cheap Haiku call that checks PR state (draft / closed / merged / already-reviewed by the user) and short-circuits the full pipeline when review is unnecessary or would be wasted effort. Matches Anthropic's own `code-review` plugin pattern (Step 1 in their protocol). Small optimization, independent of the filtering/posting rework — runnable in parallel with tickets 004-007.

## Research context

See `research/pr-review-skill-improvements/research.md` — Web & Documentation Research, Anthropic's canonical multi-agent review template: *"Step 1 (Haiku): early-exit gate — skip drafts, closed PRs, trivial changes, PRs already reviewed. Our current skill doesn't gate; we add it."*

## What this ticket delivers

New Stage 1.5 in `plugins/cortex-pr-review/skills/pr-review/references/protocol.md` between metadata fetch and triage:

- Haiku subagent receives PR metadata (state, latestReviews, author).
- Returns `exit: skip | proceed` with a one-sentence reason.
- Early-exit conditions:
  - PR is a draft → skip (unless user explicitly requested review of a draft).
  - PR is closed or merged → skip with note.
  - User has already reviewed this PR at its current head SHA → skip with note.
  - Diff is trivial (e.g., pure whitespace, dependency bump, single-line version change) → skip with note.
- On skip, main agent presents the Haiku reason verbatim and stops. No Stage 2+ execution.
- On proceed, pipeline continues as normal.

## Out of scope

- Configurable skip rules beyond the defaults above.
- Skipping based on file-type triage (that's handled by the existing Stage 2 Haiku triage, which this does not replace).
