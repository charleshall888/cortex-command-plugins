# Research: pr-review skill improvements

Target skill: `/Users/charlie.hall/Workspaces/cortex-command-plugins/plugins/cortex-pr-review/skills/pr-review/`

## Research Questions

1. **Anthropic's adversarial review recommendations.** → **Answered.** Anthropic's own `code-review` plugin (shipped in `anthropics/claude-code`) is the production reference. Pattern: Haiku early-exit + triage, Sonnet summary, four parallel reviewers (2 Sonnet for CLAUDE.md compliance, 1 Opus for bugs, 1 Opus for logic/security) scoped to the diff only, followed by **a precision-enforcing pass that drops un-validated findings**, followed by output + inline posting. Guiding principle: *"false positives erode trust"* — explicit do-not-flag list (style, linter-catchables, pedantic concerns, issues silenced by explicit comments). Our redesign absorbs the precision pass but collapses it into the synthesizer (see DR-2) rather than running it as a separate subagent fan-out — the synthesizer has strictly more context than a per-finding validator and avoids the N-subagent cost.

2. **Opus 4.7 leverage points for synthesis/filter stage.** → **Answered.** 4.7 released 2026-04-16. Key changes: adaptive thinking only (`thinking: {type: "adaptive"}`, `budget_tokens` removed — setting it 400s); new `xhigh` effort level recommended for intelligence-sensitive use cases; more literal instruction following; less validation-forward phrasing by default; fewer subagents spawned by default (steerable); new tokenizer uses ~1.0-1.35x more tokens. For the skill: (a) pin the synthesis stage to `claude-opus-4-7`; (b) load full diff + CLAUDE.mds + all critic outputs in one shot (1M context); (c) trim scaffolding prompts; (d) bump output token caps for the renderer stage. Caveat from critical review: "more literal instruction following" means the rubric prompt's exact wording matters more, not less — calibration examples need stability testing.

