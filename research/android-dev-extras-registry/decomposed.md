# Decomposition: android-dev-extras-registry

## Epic
- **Backlog ID**: 1
- **Title**: Epic: android-dev-extras plugin registry adoption & design intent

## Work Items
| ID | Title | Priority | Size | Depends On |
|----|-------|----------|------|------------|
| 2  | Document plugin design intent + curation bar | high | S | — |
| 3  | Update HOW-TO-SYNC for dual upstream (dac + github) with refresh check-step | high | S | — |
| 4  | Spike: confirm `dl.google.com/dac/dac_skills.zip` stability & refresh cadence | medium | S | — |
| 5  | Add drift-detection script (`android skills list` vs HOW-TO-SYNC coverage) | medium | S | — |
| 6  | `android-cli` curation decision + conditional vendoring | medium | M | 4 |
| 7  | Codify external-consumer + breaking-change response policies in HOW-TO-SYNC | medium | S | — |

## Suggested Implementation Order
1. `#2` and `#3` in parallel — both are small doc changes with high value; they're the foundation everything else references.
2. `#4` (spike) next — it's the sole blocker on `#6` and is independent from other work.
3. `#5` any time after `#3` is in (the drift script encodes `#3`'s covered/deferred structure).
4. `#7` after `#2` so voice and vocabulary are consistent across the combined policy doc.
5. `#6` last — it depends on `#4`, benefits from `#3` being in place, and is the culmination of the discovery's core question.

## Key Design Decisions

### Consolidation
Two consolidation merges applied in §3:
- **Design intent + skill-bar criteria → one ticket (`#2`).** Same-file overlap: both live in the plugin-level README (or a HOW-TO-SYNC preamble). The skill-bar criteria are the concrete side of the abstract motivation; splitting them across tickets created artificial per-paragraph tickets without standalone value.
- **External-consumer policy + breaking-change policy → one ticket (`#7`).** Both extend HOW-TO-SYNC's policy surface and face the same reviewer simultaneously. Treating them as one ticket lets the policy be drafted coherently rather than stitched together across sessions.

### Decision not to auto-decide `android-cli`
The research recommends option F (vendor with detect-then-load guard) but explicitly conditions it on spike `#4`. The backlog ticket `#6` therefore captures the *decision + conditional execution* as one unit, not pre-baked. The `blocked-by: [4]` relationship enforces this.

### Scope of DR-5 deferrals (`navigation-3`, `migrate-xml-views-to-jetpack-compose`)
The discovery explicitly chose not to re-litigate these already-deferred skills. No tickets generated for them. If a future project adopts Nav 3 or needs large-scale Compose migration, reconsider per-project.

## Created Files
- `backlog/001-epic-android-dev-extras-plugin-registry-adoption-design-intent.md` — Epic
- `backlog/002-document-plugin-design-intent-curation-bar.md`
- `backlog/003-update-how-to-sync-for-dual-upstream-dac-github-with-refresh-check-step.md`
- `backlog/004-spike-confirm-dlgooglecom-dac-dac-skillszip-stability-refresh-cadence.md`
- `backlog/005-add-drift-detection-script-android-skills-list-vs-how-to-sync-coverage.md`
- `backlog/006-android-cli-curation-decision-conditional-vendoring.md`
- `backlog/007-codify-external-consumer-breaking-change-response-policies-in-how-to-sync.md`
