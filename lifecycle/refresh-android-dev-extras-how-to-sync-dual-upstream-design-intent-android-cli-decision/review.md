# Review: refresh-android-dev-extras-how-to-sync-dual-upstream-design-intent-android-cli-decision

## Stage 1: Spec Compliance

### Requirement 1: Design-intent preamble
- **Expected**: A `## Why this plugin exists` section added after the opening one-sentence intro and before `## Upstream source`, naming the per-project-toggle motivation, the three-part curation bar (non-destructive, non-version-coupled, cross-project-applicable), and referencing the marketplace-level README.
- **Actual**: Section present at HOW-TO-SYNC.md line 5. Contains "per-project-toggle" (twice), "curation bar", "non-destructive", "non-version-coupled", "cross-project-applicable", and "marketplace README at the repo root." Precedes `## Upstream source` at line 11.
- **Verdict**: PASS
- **Notes**: All three ACs verified. `grep -c "^## Why this plugin exists"` = 1; `grep -Ec "per-project[ -]toggle|per-project-togglable"` = 2; `grep -Ec "non-destructive|curation bar"` = 2.

---

### Requirement 2: Dual-upstream rewrite
- **Expected**: `## Upstream source` names `dl.google.com/dac/dac_skills.zip` as authoritative primary and `github.com/android/skills` as convenience mirror; notes URL is undocumented; provides fallback instruction.
- **Actual**: All four ACs pass. `dl.google.com/dac/dac_skills.zip` appears twice (once in Upstream source, once in Accepted divergences). `github.com/android/skills` appears three times. The Upstream source body contains "Authoritative primary" and "source of truth"; "undocumented by Google" is explicit; fallback to `github.com/android/skills/archive/refs/tags/v*.*.*.zip` is documented.
- **Verdict**: PASS
- **Notes**: Corrected awk pattern used — `awk '/^## Upstream source$/{flag=1; next} /^## /{flag=0} flag'` confirms "authoritative" appears in the section body.

---

### Requirement 3: Reconciliation check-step in Sync procedure
- **Expected**: New step between step 3 and step 4 running `android skills list`, reconciling against `### Covered skills` + `### Deferred candidates`, requiring curation decision for any unlisted skill.
- **Actual**: Step 4 of the sync procedure reads `Run `android skills list` and reconcile its output against the union of `### Covered skills` + `### Deferred candidates`. Any skill in the CLI output that is absent from both lists surfaces a new curation decision — the curator must decide whether to cover or defer before completing sync; do NOT silently skip unlisted skills.` The awk-scoped AC passes (`awk '/^## Sync procedure$/{flag=1; next} /^## /{flag=0} flag'` finds `android skills list`). `grep -Ei "reconcile|reconciliation|unlisted skill|new skill"` matches.
- **Verdict**: PASS
- **Notes**: The step was inserted as step 4 (validation moved to step 5). The spec's "between step 3 and step 4" is satisfied by the relative ordering. All three ACs verified.

---

### Requirement 4: Curation section restructured
- **Expected**: Three `###` subsections in normative order: `### Covered skills`, `### Accepted divergences`, `### Deferred candidates`. All four existing deferred candidates preserved.
- **Actual**: All three subsections present at lines 62, 68, 76. Subsection order verified against normative sequence. Deferred candidates count = 4 (`agp-9-upgrade`, `migrate-xml-views-to-jetpack-compose`, `navigation-3`, `play-billing-library-version-upgrade`).
- **Verdict**: PASS
- **Notes**: `grep -c` for each heading = 1. Deferred candidates AC: `awk '/^### Deferred candidates$/,0'` shows all four names, count = 4.

---

### Requirement 5: android-cli skill vendored
- **Expected**: Three files created under `plugins/android-dev-extras/skills/android-cli/`. Content byte-identical to source except for Req 6 guard block and Req 7 HTML comment marker. Source evaluated against local `~/.android/cli/skills/android-cli/` per scope_override event.
- **Actual**: All three files exist. `diff` of `references/interact.md` and `references/journeys.md` against local source produces no output (byte-identical). `diff` of `SKILL.md` shows only the 7-line CFA-PATCH block insertion (the blank line, HTML comment, blank line, guard command, blank line, precondition-check prose, blank line). No other content differences.
- **Verdict**: PASS
- **Notes**: Scope override correctly applied — evaluated against local source, not GitHub v0.0.2. The SKILL.md diff is exactly the guard+marker block specified in Reqs 6 and 7 with no extraneous changes.

---

