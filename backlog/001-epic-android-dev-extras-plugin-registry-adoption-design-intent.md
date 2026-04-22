---
schema_version: "1"
uuid: 6bb65dd7-6cad-42e3-97c4-fb84da32f416
title: "Epic: android-dev-extras plugin registry adoption & design intent"
status: backlog
priority: high
type: epic
tags: [android-dev-extras-registry]
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Scope

The `plugins/android-dev-extras/` plugin currently vendors two skills (`r8-analyzer`, `edge-to-edge`) verbatim from `github.com/android/skills` under a sync policy documented in `HOW-TO-SYNC.md`. Discovery (April 2026) surfaced that:

- Google now ships an Android CLI (`/usr/local/bin/android`) whose `skills` subcommand references a merged catalog, authoritatively sourced from `dl.google.com/dac/dac_skills.zip`. HOW-TO-SYNC documents only the GitHub overlay — so any dac-only skill (notably `android-cli`) is invisible to the current curator.
- The plugin has no plugin-level documentation of *why* it exists; the per-project-toggle motivation is stated in the marketplace root `README.md` but not in the plugin itself.
- Vendoring `android-cli` is viable (contrary to an initial working hypothesis) because Claude Code does **not** auto-discover skills from `~/.android/cli/skills/` — a plugin is the only reasonable path for Claude Code users. Version-coupling of the skill's embedded CLI help output is a real concern and conditions the decision on dac refresh cadence.

## Children

This epic groups the tickets that close these gaps:

1. **#2** — Document plugin design intent + curation bar (codify the implicit contract).
2. **#3** — Update HOW-TO-SYNC for dual upstream (dac + github) with a refresh check-step.
3. **#4** — Spike: confirm `dl.google.com/dac/dac_skills.zip` stability & refresh cadence.
4. **#5** — Add drift-detection script (`android skills list` vs HOW-TO-SYNC coverage).
5. **#6** — `android-cli` curation decision + conditional vendoring (depends on #4).
6. **#7** — Codify external-consumer + breaking-change response policies in HOW-TO-SYNC.

## Suggested order

Start with #2 and #3 in parallel (documentation wins, unlock clarity for other work). Run #4 (spike) next — it's the blocker on #6. #5 can run any time but is most valuable once #3 is in. #7 benefits from #2 being done first so vocabulary stays consistent. #6 is the culmination and should be sequenced last.

## Discovery context

Full research, decision records, and open questions are in `research/android-dev-extras-registry/research.md`. The critical-review findings that inverted DR-1's recommendation are logged in `research/android-dev-extras-registry/events.log`.
