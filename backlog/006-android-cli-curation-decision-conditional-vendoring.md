---
schema_version: "1"
uuid: 855363d8-23f7-404a-bdff-669a83ec2fa4
title: "android-cli curation decision + conditional vendoring"
status: backlog
priority: medium
type: feature
tags: [android-dev-extras-registry, curation]
parent: 1
blocked-by: [4]
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

`android-cli` is the only skill in the merged Android catalog that HOW-TO-SYNC's deferred list doesn't mention — pure gap, not a decision. Closing it (either with a formal defer-with-reason or a vendoring) gives future reviewers a complete curation record.

## Research context

Discovery's DR-1 recommends **option F** (vendor with detect-then-load guard), conditional on the spike (#4) confirming that dac refresh cadence is predictable. Key findings that shaped the recommendation:

- Claude Code does NOT auto-load `~/.android/cli/skills/`, so "CLI install = free skill" is not true — vendoring is the only viable path for Claude Code users.
- The skill embeds literal `android help` output (SKILL.md lines 62–196), version-keyed to CLI `0.7.15232955`. Refresh obligation is real.
- Downstream (`chickfila-android`) already permits read-style `android` and `adb shell` commands but not write-style subcommands (`screen capture`, `emulator`, `run`). Vendoring would surface new permission prompts for write-style flows.

If spike #4 finds the dac URL or cadence unreliable, the recommended path falls back to **option C** (document as a formally-deferred skill with reason).

See `research/android-dev-extras-registry/research.md` DR-1 and the `android-cli` profile in the Web & Documentation Research section.

## What "done" might look like

- A decision recorded (either "vendor, with guard" or "deferred, with reason") and committed to HOW-TO-SYNC's covered or deferred list.
- If vendoring: the skill copied in under the plugin's `skills/` path, LICENSE/NOTICE updated as needed, and a detect-then-load guard added to its SKILL.md trigger logic. This would be the first Claude-specific adaptation in the plugin — HOW-TO-SYNC should capture the precedent.
- If deferring: a one-line entry in the deferred-candidates list explaining the reason (e.g., "dac bundle refresh cadence unverified; revisit after spike #4 is re-run").

## Out of scope

- Updating other skills in response to this decision.
- Building the detect-then-load guard as a reusable pattern for other skills (future work if and when it applies to more than one).
