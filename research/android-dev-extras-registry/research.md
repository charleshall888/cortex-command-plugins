# Research: android-dev-extras registry adoption & design intent

## Research Questions

1. **Why does the `android-dev-extras` plugin exist at all — is per-project toggle the actual design intent, or is there a different reason?**
   → **Strongly supported, not explicitly stated.** Root `README.md` documents per-project toggling as the marketplace's intent ("these plugins package skills that were previously bundled with `cortex-command` but are useful independently **and can be adopted per-project**"). All four sibling plugins (`cortex-ui-extras`, `cortex-pr-review`, `cortex-dev-extras`, `android-dev-extras`) follow the same toggle-via-`enabledPlugins` structural pattern. However, `HOW-TO-SYNC.md`, the single-commit message, and the plugin's `plugin.json` are all silent on *plugin-level* motivation. The design intent is marketplace-level (README), not plugin-level — which is the documentation gap.

2. **What does the `android-cli` skill actually contain, and would adding it change the plugin's character?**
   → **It is Google-CLI-coupled and version-drifting.** 376 lines / ~2,420 words across `SKILL.md` + `references/interact.md` + `references/journeys.md`. Hard dependencies on `/usr/local/bin/android` and `adb shell input *`. Critically, `SKILL.md` lines 62–196 embed the literal `android help` output keyed to CLI version `0.7.15232955` — every CLI upgrade invalidates the vendored copy. New permission surface on downstream: `Bash(android:*)`, `Bash(adb shell input*)`. Self-contained otherwise (no external scripts/network fetches).

3. **How should HOW-TO-SYNC cover the two upstreams? Are they guaranteed to stay distinct or converging?**
   → **There are not really two upstreams — there is one merged tree.** `~/.android/cli/skills/.github/workflows/update-skills.yml` fetches `https://dl.google.com/dac/dac_skills.zip` (authoritative developer.android.com bundle) and overlays it with the `github-skills` branch to produce `main`. So: `dl.google.com/dac/dac_skills.zip` is authoritative; `github.com/android/skills` is a public mirror/overlay; `developer.android.com/tools/agents/android-skills/browse` is a curated 6-item subset (omits `android-cli`). `android-cli` is dac-only. The curator's original HOW-TO-SYNC referenced only `github.com/android/skills`, so they missed the dac bundle and missed `android-cli`.

4. **What Claude-specific adaptations would justify divergence from the verbatim-mirror policy?**
   → **Currently none are present; none have been attempted.** Verification confirms `r8-analyzer` and `edge-to-edge` SKILL.md files are byte-identical to upstream (`performance/r8-analyzer/SKILL.md` and `system/edge-to-edge/SKILL.md`), with zero references to `.claude/`, `/lifecycle`, hooks, or slash commands. Possible divergence candidates (not implemented): (a) trigger-phrase tuning to CFA vocabulary, (b) `/lifecycle` integration hooks, (c) detect-then-load guards (e.g., skip `android-cli` if `command -v android` fails). These would be real divergences that break re-sync cleanliness, so the cost (merge conflict on every refresh) must be weighed against the benefit.

5. **What else is in the full catalog and which skills are viable candidates?**
   → **Seven total via `android skills list`** (CLI view): `android-cli`, `navigation-3`, `edge-to-edge` (vendored), `play-billing-library-version-upgrade`, `r8-analyzer` (vendored), `migrate-xml-views-to-jetpack-compose`, `agp-9-upgrade`. Public browse lists 6 (everything except `android-cli`). Four are already in the HOW-TO-SYNC deferred list with reasons: `agp-9-upgrade` (destructive), `migrate-xml-views-to-jetpack-compose` (destructive), `navigation-3` (applicable only mid-migration), `play-billing-library-version-upgrade` (not CFA-applicable). `android-cli` is absent from the deferred list — genuine gap.

