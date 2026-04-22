# Decomposition: pr-review-skill-improvements

## Epic
- **Backlog ID**: 003
- **Title**: Redesign pr-review skill: adversarial filtering, Conventional Comments output, pending-review posting

## Work Items

| ID | Title | Priority | Size | Depends On |
|----|-------|----------|------|------------|
| 004 | Add pr-review three-axis rubric and output-format references | high | M | — |
| 005 | Rewrite pr-review synthesis stage: Opus 4.7, integrated precision + rubric, observability footer | high | M | 004 |
| 006 | Add pr-review renderer, pending-review posting via gh api, and diagram decision rule | high | M | 005 |
| 007 | Add pr-review voice post-filter: em-dash strip and AI-tell sentence regeneration | medium | S | 006 |
| 008 | Add pr-review Haiku early-exit gate for drafts, closed PRs, and already-reviewed PRs | low | S | — |

## Suggested Implementation Order

Linear chain for the core redesign: 004 → 005 → 006 → 007. Each stage is independently testable and delivers incremental value — 004 ships rubric documentation that can be calibrated before the synthesizer adopts it; 005 gets the filtering improvement visible in terminal output; 006 adds the pending-review posting workflow; 007 applies voice discipline to the posted text.

Ticket 008 (Haiku early-exit gate) has no dependencies on the core chain and can run in parallel any time.

## Key Design Decisions

From research DR-1 through DR-5:

- **Single-pass synthesizer, not per-finding validator subagents** (DR-2). Adversarial review of the research identified that Anthropic's per-finding validator fan-out has strictly less context than the critic that produced the finding, creating a failure mode where the most valuable (history-dependent) findings get dropped. Collapsed into one Opus 4.7 synthesizer with full 1M context.
- **Threshold-gate, don't cut, Architectural Assessment and Consensus Positives** (DR-3). User complaint was "noise, not absence." New first-class finding types `cross-cutting:` and `praise:` surface only when rubric-gated.
- **Default posting path is `event=PENDING`, not paste-ready** (DR-5). Pending reviews are private to the author, editable in GitHub UI, one-click submittable — strictly safer than paste-ready, and the skill's structured output already produces the needed payload. Paste-ready becomes the fallback for posting failures.
- **Voice enforcement via deterministic post-filter + targeted regen, not prompt-only** (DR-4). Prompt-only has already been observed to leak.
- **Three-axis rubric with coarse buckets, not five 1-5 integer scales** (DR-1 Stage 1 / Question 4). Adversarial review caught that five independent 1-5 axes with hard thresholds replicate the fragility they were meant to fix. Narrowed to severity (4 buckets) / solidness (3 buckets) / signal (3 buckets) with an explicit `unknown` path that routes to `question:` findings.

## Consolidation Notes

- Diagram decision rule (DR-1 Stage 5) merged into ticket 006 (Stage 3 renderer) — the rule is one prompt addition to the renderer, no standalone deliverable value. Rationale: (b) no-standalone-value prerequisite.
- No other consolidations — each remaining ticket maps to a distinct artifact and a discrete testable increment.

## Created Files

- `backlog/003-redesign-pr-review-skill-adversarial-filtered-posted.md` — Epic
- `backlog/004-pr-review-rubric-and-output-format-references.md`
- `backlog/005-pr-review-single-pass-opus-47-synthesizer.md`
- `backlog/006-pr-review-renderer-pending-review-posting-diagram-rule.md`
- `backlog/007-pr-review-voice-post-filter.md`
- `backlog/008-pr-review-haiku-early-exit-gate.md`
