# Research: Refresh android-dev-extras HOW-TO-SYNC — dual upstream, design intent, android-cli decision

## Epic Reference

Epic research: [`research/android-dev-extras-registry/research.md`](../../research/android-dev-extras-registry/research.md). The epic evaluated vendor-or-skip options A–F and recommended option F (vendor with detect-then-load guard) per DR-1; this ticket executes the epic's recommended path with implementation-level details scoped to this PR.

## Codebase Analysis

### Files that will change

| Path | Type | Purpose |
|------|------|---------|
| `plugins/android-dev-extras/HOW-TO-SYNC.md` | MODIFY | Add design-intent preamble; rewrite Upstream section for dual upstream + check-step; add accepted-divergence entry for `android-cli` |
| `plugins/android-dev-extras/skills/android-cli/SKILL.md` | CREATE | Vendored from `~/.android/cli/skills/android-cli/SKILL.md` (197 lines) + Claude-specific guard |
| `plugins/android-dev-extras/skills/android-cli/references/interact.md` | CREATE | Verbatim copy (82 lines) |
| `plugins/android-dev-extras/skills/android-cli/references/journeys.md` | CREATE | Verbatim copy (97 lines) |

No downstream `.claude/settings.local.json` changes required — `chickfila-android` already permits read-only `android` subcommands.

### Source skill structure (`~/.android/cli/skills/android-cli/`)

- `SKILL.md` (197 lines): 2-field frontmatter (`name`, `description` only, no `preconditions`/`inputs`/`triggers`); body lines 1–61 are structured sections; lines 62–196 are literal `android help` output keyed to CLI version `0.7.15232955`.
- `references/interact.md` (82 lines), `references/journeys.md` (97 lines) — referenced from SKILL.md body.
- Total: ~2,420 words across 3 files.

### Existing skill frontmatter patterns (observed in plugins/)

- `cortex-pr-review/skills/pr-review/SKILL.md` (line 10) declares `preconditions:` array with human-readable strings (e.g., "GitHub CLI (gh) installed and authenticated").
- `cortex-ui-extras/skills/ui-lint/SKILL.md` (line 9) and `ui-check` similarly use `preconditions:`.
- `android-dev-extras/skills/r8-analyzer/SKILL.md` and `edge-to-edge/SKILL.md` use `name`, `description`, `license`, `metadata` — **no `preconditions`** (vendored verbatim from Google).
- **All existing plugin-authored skills use `preconditions`** for dependency documentation; all Google-vendored skills do not.

### Existing HOW-TO-SYNC structure (60 lines)

```
Lines 1-3:   Title + one-sentence intro
Lines 5-9:   "## Upstream source" (3 bullets; GitHub only)
Lines 11-23: "## Path mapping (normative)" (flattening rules + table)
Lines 25-40: "## Sync procedure" (6 steps)
Lines 42-47: "## Apache 2.0 attribution obligation" (§4 requirements)
Lines 49-59: "## Curation decisions worth revisiting" (deferred candidates)
```

Natural insertion points:
- Design-intent preamble → after line 3 (before `## Upstream source`).
- Dual-upstream rewrite → lines 5–9 (replace in place).
- `android skills list` reconciliation check-step → new step within `## Sync procedure` (after existing step 3).
- `android-cli` accepted-divergence entry → either (a) within existing `## Curation decisions` section promoted to a `covered` sub-list, or (b) as a new `## Accepted divergences` section.

### Sibling plugin structural pattern

All 4 marketplace plugins share identical shape:
- `.claude-plugin/plugin.json` — single-line `{"name": "<plugin>"}` only; no hooks, no lifecycle bindings, no skills registration.
- Optional `LICENSE`, `NOTICE` (`android-dev-extras` has both; others omit per license compatibility).
- `skills/<skill>/SKILL.md` + optional `references/`.
- **No plugin-level README.md in any plugin** — marketplace README at repo root is the only overview.

Downstream enablement (chickfila-android `.claude/settings.local.json`): `"android-dev-extras@cortex-command-plugins": true` is already present; toggle is boolean.

### Plugin validation constraints

`scripts/validate-skill.py` (enforced in CI via `.github/workflows/validate.yml`):
- Required frontmatter: `name`, `description` (hard gate — exit 0 required).
- Warns if body > 500 non-blank lines (suppressible via `# noqa: body-length`); `android-cli` at 197 lines is well under.
- Warns on inputs/variables mismatch (non-blocking).

### Downstream permission surface (chickfila-android)

Already permitted (read-only `android` surface):
- `Bash(android --version)`, `Bash(android describe *)`, `Bash(android docs *)`, `Bash(android list *)`, `Bash(command -v android)`, `Bash(adb -s emulator-5554 shell pm list packages)`.