### Requirement 6: Detect-then-load guard (dynamic context injection)
- **Expected**: `!`command -v android || echo NOT_INSTALLED`` preamble block near top of vendored SKILL.md (after frontmatter, before body); prose instruction telling Claude to abort if `NOT_INSTALLED` is injected; guard is advisory; `NOT_INSTALLED` appears ≥ 2 times; guard confirmed in frontmatter→first-heading window.
- **Actual**: Line 8: `` !`command -v android || echo NOT_INSTALLED` ``. Line 10: `**Preconditions check**: if the shell-injected output above reads `NOT_INSTALLED`, abort every `android`/`adb` flow in this skill`. `grep -c NOT_INSTALLED SKILL.md` = 2. Awk window check (second `---` to first `#`) confirms the guard line is in the correct position.
- **Verdict**: PASS
- **Notes**: `guard_verification` event selected `dynamic_injection` via Claude Code docs read. The prose instruction names `NOT_INSTALLED` explicitly and instructs abort behavior. Position AC (`awk '/^---$/{n++; if(n==2){found=1; next}} found && /^#/{exit} found{print}'` | `grep -F "command -v android"`) passes (exit 0).

---

### Requirement 7: Inline HTML comment marker
- **Expected**: `<!-- CFA-PATCH: see plugins/android-dev-extras/HOW-TO-SYNC.md §Accepted divergences -->` at the top of SKILL.md (immediately after closing `---` of frontmatter, before the guard block); marker mentions HOW-TO-SYNC and "Accepted divergences".
- **Actual**: SKILL.md line 6: `<!-- CFA-PATCH: see plugins/android-dev-extras/HOW-TO-SYNC.md §Accepted divergences -->`. `head -12 SKILL.md | grep -c "CFA-PATCH"` = 1. `grep -F "HOW-TO-SYNC" | grep -F "Accepted divergences"` matches the marker line.
- **Verdict**: PASS
- **Notes**: Marker is positioned on the first line of the body (line 6), immediately after the closing `---` of frontmatter (line 4), before the guard on line 8. Both ACs verified.

---

### Requirement 8: Accepted-divergences entry for android-cli
- **Expected**: Entry names `android-cli`, describes guard mechanism, cites rationale (per-project toggle; first Claude-specific patch), instructs re-sync behavior (reapply guard + marker; check CFA-PATCH marker is still present and positioned).
- **Actual**: The `### Accepted divergences` section contains a rich multi-bullet entry: names `skills/android-cli/SKILL.md`, describes source-channel divergence (local CLI install, CLI version 0.7.15232955, `android init` re-sync), describes content patch with `command -v android || echo NOT_INSTALLED` dynamic injection, cites rationale (per-project toggle; advisory guard; first accepted divergence), carries placement rule (guard sits between closing frontmatter `---` and first `#` heading; do NOT paste at literal byte offsets), and post-update check (verify `CFA-PATCH` marker present and positioned in frontmatter→first-heading window).
- **Verdict**: PASS
- **Notes**: All three spec ACs verified. `awk`-scoped search finds `android-cli` (count ≥ 1), `command -v android` (line match), and `CFA-PATCH` (line match). Placement rule is durable and self-contained — a future curator can execute re-sync without plan.md access.

---

### Requirement 9: android-cli listed under `### Covered skills`
- **Expected**: `android-cli` listed alongside `r8-analyzer` and `edge-to-edge` with a note pointing to `### Accepted divergences`.
- **Actual**: `- \`android-cli\` — interacts with Android devices and emulators via the \`android\` CLI; see \`### Accepted divergences\` for source and guard patch.` Both `r8-analyzer` and `edge-to-edge` entries retained.
- **Verdict**: PASS
- **Notes**: `awk`-scoped count for `android-cli` = 1; `r8-analyzer|edge-to-edge` count = 2. Cross-reference to `### Accepted divergences` present and specific.

---

### Requirement 10: Plugin validation exits 0
- **Expected**: `python3 scripts/validate-skill.py plugins/android-dev-extras/skills` exits 0; no `^error:` lines for `android-cli`.
- **Actual**: Validator output: `[OK] android-cli: name and description present`, `[OK] edge-to-edge: name and description present`, `[OK] r8-analyzer: name and description present`. Summary: `3 skills: 0 errors, 0 warnings, 0 infos, 3 clean`. Exit code = 0. `grep -E '^error:.*android-cli'` exits 1 (no matching lines).
- **Verdict**: PASS
- **Notes**: Both ACs verified. android-cli produces no warnings — cleaner than a "match existing patterns" outcome, since the existing skills also produce zero warnings.

---

