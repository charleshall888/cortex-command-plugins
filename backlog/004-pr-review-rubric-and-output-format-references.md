---
schema_version: "1"
uuid: a0bc2ea1-b65e-479c-86ac-eadf7136cb61
title: "Add pr-review three-axis rubric and output-format references"
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

Foundation for the entire pr-review redesign. Replaces the current "2+ agents flagged = high confidence" rule (which under-filters) with a locked three-axis rubric plus Conventional Comments taxonomy. Produces the scoring guide and label catalog that the synthesizer (next ticket) will apply. Ships independently as documentation — no pipeline code changes yet — which lets the rubric be calibrated against real PRs before the synthesizer adopts it.

## Research context

See `research/pr-review-skill-improvements/research.md` — Question 4 answer (rubric definition), Question 7 answer (`cross-cutting:` and `praise:` as first-class finding types), Web & Documentation Research (Conventional Comments taxonomy).

## What this ticket delivers

Two new reference files inside `plugins/cortex-pr-review/skills/pr-review/references/`:

- **`rubric.md`** — three-axis scoring rubric:
  - Severity: `must-fix` / `should-fix` / `nit` / `unknown`.
  - Solidness: `solid` / `plausible` / `thin`.
  - Signal: `high` / `medium` / `low`.
  - Gate thresholds per finding type, with worked examples on real PR findings.
  - `cross-cutting:` and `praise:` finding types with their own gate rules.
  - Nit cap (3 per review) with deterministic tie-break (file path alphabetical, then line ascending).
  - Drop-reason taxonomy for the observability footer (`unanchored`, `low-signal`, `linter-class`, `over-cap`).

- **`output-format.md`** — Conventional Comments label taxonomy (`issue:`, `suggestion:`, `nit:`, `question:`, `praise:`, `cross-cutting:`, with `(blocking)`/`(non-blocking)` decorators), `suggestion` block syntax rules (including the 4+ backtick outer fence for nested code blocks), line-anchoring rules (`path`, `line`, `start_line`, `side`, when to use `LEFT` vs `RIGHT`), and the voice guide (no em-dashes, no AI-tell vocabulary, no validation openers or closing fluff).

Also includes a **rubric stability test procedure**: synthesizer run on 3 real PRs × 3 runs each; ship gate is ≥ 90% label consistency. PR selection documented in the reference (one recent merged PR with known nits, one with a real bug, one pure refactor).

## Out of scope

- Editing `SKILL.md` or `references/protocol.md` — those change in ticket 005.
- Any pipeline code changes. This ticket is reference material only.
- Voice post-filter regex list — lives in ticket 007.
