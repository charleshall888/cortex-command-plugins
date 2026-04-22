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

Three child tickets covering the staged rollout from DR-1:

1. **Synthesis stage rewrite with three-axis rubric and Conventional Comments output format.** New `references/rubric.md` and `references/output-format.md` plus Stage 4 rewrite. Ship-gated on a rubric stability test (≥ 90% label consistency across 3 PRs × 3 runs).
2. **Renderer, pending-review posting, and diagram decision rule.** Converts synthesizer output to `gh api POST /pulls/{n}/reviews` with `event=PENDING` default; paste-ready fallback when posting fails; diagram triggers in the renderer prompt.
3. **Voice post-filter.** Deterministic em-dash strip + targeted sentence regeneration on AI-tell vocabulary. Calibrated to ≤ 5% false-positive rate on prior human-written comments.

Haiku early-exit gate (from DR-1 Stage 1 in the first decomposition) was dropped in consolidation review — the optimization's value doesn't justify its own ticket. Rubric + synthesizer combined into one ticket because the rubric has no standalone validation path without the synthesizer that applies it.

## Research context

Full research artifact: `research/pr-review-skill-improvements/research.md`. Key decisions:

- **DR-1**: Full redesign, staged rollout, with Stage 1 ship-gated on rubric stability (≥ 90% label consistency across 3 runs on 3 real PRs).
- **DR-2**: Collapse Anthropic's per-finding validator fan-out into the Opus 4.7 synthesizer. Single filter, single audit trail, no N-subagent cost.
- **DR-3**: Threshold-gate Architectural Assessment and Consensus Positives via new `cross-cutting:` and `praise:` finding types — don't cut them entirely; the user complained about noise, not absence.
- **DR-4**: Voice enforcement via deterministic post-filter + targeted sentence regeneration, not prompt-only.
- **DR-5**: Default posting path is `gh api` with `event=PENDING` (pending review, author-only, editable in GitHub UI). Paste-ready is the fallback when posting fails.

## Out of scope (future follow-ups)

Deferred items — worth tracking but not on this epic's critical path.

- Example-based voice transfer using a corpus of user-written review comments (tracked as a Stage 4 enhancement once corpus exists).
- CI-mode invocation with rate-limiting concerns (deferred unless confirmed).
- Preserving the `APPROVE | REQUEST CHANGES | REJECT` verdict keyword for back-compat with external scripts (confirm whether any exist before shipping).

### Surfaced by commercial-tool research (Bugbot, CodeRabbit, Ellipsis, Cloudflare, Greptile, Sourcery)

- **Risk-tiered agent count** (Cloudflare). Scale critic count by diff size: 2 agents for trivial, 4 for lite, full set for large or security-sensitive PRs. Current pipeline runs 4 regardless.
- **Upstream diff pre-filter** (Cloudflare). Strip lock files, vendored deps, minified assets, `@generated` files before they reach any critic. Current Stage 2 Haiku classifies them as `skim-ignore` but still ships the full diff to critics.
- **Named "explicitly out-of-scope" block in each critic's prompt** (Cloudflare). Do-not-flag list surfaced as a visible prompt section per critic, not buried in protocol.
- **Learned-rules feedback loop** (Bugbot, Greptile). Ingest user reactions on merged reviews to tune subsequent rubric weights. Requires persistence model; out of scope for initial redesign.
- **Agentic context-pulling** (Bugbot V2). Allow the bug/logic critic to request additional file reads within a budget, instead of a fixed context bundle. Big unpredictability shift; consider later.
- **Circuit-breaker model failback** (Cloudflare). On 429/503 from Opus 4.7, fail back to Sonnet instead of bombing the whole review.
- **Resolution-rate metric as ship-gate** (Bugbot). Post-review audit: of findings surfaced, how many led to changes in the merged diff? Richer than the current label-consistency ship gate.
- **Coordinator-level model config** (Cloudflare). Externalize model pins (Opus/Sonnet/Haiku slots) to a config file so swaps don't require editing protocol.md.
