---
schema_version: "1"
uuid: ee39e721-6736-4d53-a22e-0ecc3a284e78
title: "Update HOW-TO-SYNC for dual upstream (dac + github) with refresh check-step"
status: backlog
priority: high
type: chore
tags: [android-dev-extras-registry, docs]
parent: 1
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

The current `HOW-TO-SYNC.md` documents only `github.com/android/skills` as upstream. That's why the author-curator missed `android-cli` entirely — it's a dac-only skill. Any future dac-only additions will be missed the same way until the doc reflects the actual two-tier registry topology.

## Research context

`~/.android/cli/skills/.github/workflows/update-skills.yml` shows the source of truth is `dl.google.com/dac/dac_skills.zip`. The GitHub repo is a public overlay merged on top. `developer.android.com/tools/agents/android-skills/browse` is a further-curated user-facing subset (6 of 7).

See `research/android-dev-extras-registry/research.md` DR-2 (registry documentation) and the registry-topology findings in the Web & Documentation section.

## What "done" might look like

An updated HOW-TO-SYNC that:

- Names both upstreams and explains the relationship (dac = authoritative; github = public convenience mirror; developer.android.com browse = user-facing curated subset).
- Adds a "refresh check-step" to the procedure: on each refresh, run `android skills list`, diff against the union of HOW-TO-SYNC's covered + deferred lists, surface any skill the curator hasn't made a decision on.
- Flags that dac URL stability is not contractually guaranteed (see sibling spike ticket) — use GitHub as the working pull source today; revisit if dac becomes the primary after confirmation.

## Out of scope

- Implementing the drift-detection script itself (sibling ticket).
- Making a vendoring decision on `android-cli` (sibling ticket; depends on spike).
- Changing the Apache 2.0 attribution mechanics in HOW-TO-SYNC.
