# Decomposition: android-dev-extras-registry

## Work Item
Single ticket — the work is a single documentation PR on a personal plugin.

- **Backlog ID**: 2
- **Title**: Refresh android-dev-extras HOW-TO-SYNC: dual upstream, design intent, android-cli decision
- **Priority**: high
- **Size**: S/M (one doc refresh + one skill vendoring if option F wins)
- **Dependencies**: none

## Honed framing

Discovery's real value is closing the two gaps it surfaced (missed upstream, unstated motivation) and making the concrete `android-cli` call in one pass. The first decomposition proposed an epic + 6 children. That was over-scoping for a personal plugin with 2 skills and 1 known consumer.

Dropped work items (with rationale for dropping):
- **Spike on `dl.google.com/dac/dac_skills.zip` stability** — decision can be made without it; note the caveat in HOW-TO-SYNC and revert if the URL ever breaks.
- **Drift-detection script** — over-engineering for 2 covered + 4 deferred skills; a manual `android skills list` check on each refresh is cheaper.
- **External-consumer + breaking-change policies** — speculative for a personal repo; revisit only if the plugin gains more consumers or upstream starts breaking things.
- **Separate design-intent-doc ticket** — folded into the HOW-TO-SYNC refresh since both land in the same file.
- **Separate HOW-TO-SYNC update ticket** — folded into the same PR as above.
- **Separate android-cli vendoring ticket** — folded into the same PR; the decision and the execution are a single coherent change.

## Suggested Implementation Order

`/lifecycle 2` when ready. Single PR.

## Key Design Decisions

### Epic removed
`decompose.md` says "Epic + children" for 2+ items. Collapsing to 1 item makes the epic unnecessary. The original epic (#1) and the five speculative children (#3–#7) were deleted from the backlog.

### Ticket #2 repurposed
The original #2 was narrowly about design-intent docs. Retitled and rescoped to be the single unified ticket. UUID preserved; title, body, and tags updated.

## Created Files
- `backlog/002-document-plugin-design-intent-curation-bar.md` — single consolidated ticket (slug retained from creation; title in frontmatter is authoritative)