### Requirement 11: Existing skills unmodified
- **Expected**: `r8-analyzer/` and `edge-to-edge/` contents unchanged from pre-change state.
- **Actual**: `git diff HEAD -- plugins/android-dev-extras/skills/r8-analyzer plugins/android-dev-extras/skills/edge-to-edge` produces zero bytes of output. `git diff f81b0ca -- ...` (against the pre-implementation baseline commit) also produces zero bytes.
- **Verdict**: PASS
- **Notes**: Verified against both current HEAD and the pre-plan baseline commit `f81b0ca`.

---

### Requirement 12: Divergence preservation in sync procedure
- **Expected**: Step 3 branches on `### Accepted divergences` (snapshot before overwrite; reapply after; cross-references section by name). Post-update safety check verifies `CFA-PATCH` marker is present AND positioned inside the frontmatter→first-heading window; absence or mispositioning fails the sync.
- **Actual**: Step 3 contains two bold sub-bullets: **Divergence preservation** (snapshot BEFORE overwrite, reapply per entry's placement rule at position relative to post-pull boundaries NOT literal byte offsets; `### Accepted divergences` is the authoritative re-sync contract) and **Post-update safety check** (verify `CFA-PATCH` marker present AND positioned inside the frontmatter→first-heading window; absent OR present-but-mispositioned fails the sync). Section is cross-referenced by name ("### Accepted divergences") three times within the Sync procedure section.
- **Verdict**: PASS
- **Notes**: All three ACs verified. `grep -Ec "Accepted divergences|### Accepted"` = 3 within the section; `grep -Ei "reapply|preserve|snapshot"` matches "snapshot", "reapply", and "preserve" (three separate matches); `grep -F "CFA-PATCH"` matches two lines in the section. The post-update check explicitly requires position verification, not just presence — satisfying the "prominent notice" durability requirement.

---

## Stage 2: Code Quality

All 12 requirements pass. Stage 2 proceeds.

- **Naming conventions**: android-cli, edge-to-edge, and r8-analyzer all follow the same kebab-case directory pattern under `plugins/android-dev-extras/skills/`. The new skill directory (`android-cli/`) and its reference files (`references/interact.md`, `references/journeys.md`) match the existing layout exactly. No deviations.

- **HOW-TO-SYNC structure**: The section ordering flows sensibly: motivation preamble (`## Why this plugin exists`) → dual upstream definition (`## Upstream source`) → path mapping (normative) → sync procedure → Apache attribution → curation inventory. The preamble-first ordering means a new curator encounters motivation before operational mechanics, which matches the doc's stated purpose. The three-subsection curation inventory (Covered → Accepted divergences → Deferred) is ordered logically by disposition type.

- **Accepted divergences entry context**: The entry is self-contained for a future curator operating without plan.md. It documents: (1) why android-cli isn't on the public upstreams; (2) the CLI version at time of vendoring; (3) exactly what the guard does and why; (4) the placement rule with enough specificity to apply it mechanically (between closing frontmatter `---` and first `#` heading, not at literal byte offsets); and (5) the post-update verification step. The source-channel divergence (CLI-bundled, re-sync via `android init`) is noted explicitly alongside the content-patch divergence, which is notable given the scope_override — both divergence types are documented without requiring the reader to consult the events log.

- **Sync procedure step 3 executability**: The instructions are precise enough for unambiguous execution. "Snapshot BEFORE the verbatim overwrite" and "reapply per the entry's placement rule" are specific operation sequences. The cross-reference to `### Accepted divergences` by name allows the curator to find placement rules without searching the file. The distinction between DAC/GitHub-sourced skills (using `gh api`) and CLI-bundled skills (using `android init`) is called out inline, preventing a missed re-sync for android-cli on a future refresh.

- **Guard block quality**: The `!`command -v android || echo NOT_INSTALLED`` form matches the spec's required literal exactly. The prose abort instruction is clear and immediate — it names the marker string (`NOT_INSTALLED`), names the scope of prohibition (`every android/adb flow`), and tells Claude what to do instead (`surface the missing binary to the user and stop`). The guard is placed optimally in the frontmatter→first-heading window, so it is visible before any substantive skill content.

- **Scope override handling**: The scope_override (android-cli sourced from local install rather than GitHub v0.0.2) is transparently documented in both the Accepted divergences entry and the sync procedure. There is no silent deviation; the non-reproducibility trade-off is documented with the CLI version (0.7.15232955) and the re-sync instruction (`android init`).

---

## Requirements Drift

**State**: none
**Findings**:
- None
**Update needed**: None

---

## Verdict

```json
{"verdict": "APPROVED", "cycle": 1, "issues": [], "requirements_drift": "none"}
```
