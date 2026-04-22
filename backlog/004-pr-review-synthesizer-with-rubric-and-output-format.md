---
schema_version: "1"
uuid: 9061e399-0236-45f5-ae55-e79af91dc9d4
title: "Rewrite pr-review synthesis stage with three-axis rubric and Conventional Comments output format"
status: ready
priority: high
type: feature
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
parent: 3
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

Foundation of the redesign. Replaces the current Stage 4 "Opus synthesis" — which emits four fixed sections (including the noisy Architectural Assessment and Consensus Positives) and applies no filtering rubric — with a single-pass Opus 4.7 synthesizer that does evidence-grounding validation, rubric scoring, and filtering together. Only findings that clear the gate are surfaced. Each surfaced finding carries a Conventional Comments label so its value is legible at a glance. An observability footer reports what was dropped and why.

Directly addresses user complaints (1) "skill should filter before presenting", (3) "sell the value of each comment", and (4) "noise from Architectural Assessment and Consensus Positives".

## Research context

See `research/pr-review-skill-improvements/research.md`:
- Question 1 (Anthropic's precision-pass pattern; our collapse of it into the synthesizer)
- Question 2 (Opus 4.7 leverage points; caveat on literal instruction following)
- Question 4 (three-axis rubric definition with `unknown` path)
- Question 7 (`cross-cutting:` and `praise:` as first-class rubric-gated finding types)
- DR-2 (single-pass synthesizer instead of per-finding validator fan-out)
- DR-3 (threshold-gate Architectural Assessment and Consensus Positives)

## What this ticket delivers

Two new reference files plus a rewrite of Stage 4 of the protocol — shipped together because the references define the behavior the synthesizer executes, and neither has standalone user-visible value without the other.

### New reference: `references/rubric.md`

- Three-axis scoring rubric:
  - Severity: `must-fix` / `should-fix` / `nit` / `unknown`.
  - Solidness: `solid` / `plausible` / `thin`.
  - Signal: `high` / `medium` / `low`.
- Gate thresholds per finding type with worked examples on real PR findings.
- `cross-cutting:` and `praise:` finding types with their own gate rules (bypass locality for cross-cutting; gated by `solidness=solid AND signal=high` for both).
- Nit cap (3 per review) with deterministic tie-break (file path alphabetical, then line ascending).
- Drop-reason taxonomy for the observability footer (`unanchored`, `low-signal`, `linter-class`, `over-cap`).

### New reference: `references/output-format.md`

Conventional Comments label taxonomy (`issue:`, `suggestion:`, `nit:`, `question:`, `praise:`, `cross-cutting:`) with `(blocking)`/`(non-blocking)` decorators. Voice guide (no em-dashes, no AI-tell vocabulary, no validation openers, no closing fluff). Note: suggestion-block syntax and line-anchoring rules belong in the renderer ticket since they're renderer concerns, not synthesizer concerns.

### Rewrite: Stage 4 of `references/protocol.md`

- Synthesizer subagent pinned to `model: claude-opus-4-7`.
- Receives full diff, all CLAUDE.mds, and all four critic outputs in one context (1M window).
- Applies evidence-grounding check first: each finding must quote offending lines; unanchored findings dropped.
- Applies the rubric to score each surviving finding against severity / solidness / signal.
- Emits output as a structured list of labeled findings per Conventional Comments format — no fixed Architectural Assessment / Consensus Positives sections.
- Emits observability footer: `N considered, X surfaced, Y dropped (Y1 unanchored, Y2 low-signal, Y3 linter-class, Y4 over-cap)`.
- Removes scaffolding language ("double-check before returning") that Opus 4.7's literal instruction-following makes unnecessary.
- Existing synthesis-failure fallback preserved (present raw Sonnet outputs with explanation).

### Ship gate: rubric stability test

Before declaring the ticket shipped, run the synthesizer on 3 real PRs × 3 runs each. Require ≥ 90% label consistency across runs. Selected PRs documented in `references/rubric.md`:
- One recent merged PR with known nits.
- One recent PR with a real bug.
- One pure refactor.

If the stability bar isn't met, tighten rubric wording and calibration examples before shipping — the rest of the epic (renderer, voice filter) inherits this stage's scoring behavior, so instability here compounds downstream.

## Out of scope

- Renderer, posting to GitHub, suggestion blocks, line anchoring — next ticket.
- Diagram output — next ticket.
- Voice post-filter — later ticket.
- Any corpus-based voice transfer using user-written comments — epic-level follow-up.
