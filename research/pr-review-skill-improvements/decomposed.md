# Decomposition: pr-review-skill-improvements

## Epic
- **Backlog ID**: 003
- **Title**: Redesign pr-review skill: adversarial filtering, Conventional Comments output, pending-review posting

## Work Items

| ID | Title | Priority | Size | Depends On |
|----|-------|----------|------|------------|
| 004 | Rewrite pr-review synthesis stage with three-axis rubric and Conventional Comments output format | high | M-L | — |
| 005 | Add pr-review renderer, pending-review posting via gh api, and diagram decision rule | high | M | 004 |
| 006 | Add pr-review voice post-filter: em-dash strip and AI-tell sentence regeneration | medium | S | 005 |

## Suggested Implementation Order

Linear chain: 004 → 005 → 006. Each stage is independently testable and delivers incremental user-visible value. 004 gets filtering into terminal output and addresses complaints (1), (3), (4). 005 adds the pending-review posting workflow and addresses (2) and (5). 006 applies voice discipline to posted text and addresses (6).

## Key Design Decisions

From research DR-1 through DR-5:

- **Single-pass synthesizer, not per-finding validator subagents** (DR-2). Adversarial review of the research identified that Anthropic's per-finding validator fan-out has strictly less context than the critic that produced the finding, creating a failure mode where the most valuable (history-dependent) findings get dropped. Collapsed into one Opus 4.7 synthesizer with full 1M context.
- **Threshold-gate, don't cut, Architectural Assessment and Consensus Positives** (DR-3). User complaint was "noise, not absence." New first-class finding types `cross-cutting:` and `praise:` surface only when rubric-gated.
- **Default posting path is `event=PENDING`, not paste-ready** (DR-5). Pending reviews are private to the author, editable in GitHub UI, one-click submittable — strictly safer than paste-ready, and the skill's structured output already produces the needed payload. Paste-ready becomes the fallback for posting failures.
- **Voice enforcement via deterministic post-filter + targeted regen, not prompt-only** (DR-4). Prompt-only has already been observed to leak.
- **Three-axis rubric with coarse buckets, not five 1-5 integer scales** (DR-1 Stage 1 / Question 4). Adversarial review caught that five independent 1-5 axes with hard thresholds replicate the fragility they were meant to fix. Narrowed to severity (4 buckets) / solidness (3 buckets) / signal (3 buckets) with an explicit `unknown` path that routes to `question:` findings.

## Consolidation Notes

- **Rubric + synthesizer merged into ticket 004.** The rubric had no standalone validation path — its ship gate (≥ 90% label consistency across 3 PRs × 3 runs) requires the synthesizer to exist. Separating them was a decomposition cheat. Combined into one M-L ticket per signal (b) no-standalone-value prerequisite.
- **Haiku early-exit gate dropped.** Optimization independent of the filtering/posting rework; user confirmed the value didn't justify a ticket.
- **Diagram decision rule merged into ticket 005.** The rule is one prompt addition to the renderer, no standalone deliverable value. Rationale: (b) no-standalone-value prerequisite.
- **Voice filter (006) kept separate from renderer (005).** Voice filter is small and self-contained: one new reference file plus one pass stage. Bundling would have bloated 005 without simplifying. Clean seam at "rendered text → filtered text → posted review."

## Created Files

- `backlog/003-redesign-pr-review-skill-adversarial-filtered-posted.md` — Epic
- `backlog/004-pr-review-synthesizer-with-rubric-and-output-format.md`
- `backlog/005-pr-review-renderer-pending-review-posting-diagram-rule.md`
- `backlog/006-pr-review-voice-post-filter.md`
