---
schema_version: "1"
uuid: 31eff7f8-2716-4898-a4d1-aa20ae9a69ef
title: "Redesign pr-review skill: adversarial filtering, Conventional Comments output, pending-review posting"
status: ready
priority: high
type: epic
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

The current `/pr-review` skill surfaces too many findings that don't survive a "is this worth commenting?" pass, outputs narrative prose the user can't paste into GitHub, doesn't sell the value of each comment, leaks AI-tell writing style, and has no visuals. Users re-prompt the agent to filter further, at which point it drops half the findings — meaning the skill should have filtered before presenting. This epic redesigns the pipeline around an adversarial, opinionated reviewer that filters its own output, produces ready-to-use GitHub review comments with code suggestions, defaults to posting as a pending review (not pasting), and writes in the user's voice.

## Scope

Five child tickets covering the staged rollout from DR-1:

1. Three-axis rubric + output-format references (`references/rubric.md`, `references/output-format.md`) with Conventional Comments taxonomy.
2. Single-pass Opus 4.7 synthesizer that collapses precision validation + rubric scoring + filtering into one stage with an observability footer.
3. Renderer + pending-review posting via `gh api POST /pulls/{n}/reviews` with `event=PENDING` default; paste-ready fallback; diagram decision rule.
4. Voice post-filter: deterministic em-dash strip + targeted sentence regeneration on AI-tell vocabulary.
5. Haiku early-exit gate: skip drafts, closed PRs, already-reviewed PRs before triage.

## Research context

Full research artifact: `research/pr-review-skill-improvements/research.md`. Key decisions:

- **DR-1**: Full redesign, staged rollout, with Stage 1 ship-gated on rubric stability (≥ 90% label consistency across 3 runs on 3 real PRs).
- **DR-2**: Collapse Anthropic's per-finding validator fan-out into the Opus 4.7 synthesizer. Single filter, single audit trail, no N-subagent cost.
- **DR-3**: Threshold-gate Architectural Assessment and Consensus Positives via new `cross-cutting:` and `praise:` finding types — don't cut them entirely; the user complained about noise, not absence.
- **DR-4**: Voice enforcement via deterministic post-filter + targeted sentence regeneration, not prompt-only.
- **DR-5**: Default posting path is `gh api` with `event=PENDING` (pending review, author-only, editable in GitHub UI). Paste-ready is the fallback when posting fails.

## Out of scope

- Example-based voice transfer using a corpus of user-written review comments (tracked as a Stage 4 enhancement once corpus exists).
- CI-mode invocation with rate-limiting concerns (deferred unless confirmed).
- Preserving the `APPROVE | REQUEST CHANGES | REJECT` verdict keyword for back-compat with external scripts (confirm whether any exist before shipping).