6. **Would adding `android-cli` cause surprise in downstream projects?**
   → **Only one downstream project detected *on this machine*** (`chickfila-android`) — its `.claude/settings.local.json` already permits `Bash(android --version)`, `Bash(android describe *)`, `Bash(android docs *)`, `Bash(android list *)`, `Bash(command -v android)`, and targeted `adb shell` commands. Low permission-prompt risk for the android-cli skill's common flows. New subcommands (`screen capture`, `screen resolve`, `emulator`, `run`) are NOT in the allow list and would prompt. Tooling dependency: the skill is useless without `/usr/local/bin/android` installed — activation on a machine without it teaches the agent to invoke a missing binary. **Caveat**: the marketplace repo is public (`github.com/charleshall888/cortex-command-plugins`), so external consumers who cloned it are invisible from this machine. Blast-radius reasoning here is local-scope only; external compatibility impact of any skill-set change is an open question.

   **Additionally verified** (post-critical-review): Claude Code does NOT auto-load skills from `~/.android/cli/skills/`. `~/.claude/settings.json` has no `skillsPaths` override, `~/.claude/skills/` has no symlink to the android CLI install location, and there is no other configured mechanism by which the CLI's installed skill directory becomes visible to Claude Code. The CLI installs skills for Google's agent interaction model (Gemini, Antigravity, etc.); for Claude Code, skills must come via a plugin, symlink, or explicit configuration. **This inverts an earlier assumption**: vendoring `android-cli` into this plugin is *not* redundant with `android init`. If a Claude Code user wants the skill, a plugin (or manual symlink) is the only reasonable path.

## Codebase Analysis

### Plugin shape (as vendored)
- `plugins/android-dev-extras/` contains: `.claude-plugin/plugin.json` (single line: `{"name": "android-dev-extras"}`), `LICENSE` (Apache 2.0 copy), `NOTICE` (plugin-authored attribution per § 4), `HOW-TO-SYNC.md`, `skills/r8-analyzer/` (SKILL.md + 6 reference files), `skills/edge-to-edge/` (SKILL.md only).
- No `plugin.json` fields beyond `name`. No hooks, no slash-command declarations, no lifecycle bindings. Skills are passively loaded by Claude Code when the plugin is enabled.

### Sibling plugin pattern (structural consistency)
All four plugins in the marketplace have identical shape: single-line `plugin.json`, optional `LICENSE`/`NOTICE`, a `skills/` directory, nothing else. The marketplace's `marketplace.json` registers each with the same metadata schema. Carve-out convention is **per-domain, per-project-togglable** — not grouped-by-technology.

### HOW-TO-SYNC current state
- Upstream declared: `https://github.com/android/skills` (single).
- Path mapping: flattens upstream category prefixes (`performance/r8-analyzer/` → `skills/r8-analyzer/`).
- Apache 2.0 compliance: LICENSE copy + plugin-authored NOTICE.
- Deferred candidates section: 4 skills with per-skill reasons.
- **Silent on**: the `dl.google.com/dac/dac_skills.zip` bundle, the `developer.android.com/tools/agents/android-skills/browse` curated list, the existence of `android-cli`, and the *motivation* for the plugin itself.

### Git history
- Single commit `b75a7e0` (Apr 17 2026). Message focuses on mechanics (vendoring, flattening, licensing). No author statement of motivation.
- No GitHub issues or PRs on the plugin.

### Downstream toggle surface
- Only `chickfila-android` project enables `android-dev-extras@cortex-command-plugins` via `.claude/settings.local.json`.
- Project permissions already permit the `android` CLI for read-only subcommands (`--version`, `describe`, `docs`, `list`, `command -v`) and targeted `adb shell pm list packages`.

## Web & Documentation Research

