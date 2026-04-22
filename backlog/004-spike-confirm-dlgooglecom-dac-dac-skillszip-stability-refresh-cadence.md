---
schema_version: "1"
uuid: 324ad966-d4f7-4e6c-8ace-9428845d1b16
title: "Spike: confirm dl.google.com/dac/dac_skills.zip stability & refresh cadence"
status: backlog
priority: medium
type: spike
tags: [android-dev-extras-registry, research]
parent: 1
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

Two downstream decisions depend on this answer: (a) whether to migrate HOW-TO-SYNC's primary source to dac in the long run, and (b) whether to vendor `android-cli` (which embeds CLI-version-coupled content, making refresh cadence load-bearing for the decision).

## Research context

`~/.android/cli/skills/` on the author's machine shows an `update-skills.yml` workflow that downloads `https://dl.google.com/dac/dac_skills.zip` as the authoritative source. What the research could NOT determine:

- Is that URL contractually stable, or is it an internal publishing artifact subject to change?
- What is the refresh cadence — is it tied to CLI releases, a fixed schedule, or ad hoc?
- Are there notifications (RSS, changelog page, release tag) when the bundle updates?

See `research/android-dev-extras-registry/research.md` Open Questions and Feasibility table.

## What the spike might explore

- Observe the `ETag` / `Last-Modified` on `dl.google.com/dac/dac_skills.zip` over multiple CLI releases; correlate with Android CLI version changes.
- Search for public statements from Google (blog comments, dev relations, android-developers forum) about the URL's intended stability.
- File a GitHub issue on `github.com/android/skills` asking the maintainers directly (the README invites issue-based contact).
- If no stability signal emerges, define a fallback policy (stay on GitHub-only; pin a dac URL snapshot; set a sentinel file).

## Deliverables

A short findings note (can live alongside research/, or become a HOW-TO-SYNC appendix) summarizing: is dac-primary safe to adopt, or must we hedge? This unblocks ticket #6 and potentially reshapes #3.
