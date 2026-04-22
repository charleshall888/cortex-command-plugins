---
schema_version: "1"
uuid: 9377a525-0177-4930-afc3-1902e7627b61
title: "Codify external-consumer + breaking-change response policies in HOW-TO-SYNC"
status: backlog
priority: medium
type: chore
tags: [android-dev-extras-registry, policy, docs]
parent: 1
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

Two policy gaps came out of critical review: (a) the marketplace is a public repo — anyone who ran `claude /plugin marketplace add` is an invisible consumer, and changes to `android-dev-extras` can silently break their workflows; (b) HOW-TO-SYNC has no documented response for the day upstream ships a breaking refactor of a vendored skill. Closing both gaps sets expectations for future maintainers and sets a contract with consumers.

## Research context

From `research/android-dev-extras-registry/research.md` Open Questions (final two bullets) and the critical-review findings logged in `events.log`. The downstream-search that found only `chickfila-android` as a consumer was scoped to the author's machine only — external consumers are structurally unobservable without an explicit policy for how to communicate changes.

## What "done" might look like

A new "Policy" section in HOW-TO-SYNC (or a sibling `POLICY.md`) that documents:

- **Consumer-compatibility stance**: is this plugin an author-owned curation surface where changes ship freely, or a public API with deprecation/communication obligations? If the latter, define the communication channel (CHANGELOG, GitHub release notes, something else).
- **Breaking-change response**: what the curator does when upstream ships a refactor that breaks verbatim-mirror assumptions. Options include pinning to a known-good revision, rewriting with local edits (first real divergence from verbatim-mirror), or deferring and reopening deferred-candidates review.

Exact shape is open; the ticket closes when the policies are written down somewhere discoverable.

## Out of scope

- Building any automation around consumer communication (CHANGELOG generator, release-note bot, etc.).
- Retroactively classifying past changes as breaking or non-breaking.
