---
schema_version: "1"
uuid: d69dda29-70dc-44da-ab36-9cc3c7c938a4
title: "Add pr-review voice post-filter: em-dash strip and AI-tell sentence regeneration"
status: ready
priority: medium
type: feature
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
parent: 3
blocked-by: [5]
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

Makes the review comments sound like the user, not like AI. Prompt-only constraints leak under long contexts — user has observed this. A deterministic post-filter pass catches tells the synthesizer keeps writing despite explicit instructions not to. Em-dash replacement is deterministic (no model call); vocabulary and structural tells trigger targeted single-sentence regeneration (cheaper than full-review redo). Addresses user complaint (6) directly.

## Research context

See `research/pr-review-skill-improvements/research.md` — Question 5 (voice enforcement via post-filter + targeted regen), DR-4 (prompt-only is insufficient), Web & Documentation Research (high-signal tells cluster).

## What this ticket delivers

New reference `plugins/cortex-pr-review/skills/pr-review/references/voice-filter.md` and a voice-pass stage in `references/protocol.md` that runs on rendered comment text before the posting step:

- Deterministic transforms (no model call):
  - Em-dashes (U+2014) replaced with either `. ` or ` - ` depending on context.
  - Double-hyphen proxies (` -- `) collapsed to ` - `.
- Regex-flagged patterns trigger sentence-level regeneration:
  - Validation openers: "Certainly", "Great question", "I hope this helps", "Happy to help".
  - Hedge/filler: "It's worth noting", "It's important to note".
  - "Not just X, but Y" negative parallelism.
  - Closing fluff: "Let me know if you have any questions", "Happy reviewing".
  - AI-tell vocabulary cluster: `delve`, `tapestry`, `seamless`, `meticulous`, `leverage` (as verb), `realm of`, `in the landscape of`, `underscores`, `myriad`, `plethora`, `embark on`, `at the heart of`, `paves the way`.
  - `robust` and `leverage` flagged (not auto-stripped) — regen decides based on context.
- Regen prompt: "rewrite this sentence without the flagged word, same meaning, ≤ same length".
- Calibration target: ≤ 5% false-positive rate. Calibrated against ≥ 5 prior human-written review comments — collected during this ticket's implementation (from user's GitHub activity or synthesized from user instructions if unavailable).

## Out of scope

- Example-based voice transfer using a full corpus of user-written review comments (epic-level follow-up once corpus exists).
- Adjusting the synthesizer prompt to reduce tells at source — rely on the post-filter, since prompt-only has already been shown to leak.