NOT permitted — would prompt on first invocation if skill uses these:
- `Bash(android screen capture *)`, `Bash(android screen resolve *)` — invoked in `references/interact.md` line 25, 37.
- `Bash(android emulator *)` — invoked in SKILL.md line 44.
- `Bash(android run *)` — invoked in SKILL.md line 39.

## Web Research

### DAC bundle URL verification

- `https://dl.google.com/dac/dac_skills.zip` — **VERIFIED LIVE as of 2026-04-22**. HEAD + GET both return HTTP/2 200, `content-type: application/zip`, `content-length: 167266`, `last-modified: Tue, 14 Apr 2026 12:41:51 GMT`, `etag: "5bd6239"`, `cache-control: public, max-age=86400`, `server: downloads`.
- Unzip confirms identical content tree to `github.com/android/skills` at tag `v0.0.2` (DAC zip: 163KB; GitHub release zip: 168KB — difference is `README.md` absent from DAC packaging).
- Google does NOT publicly document this URL in [android-cli docs](https://developer.android.com/tools/agents/android-cli) or [android-skills docs](https://developer.android.com/tools/agents/android-skills). Both pages point at the GitHub repo as canonical.
- Cadence: two releases in first week (v0.0.1 on 2026-04-13, v0.0.2 on 2026-04-14). Treat DAC URL as **authoritative-but-undocumented mirror**; GitHub tag is the publicly-attested ground truth.

### Claude Code skill loader capabilities

From [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills) — **canonical frontmatter fields**: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell`.

- **No `preconditions`, `requires`, `detect`, `if-exists`, or binary-precondition field exists in the spec.** The `preconditions:` field observed in sibling plugin skills (pr-review, ui-lint) is user-authored documentation only — Claude Code does not enforce it; the loader silently ignores unknown keys.
- **Mechanisms that *can* implement a detect-then-load guard**:
  - `` !`command -v android || echo NOT_INSTALLED` `` — documented dynamic context injection; output is interpolated inline before Claude reads the body.
  - `hooks:` frontmatter — heavier, deterministic skill-scoped hook.
  - `paths:` glob gate — fires on matching working files, not host binaries.
- **Anti-pattern**: inventing a non-existent frontmatter field and relying on it for enforcement — it will be silently ignored.

### `android skills list` diffability

- Documented form: `android skills list [--long]`. No `--json` / `--format` flag documented in [android-cli docs](https://developer.android.com/tools/agents/android-cli).
- Plain text output. For a reconciliation check-step, line-based `diff`/`comm -23` against a committed baseline is the safest approach.
- `--long` adds description + installed-for-agents columns.

### Apache 2.0 §4(b) compliance patterns for Claude-specific patches

[Apache License 2.0 §4(b)](https://www.apache.org/licenses/LICENSE-2.0) requires: *"You must cause any modified files to carry prominent notices stating that You changed the files"* — form unspecified.

Widely-used conventions:
- **Chromium `README.chromium`** — per-directory README with `Local Modifications:` section + numbered `.diff` files.
- **Debian quilt `debian/patches/series`** — pristine upstream + ordered patch queue file.
- **Inline header comment + repo-level `PATCHES.md`** — lightweight; marker in each modified file points at a central enumeration.

### Related prior art

- [skydoves/android-skills-mcp](https://github.com/skydoves/android-skills-mcp) — bundles `android/skills` for Claude Code. Notable: gitignored clone + `sync-skills.mjs` regeneration script. **No detect-then-load guard** — assumes consumers pre-parse rather than call the live CLI.

## Requirements & Constraints

### Apache 2.0 obligations (plugins/android-dev-extras/LICENSE, NOTICE)

- **LICENSE**: verbatim copy of Apache 2.0. Must remain byte-identical.
- **NOTICE**: plugin-authored per §4(d) ("Copyright 2026 Google LLC"). Upstream ships no NOTICE; if upstream adds one on a future refresh, copy verbatim.
- HOW-TO-SYNC line 47: "only modify SKILL.md content if strictly necessary — prefer verbatim vendoring." The guard patch on `android-cli` SKILL.md is the first case where "strictly necessary" applies.

### Existing HOW-TO-SYNC policy (preserve vs. rewrite)

| Policy | Line | Action |
|---|---|---|
| Single upstream `github.com/android/skills` | 7 | **REWRITE** → dual upstream per DR-2 |
| Verbatim-mirror stance (line 8, 47) | 8, 47 | **PRESERVE** as default; explicitly carve out accepted-divergence category |
| Contributions paragraph | 9 | **PRESERVE** |
| Path-flattening table + collision rule | 11–23 | **PRESERVE** |
| Sync procedure (steps 1–6) | 28–40 | **ADD reconciliation step** (run `android skills list`, diff against covered + deferred inventory) |
| Apache 2.0 §4 attribution | 42–47 | **PRESERVE**; reference from accepted-divergence entry |
| Curation decisions worth revisiting | 49–59 | **UPDATE**: `android-cli` moves from "absent" to "covered (with guard)"; keep existing deferrals |

### Plugin validation (`scripts/validate-skill.py`)

- Required fields: `name`, `description` — must be present in every vendored SKILL.md (android-cli already has these).
- CI gate: `python3 scripts/validate-skill.py plugins/android-dev-extras/skills` must exit 0.

### Downstream permission contract (chickfila-android)

Already-permitted subcommands cover `describe`, `docs`, `list`, `--version`, `command -v` flows. Permission prompts would fire for `screen capture`, `screen resolve`, `emulator`, `run` — acceptable per ticket's "proportionate" framing and user owns that downstream project.

### Sibling plugin structural invariants

- No new fields in `.claude-plugin/plugin.json`.
- No new top-level hooks/slash-commands for this ticket.
- No new `README.md` at plugin root (sibling plugins don't have one; design intent stays in HOW-TO-SYNC).

### Scope boundaries (per backlog item + decomposed.md)

**IN**: HOW-TO-SYNC refresh (preamble, dual upstream, check-step, android-cli entry), android-cli vendoring (files + guard). **OUT**: drift-detection script, dac-URL-stability spike, external-consumer/breaking-change policies, trigger-phrase tuning to CFA vocabulary, `/lifecycle` integration hooks.

## Tradeoffs & Alternatives

### Decision 1 — Guard mechanism for android-cli SKILL.md

| Alt | Description | Implementation cost | Re-sync friction | Enforcement |
|---|---|---|---|---|
| A | `preconditions:` frontmatter field (like sibling plugin-authored skills) | Low | Low (separate from upstream body) | **None — field is documentation-only; loader ignores it** |
| B | Opening `## When to use` / prose paragraph gating on `command -v android` | Low | Low | Relies on model reading |
| C | `` !`command -v android \|\| echo NOT_INSTALLED` `` dynamic context injection at top of SKILL.md | Low-Medium | Low-Medium (prefix block, separate from upstream body) | Strong — output is injected before Claude sees body; body can branch explicitly |
| D | `hooks:` frontmatter with skill-scoped pre-execution check | Medium-High | High (new mechanism, new file) | Strong |
| E | `paths:` glob gate | N/A | N/A | Matches files, not binaries — inapplicable |

**Key insight**: Codebase agent observed `preconditions:` in plugin-authored skills. Web agent verified that `preconditions:` is **not in the canonical Claude Code frontmatter spec** — it's documentation the loader ignores. So alternative A provides no enforcement beyond model-read; it's effectively alternative B with YAML syntax.

**Recommended**: **C (dynamic context injection `` !`command -v android` ``)** — the only documented mechanism with actual load-time enforcement in the Claude Code spec.

**Fallback if C conflicts with upstream SKILL.md structure**: **B** (leading paragraph), accepting model-read semantics.

**Rationale**: The ticket's "accepted divergence" framing explicitly calls out that this is a Claude-specific patch; using a Claude-specific mechanism (dynamic context injection is documented in Claude Code skills spec) is consistent. `preconditions:` is superficially tempting because sibling skills use it, but the Web agent verified those are silently ignored — it would be false comfort.

### Decision 2 — Vendoring source for android-cli files

| Alt | Description | Reproducibility | URL/state stability | CI-friendliness |
|---|---|---|---|---|
| A | `dl.google.com/dac/dac_skills.zip` (authoritative DAC bundle) | High (content-addressable via etag) | Medium (URL is undocumented by Google; subject to change) | Yes |
| B | Local `~/.android/cli/skills/android-cli/` (user's machine) | Low (machine-dependent) | N/A | No |
| C | `github.com/android/skills` (public overlay) | High | High (publicly attested) | Yes |

**Recommended for this PR**: **C for GitHub checkout + A for policy primary** — pull content from GitHub at tag `v0.0.2` (stable git ref; browsable diffs); document DAC as primary-authoritative in HOW-TO-SYNC with fallback note. Verified: GitHub `v0.0.2` content is byte-identical to DAC v0.0.2.

**Why not B**: non-reproducible on a CI agent or any machine without `android init` already run.

**Why not pure A**: URL stability is an open question (Google doesn't document the URL); fetching via zip loses git provenance (tag + commit hash). Use GitHub for concrete fetch; reference DAC as the true source of truth so future sync procedures know to check DAC when a suspected skill is missing from GitHub.

### Decision 3 — Accepted-divergence documentation form

| Alt | Description | Discoverability | Re-sync conflict surface | Scalability |
|---|---|---|---|---|
| A | Dedicated `## Accepted divergences` section in HOW-TO-SYNC.md | High (from HOW-TO-SYNC) | Low (separate from vendored content) | High |
| B | Inline header comment in modified SKILL.md (e.g., `<!-- CFA-PATCH: see HOW-TO-SYNC -->`) | Low (must read each SKILL.md) | Medium (may collide with upstream edits) | Low |
| C | Both A + B (redundant) | Highest | Medium | Medium (double maintenance) |
| D | Separate `PATCHES.md` sidecar file at plugin root | Medium (requires awareness of the file) | Low | High |

**Recommended**: **A (dedicated `## Accepted divergences` section in HOW-TO-SYNC.md)**. HOW-TO-SYNC is the canonical curator document; a section there is the natural home for patch policy. Scales to future patches without requiring new files.

**Optional reinforcement**: if an inline marker in `android-cli/SKILL.md` is desired for re-sync safety, add a single HTML comment at the top referencing the HOW-TO-SYNC section — but treat this as ancillary, not primary.

### Decision 4 — Design-intent preamble placement

| Alt | Description | Discoverability | Maintenance overhead |
|---|---|---|---|
| A | Top of HOW-TO-SYNC (before any other section) | Medium (mixes motivation + procedure) | Low |
| B | Dedicated `## Why this plugin exists` section after the one-line intro | High (natural reading order) | Low |
| C | Separate `plugins/android-dev-extras/README.md` | High (root-level README convention) | Medium (new file, two docs to maintain) |

**Recommended**: **B (`## Why this plugin exists` section after line 3 of HOW-TO-SYNC)**. Sibling plugins don't have README.md files — consistency favors keeping design intent in HOW-TO-SYNC. Curators already read HOW-TO-SYNC during refresh; this puts motivation in their path.

### Decision 5 — `android skills list` reconciliation check-step placement

| Alt | Placement | Pros/Cons |
|---|---|---|
| A | Between existing Sync step 3 (diff-summary) and step 4 (validation) | Fits naturally — curator has just seen upstream diffs; next step is "are there new skills I haven't decided on?" |
| B | As a new first step before Sync step 1 | Premature — diffing against upstream is more useful after content is fetched |
| C | Inside existing step 3 as a sub-bullet | Cramped — the check is a meaningfully distinct operation |

**Recommended**: **A** — insert as a new step between current steps 3 and 4.

## Open Questions

- **DR-1's conditional on DAC refresh cadence**: the epic's DR-1 recommended option F *pending refresh-cadence confirmation* for `dl.google.com/dac/dac_skills.zip`. Web research verified the URL is live (2 releases in 2 days early April) but Google does not publicly document the URL or its cadence. **Deferred**: proceed with option F per the ticket body's fallback policy ("If the refresh cadence later proves unreliable, revert to a deferred-candidate entry"). HOW-TO-SYNC should note the caveat.
- **Guard mechanism final choice (C vs. A/B)**: The Codebase agent observed `preconditions:` in sibling skills, but the Web agent verified the field is not in the Claude Code frontmatter spec and is silently ignored by the loader. The Tradeoffs agent recommended a leading prose paragraph (B). The three agents diverge on what "the first Claude-specific patch" should actually look like. **For Spec to resolve**: confirm whether the guard should use (C) dynamic context injection, (B) prose paragraph, or (A) `preconditions:` YAML (accepting it's advisory-only).
- **`android-cli` position in HOW-TO-SYNC curation section**: the existing section is titled "Curation decisions worth revisiting" and lists deferrals. Should it be retitled/split (e.g., "Covered skills" + "Deferred candidates" + "Accepted divergences"), or should `android-cli` be promoted inline with a status tag? **For Spec to resolve**.
- **Inline marker in modified SKILL.md**: should the first Claude-specific patch also carry an inline HTML comment marker (e.g., `<!-- CFA-PATCH: see HOW-TO-SYNC#accepted-divergences -->`) at the top of the vendored `android-cli/SKILL.md` for re-sync safety, or is the central HOW-TO-SYNC section sufficient? **For Spec to resolve**.
- **NOTICE file update**: upstream `android/skills` does not currently ship a NOTICE file. Confirm at implementation time whether the v0.0.2 tag (or dac bundle) has added one, and if so, copy verbatim per §4(d).