3. **GitHub review comment format.** → **Answered.** Suggestion syntax: ```` ```suggestion ```` fenced block inside a review comment body; contents replace the anchored line(s) verbatim. Single-line vs. multi-line controlled by the anchor (`line` alone vs. `start_line`+`line` with matching `side`/`start_side`). Nested fences need 4+ backticks outer. `gh pr review` has **NO inline comment support** — must use `gh api POST /repos/{o}/{r}/pulls/{n}/reviews` with a `comments[]` array and an `event` field. **Critical capability: `event=PENDING` creates a draft review visible only to the author, editable in GitHub's native UI, submittable with one click. This is our default posting path (see DR-5).** `side=RIGHT` for added/context lines; `side=LEFT` for deleted lines (suggestion blocks only "Commit suggestion"-work on `RIGHT`). `commit_id` binds the anchor — fetch `headRefOid` fresh before posting.

4. **Filtering rubric — simple enough to hold up, rich enough to be useful.** → **Answered.** Three axes, coarse buckets (not 1-5 integers), plus an explicit `unknown` path.
   - **Severity** (4 buckets): `must-fix` (correctness, security, data loss), `should-fix` (real bug or clear maintainability loss), `nit` (style or minor improvement), `unknown` (reviewer cannot confidently judge without more context).
   - **Solidness** (3 buckets): `solid` (finding quotes specific diff lines + has a concrete fix), `plausible` (finding names a specific concern but fix or evidence is hand-wavy), `thin` (finding is speculative or unanchored).
   - **Signal** (3 buckets): `high` (surfacing teaches the author or prevents a real future mistake), `medium` (surfacing is net-helpful but marginal), `low` (linter/formatter would catch it; surfacing is noise).
   
   **Gate:**
   - `severity=must-fix` AND `solidness ≥ plausible` → surface as `issue (blocking):`.
   - `severity=should-fix` AND `solidness=solid` AND `signal ≥ medium` → surface as `suggestion:`.
   - `severity=nit` AND `solidness=solid` AND `signal=high` → surface as `nit (non-blocking):`, capped at 3 per review (tie-break: file path alphabetical, then line ascending).
   - `severity=unknown` AND `solidness ≥ plausible` → surface as `question:` (never blocking, never with a fix — asking means the reviewer needs info).
   - Everything else → drop.
   - **`praise:` finding type** (not on the severity axis — orthogonal): surface if `solidness=solid` AND `signal=high`, capped at 2 per review. Author did something non-obvious that deserves calling out.
   - **`cross-cutting:` finding type** (for architectural drift that spans files and isn't line-anchored): surface if `solidness ≥ plausible` AND `signal=high`, capped at 1 per review. Bypasses the locality requirement.
   
   **Observability:** footer reports `N considered, X surfaced, Y dropped` (bucketed by why: unanchored / low-signal / linter-class / over-cap). User can ask for the drop list.

5. **Voice enforcement.** → **Answered.** Prompt-only is insufficient. Prompt instructions + deterministic post-filter pass. Em-dash strip is deterministic; vocabulary matches ("delve", "tapestry", "seamless", "meticulous", "leverage" as verb, "realm of", "not just X but Y", validation openers) trigger sentence-level regeneration (cheaper than full-review redo). Target ≤ 5% false-positive rate on a corpus of prior human-written review comments.

6. **Diagram decision rule.** → **Answered.** Emit a visual only if ALL hold: `severity ≥ should-fix` AND ≥ 3 nodes or ≥ 2 dimensions AND one of: (a) control/data flow crosses ≥ 3 files or ≥ 2 async boundaries; (b) state machine changes; (c) concurrency/lock sequencing; (d) refactor with ≥ 3 related call-site or symbol renames (before/after table); (e) type hierarchy change needing > 4 sentences of prose. Heuristic: *if a competent reader can hold the relationship in their head after one read of your prose, skip the diagram.* Explicitly forbidden: PR overviews, rename diagrams, single-function bugs, naming nits. Format: mermaid (GitHub renders it natively in PR comments).

7. **Fate of "Architectural Assessment" and "Consensus Positives".** → **Answered.** Threshold-gate, not cut. Both become first-class finding types in the rubric: `cross-cutting:` (architectural concern that spans files or deepens pre-existing fragility — capped at 1 per review) and `praise:` (author did something non-obvious worth reinforcing — capped at 2 per review). Both gated by `solidness=solid` + `signal=high`. If the bar isn't met, they don't surface. Conventional Comments defines both labels as first-class — we use that taxonomy end-to-end.

## Codebase Analysis

### Current skill structure (`plugins/cortex-pr-review/skills/pr-review/`)

Five-stage pipeline in `references/protocol.md`:
1. Stage 1: `gh pr view` + `gh pr diff --patch`.
2. Stage 2: Haiku triage (deep-review vs. skim-ignore).
3. Stage 3: four parallel Sonnet agents — CLAUDE.md compliance, bug scan, git history (`git log --follow -p` + `git blame`), previous PR comments.
4. Stage 4: Opus synthesis with fixed sections: High-Confidence Issues, Observations, **Architectural Assessment**, **Consensus Positives**.
5. Stage 5: main agent presents synthesis.

### Observed weaknesses vs. user complaints

| Complaint | Root cause in current protocol |
|---|---|
| (1) Surfaces findings that don't survive a "is this worth commenting?" pass | No rubric. Synthesis rule is "2+ agents = high-confidence, 1 agent = observation, never dismiss single-agent findings" — zero filtering by severity/actionability/signal. |
| (2) No code-suggestion blocks, no ready-to-post format | Output format is narrative prose with `<file, line, description>` bullets. No `suggestion` fence, no anchoring (file + line + side). |
| (3) Value of each comment not sold | No "why this matters" requirement. Nothing distinguishes a nit from a must-fix. |
| (4) Noise from Architectural Assessment + Consensus Positives | These are named sections in the Opus template — the agent produces content to fill them even when thin. |
| (5) No visuals | No guidance, no decision rule. |
| (6) AI-tell voice, em-dashes | No voice guide; no post-filter. |

### Reusable patterns from sibling skills

- **`critical-review`** (`~/.claude/skills/critical-review/`): pre-derive 3-4 **angles** specific to the artifact before dispatching agents. Angles must quote artifact text. Synthesis produces through-lines (multi-angle hits), tensions, concerns — **no "balanced" or "what went well" sections**. Partial-failure fallback: proceed with available angles.
- **`research`** (`~/.claude/skills/research/`): matrix-based agent count; adversarial agent (Agent 5) runs *after* others and challenges their findings. Injection-resistant prompts on any agent that fetches web content.
- **`devils-advocate`** (cortex-dev-extras): enforces four mandatory, *specific* sections — each demands artifact-grounded reasoning.

### Files affected by redesign

- `plugins/cortex-pr-review/skills/pr-review/SKILL.md` — front-matter unchanged; body updates to reflect new pipeline.
- `plugins/cortex-pr-review/skills/pr-review/references/protocol.md` — substantial rewrite (stages, prompts, output format).
- NEW: `references/rubric.md` — three-axis rubric, gate thresholds, worked examples on calibration PRs.
- NEW: `references/output-format.md` — Conventional Comments label taxonomy, `suggestion` block rules, pending-review payload shape, diagram decision rule, voice guide + post-filter regex list.

## Web & Documentation Research

### Anthropic's canonical multi-agent review template

`anthropics/claude-code/plugins/code-review/commands/code-review.md`:
- **Step 1 (Haiku): early-exit gate** — skip drafts, closed PRs, trivial changes, PRs already reviewed. Our current skill doesn't gate; we add it.
- **Step 4: four parallel reviewers scoped to the diff only** — two Sonnet for CLAUDE.md compliance, one Opus for bugs, one Opus for logic/security.
- **Step 5: precision-enforcement pass** re-checks each finding with the PR title/description as context; un-validated findings are dropped in Step 6.
- **Step 7-9:** output, then inline posting. One inline per unique issue. `suggestion` blocks used only when the suggestion fully fixes the issue.
- **Do-NOT-flag list:** style, linter-catchables, pedantic concerns, issues silenced by explicit comments, "could-be issues depending on state." Directive: *"false positives erode trust."*

**Our departure from Anthropic's pattern:** they run the precision pass as parallel validator subagents per finding. We collapse this into the synthesizer (Opus 4.7 with 1M context) — see DR-2. Same precision-enforcement function, different mechanism; avoids N extra subagent calls and the "validator with less context than the critic" failure mode.

**Multi-agent research system blog post** (Anthropic engineering): subagents need "an objective, an output format, guidance on the tools and sources to use, and clear task boundaries" — without that they "duplicate work, leave gaps, or fail to find necessary information."

**Sycophancy guidance** (Claude Constitution): "diplomatically honest rather than dishonestly diplomatic." Opus 4.7 is tuned for less validation-forward phrasing. CriticGPT (OpenAI) documents the symmetric failure mode: critics produce nitpicks and hallucinations when not grounded. Mitigations: evidence anchoring (diff-only), locked rubric with `unknown` bucket, explicit do-not-flag list.

### Opus 4.7 mechanics for a Claude Code skill

From `platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7`:
- **Adaptive thinking is the only thinking mode.** `budget_tokens` is removed.
- **`xhigh` effort level** recommended for coding and agentic use.
- **Behavior changes:** more literal instruction following (cuts both ways — rubric wording becomes load-bearing), fewer tool calls by default, more opinionated tone.
- **Tokenizer:** 1.0-1.35x more tokens than 4.6.

Skill-level application: pin `model: claude-opus-4-7` on the synthesis agent, load full context in one shot, trim scaffolding that 4.7 no longer needs. **Calibrate rubric wording with stability testing** (DR-1 Stage 1) before shipping.

### GitHub comment format (authoritative spec)

Four distinct surfaces:
1. **Issue comment** (general): `POST /repos/{o}/{r}/issues/{n}/comments`. Not diff-anchored.
2. **Review (the one we use)**: `POST /repos/{o}/{r}/pulls/{n}/reviews` with `commit_id`, `body` (summary), `event` (`COMMENT`/`APPROVE`/`REQUEST_CHANGES`/`PENDING`), `comments[]` (each with `path`, `line`/`start_line`, `side`/`start_side`, `body`).
   - **`event=PENDING`** creates a draft review visible only to the author. Author opens GitHub → sees "You have a pending review" → edits/deletes individual comments → submits with one click. This is our default posting path.
   - **`event=COMMENT/APPROVE/REQUEST_CHANGES`** posts immediately; reviewers are notified. Behind an explicit `--submit` flag.
3. **Standalone inline comment**: `POST /repos/{o}/{r}/pulls/{n}/comments`. Not used by us.
4. **Review summary**: the `body` field on the review object in #2.

**Suggestion block rules:**
- Single-line: anchor to one line, `suggestion` block replaces it.
- Multi-line: `start_line <= line`, both sides usually `RIGHT`.
- Nested fences: 4+ backticks on outer fence.
- `side=LEFT` (deleted lines): suggestion renders but "Commit suggestion" won't apply.
- Out-of-hunk `line` → 422.

**`gh` CLI:** `gh pr review` supports summary+verdict only. For inline comments must use `gh api`.

### Voice / de-AI guidance

- Prompt-only constraints degrade under long contexts.
- Dedicated post-pass (the `blader/humanizer` pattern) is more reliable: audit for specific tells against a checklist.
- High-signal tells: em-dashes (highest), "not just X, but Y", copula-avoidance, rule-of-three, validation openers, closing fluff.
- Vocabulary cluster: delve, tapestry, seamless, meticulous, leverage (as verb), realm of, in the landscape of, underscores, myriad, plethora, embark on, at the heart of, paves the way.
- Context over rules: examples of target voice beat negative rules. Regex post-filter is a cheap proxy.

## Domain & Prior Art

### Code review conventions

- **Conventional Comments** (`conventionalcomments.org`): label taxonomy — `issue:`, `suggestion:`, `nit:`, `question:`, `praise:`, `thought:`, `chore:`, with `(blocking)`/`(non-blocking)` decorators. The CC spec explicitly defines `praise:` as a first-class label — "A parting thought, a nod of respect, an encouragement. Provided there's a culture where praise is given intentionally." We adopt the full taxonomy, including `praise:` as a rubric-gated first-class output. The CC warning against **false** praise is about unearned praise, not all praise.
- **Google Engineering Practices** — reviewer standard: "reviewers should favor approving a PR once it is in a state where it definitely improves the code." Emphasizes "explain your reasoning," "provide guidance rather than just pointing out problems" — applies to nits *and* to positive reinforcement.

### Prior adversarial-review art

- **ASDLC "Adversarial Code Review" pattern**: Builder agent writes code; Critic agent in a separate session with different framing challenges it. Separation prevents echo-chamber validation. Our synthesis stage uses a *challenge* framing, not a *summarize* framing.
- **CriticGPT (OpenAI)**: critics produce nitpicks and hallucinations without grounding. Mitigation — force each finding to quote exact offending lines; synthesizer rejects findings that can't quote evidence.
- **LLM-as-judge literature** (evidently.ai, Confident AI, "Rulers: Locked Rubrics…" arXiv): prompt sensitivity is high; lock the rubric, require explicit `unknown` bucket, quote evidence.

### Commercial AI PR review tools (added in follow-up research pass)

Patterns adopted into the design, grouped by source:

- **Ellipsis** — (a) Attached `Evidence` objects on every finding (`{path, line_range, quoted_text}`) feed deterministic filter stages. Synthesizer rejects findings whose `quoted_text` doesn't appear literally in the diff. Ground-truth match, not LLM judgment. (b) Dropped-findings list exposed with per-comment reason, not just counts — trust signal for users. (c) LLMs are "notoriously bad at correctly identifying column numbers and often off-by-one on line numbers"; renderer snaps to the actual line by fuzzy-matching `quoted_text` against the diff hunk rather than trusting the LLM's line number. Eliminates a class of 422 out-of-hunk errors. ([ellipsis.dev/blog/how-we-built-ellipsis](https://www.ellipsis.dev/blog/how-we-built-ellipsis))
- **CodeRabbit** — (a) "Actionable" as the stricter judge criterion: a comment must contain enough information for the developer to act. Folded into the `solidness=solid` bucket definition (must name the concrete next action). (b) Walkthrough body format: collapsed top-of-PR comment with file-by-file one-liner summary + optional mermaid diagrams, separate from the inline findings list. Gives the author one-screen orientation. ([coderabbit.ai/blog/how-coderabbit-delivers-accurate-ai-code-reviews-on-massive-codebases](https://www.coderabbit.ai/blog/how-coderabbit-delivers-accurate-ai-code-reviews-on-massive-codebases))
- **Cloudflare** — (a) "Prompts define what to ignore": each critic's prompt has a named `Explicitly out-of-scope for this reviewer` block (e.g., security reviewer excludes "theoretical risks" and "defense-in-depth when primary defenses are adequate"). Noted as a future follow-up since it touches all four critic prompts, not the synthesizer. (b) Diff pre-filter (strip lock files, vendored deps, `@generated` files before any agent sees them). Future follow-up. (c) Risk-tiered agent count by diff size (2/4/7). Future follow-up. ([blog.cloudflare.com/ai-code-review](https://blog.cloudflare.com/ai-code-review))
- **Bugbot (Cursor)** — V1→V2 migration lesson: static context + restraint lost to dynamic context + aggression. Their `resolution rate` (findings that led to author changes) is a richer metric than label consistency. Both noted as future follow-ups. ([cursor.com/blog/building-bugbot](https://cursor.com/blog/building-bugbot))

Patterns NOT adopted (noted for completeness):
- Bugbot's learned-rules loop and Greptile's reaction-based tuning — require persistence model beyond the current epic's scope.
- GitHub Copilot Code Review's "always `event=COMMENT`, never block" posture — intentionally weaker than our `event=PENDING` + author-submits design.
- CodeRabbit's 20+ linter/SAST integration layer — out of scope; we're not replacing linters.

## Feasibility Assessment

| Approach | Effort | Risks | Prerequisites |
|---|---|---|---|
| **A: Full pipeline redesign — three-axis rubric, single-pass synthesizer (no separate validator), pending-review posting default, voice post-filter, new output format** | L | Breaking change for anyone invoking `/pr-review` in scripts; rubric wording stability needs calibration; post-filter false positives on technical vocabulary | None beyond current tooling. Keep `gh` + parallel Task dispatch. |
| **B: Rubric + output format only (no pending-review integration, no voice filter)** | M | Addresses filtering but leaves the workflow-friction and voice complaints | None |
| **C: Thin prompt changes** | S | Addresses symptoms not cause; prompt-only instructions already fail | None |

**Recommendation: A**, staged. See DR-1.

## Decision Records

### DR-1: Full redesign, staged rollout

- **Context**: User lists six distinct problems all pointing at the same root cause — the protocol produces broad, unfiltered output because there's no rubric, no precision pass, no voice discipline, and no posting integration. Prompt tweaks (Option C) have already failed.
- **Options considered**: A (full redesign), B (rubric + format only), C (prompt patches).
- **Recommendation**: **A**, broken into stages so each can ship independently.
  - **Stage 1 (foundation)**: three-axis rubric + output-format references, Conventional Comments label taxonomy (`issue` / `suggestion` / `nit` / `question` / `praise` / `cross-cutting`), threshold-gate Architectural Assessment and Consensus Positives (surface only when rubric passes). **Ship requirement: rubric wording stability test** — run the synthesizer on 3 real PRs, 3 times each, confirm ≥ 90% label consistency before declaring Stage 1 shipped.
  - **Stage 2 (single-pass synthesizer with precision enforcement)**: Opus 4.7 synthesizer receives all critic outputs + full diff in one context. Each critic output must include a structured `evidence: {path, line_range, quoted_text}` field (Ellipsis pattern). Synthesizer deterministically rejects findings where `quoted_text` doesn't appear literally in the diff, then applies the rubric to score survivors. Emits labeled output + footer of `N considered, X surfaced, Y dropped` plus a collapsed `<details>` block listing each dropped finding with its drop reason. No separate validator subagents.
  - **Stage 3 (renderer + pending-review posting)**: synthesizer emits structured JSON (path, line, start_line, side, body, suggestion, evidence); renderer converts to `gh api POST /pulls/{n}/reviews` payload with `event=PENDING` by default. **Line anchoring uses fuzzy match on `evidence.quoted_text` against the diff hunk** rather than trusting the LLM's line number (Ellipsis pattern, eliminates 422 out-of-hunk errors). Review `body` uses the CodeRabbit walkthrough format: verdict + file-by-file one-liner summary + optional mermaid + observability footer. `--submit` flag switches `event` to `COMMENT/APPROVE/REQUEST_CHANGES`. Paste-ready markdown is the fallback when posting fails.
  - **Stage 4 (voice pass)**: post-filter regex + focused sentence regeneration on AI tells. Calibrated against ≥ 5 prior human-written review comments in `~/.claude/projects/.../examples/` (to be collected).
  - **Stage 5 (diagram rule)**: strict trigger conditions in the renderer prompt. Mermaid output only.
- **Trade-offs**: Larger rewrite than prompt patches, but pays down the root cause. Calibration testing adds a ship gate to Stage 1, increasing time-to-first-ship in exchange for baseline stability. Accepted.

### DR-2: Collapse validator-pass into synthesizer, don't run it as parallel subagents

- **Context**: Anthropic's `code-review` plugin runs a per-finding validator subagent fan-out. The adversarial review of this research identified three problems with that pattern for our context: (a) the validator has strictly less context than the critic that produced the finding, so it disproportionately drops high-value history/prior-review findings; (b) N extra subagent calls per review with unverified prompt-caching behavior across Task-dispatched subagents; (c) duplicate filtering responsibility with the rubric, with no documented precedence.
- **Options considered**: (a) parallel validator subagents (Anthropic's pattern verbatim); (b) collapse validation into the synthesizer (synthesizer does evidence-grounding + rubric scoring in one pass, with full diff + all critic outputs + CLAUDE.mds in context); (c) no validation at all (trust critic outputs directly).
- **Recommendation**: **(b)**. The synthesizer runs on Opus 4.7 with 1M context — it has strictly more information than a per-finding validator subagent, so it can adjudicate evidence grounding at least as well, and often better. Single filter means single drop path, single audit trail, single place to tune behavior. Footer (`N considered, X surfaced, Y dropped with reasons`) makes every drop visible.
- **Trade-offs**: Departs from Anthropic's exact protocol — worth it because our constraints differ (Task-dispatched subagents, unverified prompt caching, user complaint (1) is specifically about filtering transparency). Synthesizer prompt becomes longer and more load-bearing; Stage 1 calibration test covers this risk.

### DR-3: Threshold-gate Architectural Assessment and Consensus Positives, don't cut

- **Context**: User's complaint (4) is "a lot of noise with the architectural assessment and consensus positive" — noise, not absence. Prior version of this research cut both entirely; adversarial review caught the over-correction. Cross-cutting architectural concerns are structurally unrepresentable as line-anchored findings (they span files or describe fragility deepening), so they need their own rubric path. Genuine praise has teaching value in peer/mentor review and is defined as a first-class label by Conventional Comments.
- **Options considered**: (a) cut entirely; (b) threshold-gate via rubric; (c) keep as named sections with soft guidance.
- **Recommendation**: **(b)**. Add two first-class finding types to the rubric: `cross-cutting:` (capped at 1 per review, gated `solidness ≥ plausible` AND `signal=high`, bypasses locality) and `praise:` (capped at 2 per review, gated `solidness=solid` AND `signal=high`). When the bar isn't met, they don't surface — no named sections, no manufactured content. When the bar IS met, the user gets the signal they were missing from DR-3's prior cut-entirely approach.
- **Trade-offs**: Rubric gets two more finding types to score. Acceptable — both are orthogonal to severity (architectural concerns can be must-fix or should-fix; praise is praise), and both have strict caps that prevent section bloat.

### DR-4: Voice enforcement via post-filter + targeted regen, not prompt-only

- **Context**: Complaint (6). User has tried prompt-only; it leaks.
- **Options considered**: (a) prompt-only with negative examples; (b) prompt + deterministic post-filter + targeted sentence regeneration; (c) example-based style transfer using prior human-written comments.
- **Recommendation**: **(b)** for Stage 4, with (c) as a later enhancement. Em-dashes get deterministic replacement (`. ` or ` - `). Vocabulary/structural tells get flagged; matched sentences get a single-sentence regen pass. Option (c) requires a corpus of user-written comments that doesn't exist yet — collected during Stage 4 calibration.
- **Trade-offs**: Regex false positives on `robust`/`leverage` in legit technical contexts. Flag, don't strip — let the regen decide.

### DR-5: Pending review as default posting path, paste-ready as fallback

- **Context**: Complaint (2) — "where to leave them suggested to me" — most naturally reads as *drafted as a review I can edit*, which is exactly GitHub's native pending-review UX (`event=PENDING`). Prior version of this research defaulted to paste-ready with auto-post behind a flag; adversarial review identified that (i) pending reviews are strictly safer than paste-ready (visible only to author, editable in GitHub's UI, deletable with one click), (ii) renderer already emits the structured JSON needed for `gh api /reviews`, so posting is ~20 lines on top of rendering, (iii) paste-ready format adds placement metadata tokens to every run that the user must manually parse and act on.
- **Options considered**: (a) paste-ready only; (b) immediate auto-post with confirmation; (c) both modes behind a flag with paste-ready default; (d) pending-review default, immediate-submit behind a flag, paste-ready as fallback for posting failures.
- **Recommendation**: **(d)**. Default: `gh api POST /pulls/{n}/reviews` with `event=PENDING`. User sees terminal summary ("Posted 5 comments as pending review — open PR to edit/submit"), opens GitHub, edits/deletes/submits in the native UI. `--submit` flag switches `event` to `COMMENT/APPROVE/REQUEST_CHANGES` for high-trust runs. Paste-ready markdown is emitted only when posting fails (auth error, out-of-hunk 422, stale commit_id).
- **Trade-offs**: Skill now does a write operation by default (creates a pending review). Acceptable because pending reviews are private to the author and reversible with one click — they do not notify reviewers, they do not appear in the PR's public conversation until the author submits. Failure paths (out-of-hunk, stale SHA) need graceful fallback to paste-ready; handled in Stage 3.

## Open Questions

- **Calibration corpus for rubric.** Stage 1's ship gate requires ≥ 90% label consistency across 3 runs on 3 real PRs. Which PRs? Suggest: one recent merged PR with known nits, one recent PR with a real bug, one pure refactor. Confirm selection before Stage 1 ships.
- **Calibration corpus for voice filter.** ≥ 5 prior human-written review comments. Collect from: user's past GitHub activity, or synthesize from user instructions if unavailable at Stage 4 start.
- **Prompt caching behavior across Task-dispatched subagents.** Unverified. DR-1 no longer leans on caching as a cost defense — synthesizer is now one call, not N. Caching still helps if it works but isn't load-bearing.
- **Breaking changes to verdict output.** Current skill emits `APPROVE | REQUEST CHANGES | REJECT`. New output replaces verdict with a footer summary ("5 findings: 1 blocking, 2 suggestions, 2 nits"). If anyone scripts against the verdict keyword, preserve a `Verdict: APPROVE | REQUEST CHANGES | REJECT` line at the top of the footer for back-compat. Confirm whether this is needed (probably not — the skill is interactive-first).
- **Nit cap interaction with user agency.** Cap is 3 nits. Tie-break deterministic (file path alphabetical, line ascending). If the user wants more, the summary footer notes `(2 additional nits suppressed — run with --all-nits to see them)`.
- **Diagram format.** Mermaid confirmed — GitHub renders natively in PR comments. No need for ASCII or PlantUML.
