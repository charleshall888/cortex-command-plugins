---
schema_version: "1"
uuid: c6ebe8a1-8a5a-489e-9237-fbf28ae57c4e
title: "Rewrite pr-review synthesis stage: Opus 4.7, integrated precision + rubric, observability footer"
status: ready
priority: high
type: feature
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
parent: 3
blocked-by: [4]
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

Core of the filtering fix. Replaces the current Stage 4 "Opus synthesis" — which emits four named sections including noisy Architectural Assessment and Consensus Positives, and applies no rubric — with a single-pass Opus 4.7 synthesizer that does evidence-grounding validation, rubric scoring, and filtering together, emitting only findings that clear the gate. Adds an observability footer so users can see what was dropped and why. Directly addresses user complaints (1) and (3) — "already filter before presenting" and "sell the value of each comment".

## Research context

See `research/pr-review-skill-improvements/research.md` — Question 1 (Anthropic's precision pass pattern, our collapse of it), Question 2 (Opus 4.7 leverage points), DR-2 (single-pass synthesizer decision), DR-3 (`cross-cutting:` and `praise:` as rubric-gated outputs).

## What this ticket delivers

Rewrites Stage 4 of `plugins/cortex-pr-review/skills/pr-review/references/protocol.md`:

- Synthesizer subagent pinned to `model: claude-opus-4-7`.
- Receives full diff, all CLAUDE.mds, and all four critic outputs in one context (1M window).
- Applies evidence-grounding check first — each finding must quote offending lines; unanchored findings are dropped.
- Applies the rubric from `references/rubric.md` (ticket 004) to score each surviving finding.
- Emits output as a structured list of labeled findings per Conventional Comments format — no fixed Architectural Assessment / Consensus Positives sections.
- Emits observability footer: `N considered, X surfaced, Y dropped (Y1 unanchored, Y2 low-signal, Y3 linter-class, Y4 over-cap)`.
- Removes scaffolding language (e.g. "double-check before returning") that Opus 4.7's more literal instruction-following makes unnecessary.
- Updated failure handling: if synthesizer fails, present raw Sonnet outputs as before (existing fallback preserved).

Also updates `SKILL.md` if the description needs to reflect the new behavior (no structural changes to front-matter).

## Out of scope

- Posting findings to GitHub — ticket 006.
- Diagram output — ticket 006.
- Voice post-filter — ticket 007.
- Haiku early-exit gate — ticket 008.
