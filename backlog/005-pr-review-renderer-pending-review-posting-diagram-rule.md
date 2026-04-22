---
schema_version: "1"
uuid: 8a173f35-8f46-4266-a811-6b87f2c23bcf
title: "Add pr-review renderer, pending-review posting via gh api, and diagram decision rule"
status: ready
priority: high
type: feature
tags: [pr-review-skill-improvements]
created: 2026-04-22
updated: 2026-04-22
parent: 3
blocked-by: [4]
discovery_source: research/pr-review-skill-improvements/research.md
---

## Value

Closes the workflow loop. The synthesizer (ticket 005) produces labeled findings; this ticket turns them into an actual GitHub review the user can edit and submit with one click. Defaults to `event=PENDING` — the review is drafted, visible only to the author, editable/deletable in GitHub's native UI. Paste-ready markdown is the fallback when posting fails. Adds the diagram decision rule to the renderer prompt so mermaid visuals surface only when they earn their keep. Directly addresses user complaints (2) and (5).

## Research context

See `research/pr-review-skill-improvements/research.md` — Question 3 (GitHub comment format, `gh api /reviews` with `event=PENDING`), Question 6 (diagram decision rule), DR-5 (pending-review as default posting path).

## What this ticket delivers

New renderer stage in `plugins/cortex-pr-review/skills/pr-review/references/protocol.md` after the synthesizer:

- Converts synthesizer's labeled findings into the GitHub review payload: `commit_id` (fresh `headRefOid`), `body` (walkthrough + summary + footer, see below), `event` (default `PENDING`), `comments[]` with `path`/`line`/`start_line`/`side`/`start_side`/`body`.
- `suggestion` blocks used only when the suggestion fully fixes the issue (per Anthropic's pattern); 4+ backtick outer fence for nested code blocks.
- **Fuzzy line-number anchoring**: the renderer does NOT trust the synthesizer's line number. Instead it matches the finding's `evidence.quoted_text` against the diff hunk and snaps to the actual line where that text appears. Eliminates a class of 422 out-of-hunk errors that Ellipsis publicly flagged as a known LLM failure mode ("LLMs are notoriously bad at correctly identifying column numbers and often off-by-one on line numbers"). If the quote doesn't match any line in the hunk, the renderer demotes the finding to the review body with a note, rather than posting to the wrong line.
- **Walkthrough body format** (CodeRabbit pattern): review `body` contains (1) one-line verdict + footer counts, (2) collapsed `<details>` walkthrough — file-by-file one-liner summary of what changed, (3) optional mermaid diagram if the diagram rule triggers, (4) the synthesizer's observability footer with dropped findings. Gives the reviewer one-screen orientation before opening the inline comments.
- Diagram decision rule embedded in renderer prompt: emit a mermaid diagram only when severity ≥ should-fix AND one of the structural triggers fires (cross-file flow, state machine change, concurrency sequencing, ≥ 3-site refactor, non-trivial type hierarchy change). Explicit exclusions for PR overviews, renames, single-function bugs, naming nits. Diagram goes in the body walkthrough, not an inline comment.
- Posting path: `gh api POST /repos/{o}/{r}/pulls/{n}/reviews --input payload.json` with `event=PENDING`. `--submit` flag switches `event` to `COMMENT` / `APPROVE` / `REQUEST_CHANGES`.
- Terminal output after posting: one-line summary (`Posted 5 comments as pending review, open PR to edit/submit: <URL>`) plus the observability footer.
- Graceful fallback: if posting fails (auth, out-of-hunk 422, stale `commit_id`), emit paste-ready markdown with explicit drag-select instructions for multi-line comments. This is the fallback path only, not the default.

Updates `SKILL.md` description and `argument-hint` to reflect `--submit` flag. Adds a `--paste` flag for users who prefer the old paste-ready behavior on demand.

## Out of scope

- Voice post-filter pass — next ticket (006). Rendered text is handed to the voice filter before posting.
- Risk-tiered agent count, diff pre-filter upstream of critics, learned-rules loop, circuit-breaker model failback — epic-level follow-ups surfaced from commercial-tool research, not in this ticket.
- Corpus-based voice transfer — epic-level follow-up.
