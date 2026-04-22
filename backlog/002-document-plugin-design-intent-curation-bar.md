---
schema_version: "1"
uuid: d2654183-a31e-471c-b984-aa3a033b386c
title: "Refresh android-dev-extras HOW-TO-SYNC: dual upstream, design intent, android-cli decision"
status: backlog
priority: high
type: chore
tags: [android-dev-extras-registry]
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

Single doc refresh that closes the two gaps discovery surfaced (missed dac upstream, unstated plugin motivation) and makes the concrete `android-cli` curation call the discovery was convened to answer. One PR, not a multi-ticket program — the plugin has 2 skills and 1 known consumer; the work is proportionate.

## What the refresh covers

1. **Add a "Why this plugin exists" preamble** to HOW-TO-SYNC: per-project-togglable capsule for Android-repo projects; curation bar is non-destructive, non-version-coupled, cross-project-applicable skills.
2. **Rewrite the upstream section** to name both sources: `dl.google.com/dac/dac_skills.zip` (authoritative, per the `~/.android/cli/skills/.github/workflows/update-skills.yml` merge workflow) and `github.com/android/skills` (public overlay; convenient pull source today). Add a check-step: run `android skills list` on each refresh and reconcile with covered + deferred lists.
3. **Make the android-cli call inline**. Per research DR-1 option F: vendor with a detect-then-load guard (skill activation gated on `command -v android`). If the refresh cadence of `dl.google.com/dac/dac_skills.zip` later proves unreliable, revert to a deferred-candidate entry. This is the first Claude-specific patch on vendored content — HOW-TO-SYNC should capture it as an accepted divergence with rationale.

## Research context

All findings, decision records, and open questions in `research/android-dev-extras-registry/research.md`. Key points:

- Claude Code does NOT auto-load `~/.android/cli/skills/`, so vendoring `android-cli` is the only path for Claude Code users — not redundant with `android init` (this assumption was verified false during critical review).
- `android-cli`'s SKILL.md embeds literal `android help` output (lines 62–196), keyed to CLI version `0.7.15232955` — refresh obligation is real but bounded.
- Downstream (`chickfila-android`) already permits read-style `android`/`adb shell` commands; write-style subcommands (`screen capture`, `emulator`, `run`) would prompt on first use.

## Deferred — not in this ticket

The discovery surfaced several speculative follow-ups — drift-detection script, dac-URL-stability spike, external-consumer/breaking-change policy codification. They were considered and dropped as over-scoped for a personal 2-skill plugin with 1 known consumer. If the plugin grows (more consumers, more skills, more frequent upstream churn), reopen from the research artifact.
