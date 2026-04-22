---
schema_version: "1"
uuid: 028b432a-f709-4b84-9ce4-a7b33064ef2b
title: "Add drift-detection script (android skills list vs HOW-TO-SYNC coverage)"
status: backlog
priority: medium
type: feature
tags: [android-dev-extras-registry, automation]
parent: 1
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

The missed-`android-cli` discovery is the canonical failure mode of a human-maintained deferred list: the curator sees what's in front of them, and a new upstream skill that isn't mentioned anywhere doesn't get considered. A script that diffs `android skills list` against HOW-TO-SYNC's covered + deferred sets converts this class of failure from "maybe noticed on the next refresh" to "surfaced automatically whenever the script runs."

## Research context

`android skills list` (post-`android init`) emits a flat list of 7 skills. HOW-TO-SYNC has a "covered" set (the 2 vendored skills) and a "deferred candidates" set (currently 4 with reasons). The union should equal the CLI's list; any asymmetry is a signal.

See `research/android-dev-extras-registry/research.md` Open Questions and the Feasibility Assessment Option A notes.

## What "done" might look like

A small script (likely under `scripts/` in this repo) that:

- Parses `android skills list` output (or falls back to a manually-updated local snapshot if the CLI isn't installed — useful for CI).
- Parses HOW-TO-SYNC to extract the covered + deferred skill lists.
- Reports any skill present in one set but not the other.
- Returns non-zero if drift is detected (so it can be wired into CI or a pre-refresh hook).

Shape is open: a shell script, a short Python script, or a Makefile target are all reasonable.

## Out of scope

- Actually running the script in CI (separate follow-up once the script exists).
- Auto-updating HOW-TO-SYNC from the CLI output — the script should detect drift, not silently mutate curation.