### Google's Android CLI for AI agents (April 2026)
- Announced 2026-04 at `android-developers.googleblog.com/2026/04/build-android-apps-3x-faster-using-any-agent.html`. Claims 70%+ token reduction and 3× faster task completion vs. raw `gradle`/`adb`.
- CLI subcommands: `create`, `run`, `sdk`, `emulator`, `screen`, `layout`, `docs`, `describe`, `info`, `init`, `update`, `skills`.
- Explicitly cross-agent: "Whether using Gemini in Android Studio, Gemini CLI, Antigravity, or third-party agents like Claude Code or Codex."
- `android skills` verbs: `add`, `remove`, `list`, `find`. No `show`/`describe`. On-disk package manager.
- Install path: `~/.android/cli/skills/` (confirmed populated via `android init`).

### Registry topology (authoritative finding)
- `~/.android/cli/skills/.github/workflows/update-skills.yml` merges `dl.google.com/dac/dac_skills.zip` with the `github-skills` branch of `android/skills` → `main`.
- Public browse at `developer.android.com/tools/agents/android-skills/browse` curates 6 user-facing skills (omits `android-cli`).
- `github.com/android/skills` README: *"Public contributions are not accepted at this time. Submit a GitHub issue…"* Curation is Google-internal.
- No convergence signal because there is no real divergence — one authoritative source (dac) with a public-mirror overlay.

### `android-cli` skill itself
- Lives at `~/.android/cli/skills/android-cli/`, missing from `developer.android.com/tools/agents/android-skills/browse`.
- Multi-file: `SKILL.md` (197 lines) + `references/interact.md` (82) + `references/journeys.md` (97).
- Frontmatter: `description` only, no explicit trigger phrases.
- Embeds literal `android help` output (lines 62–196) — version-coupled to CLI `0.7.15232955`.

## Domain & Prior Art

### Comparable marketplace plugins in this repo
- `cortex-ui-extras`: 6 skills, design-enforcement pipeline. Internal to cortex-command workflow; no upstream-mirror relationship.
- `cortex-pr-review`: 1 skill. Internal.
- `cortex-dev-extras`: 2 skills (`devils-advocate`, `skill-creator`). Internal.
- `android-dev-extras`: 2 skills, **only** plugin that vendors from a third-party upstream. It is the outlier and the only one for which a sync policy matters.

### Vendoring policy trade-offs (observed)
- **Verbatim mirror** (current policy): cheap sync, no divergence debt, but no Claude-specific tuning. Works when upstream is stable.
- **Fork with patches**: permits trigger tuning, `/lifecycle` integration, CFA-vocabulary rewrites — but every re-sync risks merge conflicts. Cost scales with number of patches.
- **Detect-and-load guards** (not yet applied): lightweight patch that gates skill activation on environmental preconditions (e.g., `command -v android`). For a single-file skill, low-cost patch; for version-coupled content (like `android-cli`), no amount of guarding fixes drift.

## Feasibility Assessment

| Approach | Effort | Risks | Prerequisites |
|---|---|---|---|
| **A: Update HOW-TO-SYNC to document both upstreams (dac + github)** | S | Low. Doc-only change. | Confirm via maintainers that dac URL is stable (workflow suggests yes). |
| **B: Add `android-cli` skill verbatim** | S (to add) / M (to maintain) | **High drift** — embedded CLI help output version-couples the vendored copy; every `android update` creates a sync obligation. Downstream projects without `/usr/local/bin/android` get a skill that teaches agents to invoke a missing binary. | `dl.google.com/dac/dac_skills.zip` as sync source; decide on detect-then-load guard. |
| **C: Skip `android-cli` permanently** | S | Low author cost, but note: Claude Code does **not** auto-load `~/.android/cli/skills/`, so the "CLI install = free skill" shortcut does not apply. Skipping means Claude Code users needing the skill rely on a manual symlink (worse ergonomics) or nothing. | Document rationale in HOW-TO-SYNC deferred list. |
| **D: Add `navigation-3` and `migrate-xml-views-to-jetpack-compose`** | S each | `migrate-xml-views-to-jetpack-compose` is flagged destructive by existing curation policy; `navigation-3` is only useful mid-migration. | Case-by-case per-project judgment; matches existing deferred-candidate philosophy. |
| **E: Document plugin design intent (one-paragraph addition)** | XS | None — pure documentation win. Codifies the implicit contract so future curators don't have to re-derive it. | None. |
| **F: Introduce Claude-specific adaptations (e.g., detect-then-load guard)** | M | Breaks verbatim-mirror purity. Every re-sync needs manual patch reapplication. Only justified if the skill would be broken without the guard. | Requires an explicit policy carve-out in HOW-TO-SYNC. |

## Decision Records

### DR-1: Vendoring `android-cli` — recommend **F (vendor with detect-then-load guard) pending refresh-cadence confirmation**
- **Context**: The missing-upstream discovery opened the question of whether to vendor `android-cli`. It is the only skill in the merged catalog that isn't already on the deferred list.
- **Earlier (now-retracted) assumption**: "CLI users get the skill for free via `android init`." Verified FALSE — Claude Code does not auto-load `~/.android/cli/skills/`; the CLI's install path serves Google's agent stack, not Claude Code. With that premise gone, "vendoring is redundant" collapses.
- **Options considered**:
  - **B** (vendor verbatim, no guard): simple, but a missing `/usr/local/bin/android` on a developer's machine makes the skill's advice unexecutable — the agent confidently invokes a binary that isn't there.
  - **C** (skip permanently): would leave Claude Code users without an official path to surface the CLI skill per-project; the only workaround is a manual symlink, which is worse ergonomics than the plugin.
  - **F** (vendor with detect-then-load guard — e.g., SKILL.md's trigger logic gates on `command -v android`): preserves per-project toggle, avoids activation on machines without the CLI. Cost: first real divergence from the verbatim-mirror policy.
- **Recommendation**: **F**, conditional on confirming `dl.google.com/dac/dac_skills.zip` has a predictable refresh cadence tied to CLI releases. If it does, vendoring is viable — refresh `android-cli` alongside CLI version bumps. If it doesn't, fall back to **C** until upstream stability is established.
- **Trade-offs**: (a) First Claude-specific patch on a vendored skill — sets precedent; HOW-TO-SYNC needs to document the guard as an accepted divergence category. (b) Refresh obligation: every material CLI release triggers a sync check, because embedded `android help` output (SKILL.md lines 62–196) is version-coupled. (c) Permission surface expansion on downstream: `Bash(android:*)`, `Bash(adb shell input*)` — already permitted in `chickfila-android` for read-only subcommands but will prompt for new write-style subcommands (`screen capture`, `emulator`, `run`).

### DR-2: Registry documentation — recommend **A (document both upstreams) with explicit dac-primary framing**
- **Context**: HOW-TO-SYNC points at `github.com/android/skills` only. The actual authoritative source is `dl.google.com/dac/dac_skills.zip`, merged with a GitHub branch overlay to produce the public repo. Future curators looking at HOW-TO-SYNC would miss `android-cli` (and any future dac-only skills) for the same reason the current curator did.
- **Options considered**:
  - **Status quo** (single-upstream doc): preserves simple mental model but is structurally wrong; the omission caused the current gap and will cause future gaps.
  - **A** (document both, primary = dac): accurate, but adds a two-source mental model.
  - **Restructure sync to pull directly from dac**: most principled — aligns the doc with the actual source of truth and drops the overlay middleman. Cost: loses GitHub's browsable diff view; re-zipping is less ergonomic than `git log`; and `dl.google.com/dac/dac_skills.zip` may be an internal publishing artifact whose URL is not contractually stable (see Open Question).
- **Recommendation**: **A**, with dac explicitly marked primary and GitHub called out as a convenience mirror for content that lives in both. Add a refresh-procedure check-step: `android skills list` output must match the union of HOW-TO-SYNC's covered + deferred lists — a mismatch means there's a new skill the curator hasn't decided on.
- **Trade-offs**: Doc grows by ~15 lines. Two-source mental model adds cognitive load, but downgrading to a GitHub-only doc is what caused the current gap. If `dl.google.com/dac/dac_skills.zip` is later confirmed stable by upstream, revisit and move to a dac-only doc at that point.

### DR-3: Design-intent documentation — recommend **E (document in plugin-level README)**
- **Context**: The per-project-toggle motivation is real and endorsed by the marketplace README, but the *plugin* has no README or motivation statement. Future changes (e.g., "should we add X?") re-open the same design question every time.
- **Options considered**: Add plugin-level README, embed motivation in HOW-TO-SYNC, leave implicit.
- **Recommendation**: **E** — add a short `plugins/android-dev-extras/README.md` (or a "Why this plugin exists" section in HOW-TO-SYNC) stating: (1) the plugin is a per-project capsule for Android-repo projects; (2) the curation bar is non-destructive, non-version-coupled, cross-project-applicable skills from the merged Android skills catalog; (3) skills deemed destructive, narrow, or CLI-version-coupled are explicitly deferred.
- **Trade-offs**: None material. Codifying the implicit contract is strictly additive and makes future curation decisions faster.

### DR-4: Claude-specific adaptations — recommend **hold**
- **Context**: None have been made. The question is whether to start.
- **Recommendation**: Do not adopt until a concrete driver appears. The verbatim-mirror policy is working. The only adaptation that has real demand pressure is the detect-then-load guard for CLI-coupled skills, and DR-1 handles that by declining to vendor `android-cli` at all.
- **Trade-offs**: Accepts that trigger phrases and descriptions will remain in Google's voice rather than CFA's. That's fine — skill description semantic match works without vocabulary tuning for these two skills.

### DR-5: `navigation-3` / `migrate-xml-views-to-jetpack-compose` — recommend **keep deferred**
- **Context**: Existing HOW-TO-SYNC curation policy already defers them. The discovery opened the question of whether to reconsider.
- **Recommendation**: Keep deferred. `migrate-xml-views-to-jetpack-compose` is explicitly destructive and violates the curation bar. `navigation-3` is only useful mid-migration, and the CFA Android app isn't actively migrating to Nav 3 (per user context).
- **Trade-offs**: If a future project adopts Nav 3, reconsider for that project specifically.

## Open Questions

- Is `dl.google.com/dac/dac_skills.zip` guaranteed stable, or is that URL internal to Google's publishing pipeline and subject to change? Worth asking upstream before rewriting HOW-TO-SYNC around it. (Low-risk open question — fallback is to stay on `github.com/android/skills` as working source and merely *note* the dac URL.)
- What is the refresh cadence of the dac bundle relative to CLI releases? `android-cli`'s embedded help output (SKILL.md 62–196) is pinned to a specific CLI version; if the bundle refreshes on every CLI release, vendoring cost is bounded. If not, DR-1 falls back to option C (skip).
- Should the refresh procedure add a CI-friendly check (e.g., a shell script that runs `android skills list` and diffs against the HOW-TO-SYNC deferred list)? This would turn the "missed `android-cli`" failure mode into a caught-by-automation one.
- Should the marketplace README gain a pointer to per-plugin README files so the toggle-motivation is discoverable from any plugin (not just the marketplace root)?
- **External consumers**: the marketplace repo is public (`github.com/charleshall888/cortex-command-plugins`). Anyone who installed it via `claude /plugin marketplace add` is an invisible consumer whose project state can't be observed from this machine. Open question for decomposition: does this plugin's blast-radius calculation need to treat `android-dev-extras` as a public API (changes require deprecation/communication) or as an author-owned curation surface (changes ship freely)?
- **Breaking-change response policy**: verbatim-mirror has no documented failure mode for the day upstream ships a breaking refactor of `r8-analyzer` or `edge-to-edge`. Should HOW-TO-SYNC codify a review gate (e.g., manually compare upstream diff before accepting refreshes)?
- **Skill-bar criteria**: DR-3 relies on "non-destructive, non-version-coupled, cross-project-applicable" as the curation bar, but this was inferred from existing deferrals, not explicitly codified. Should the plugin's design-intent doc interrogate whether these are the right criteria, or simply assert them?
