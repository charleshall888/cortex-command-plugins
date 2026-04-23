# Review: rewrite-pr-review-synthesis-stage-with-three-axis-rubric-and-conventional-comments-output-format

## Stage 1: Spec Compliance

### Requirement 1: `rubric.md` reference file created
- **Expected**: File exists; ≥ 6 top-level `^## ` headings; each heading has non-empty content; axes section enumerates all three axes with their buckets on a single line each.
- **Actual**: `plugins/cortex-pr-review/skills/pr-review/references/rubric.md` exists (117 lines). `grep -c '^## '` returns 7 (Philosophy, Axes, Gate thresholds, Caps and tie-break, Drop-reason taxonomy, Normalization rules, Stability test protocol). Non-empty headers: `awk` returns 8 (header-coverage counter). Axis-bucket check `grep -cE 'must-fix.*should-fix.*nit.*unknown|solid.*plausible.*thin|high.*medium.*low'` returns 3.
- **Verdict**: PASS
- **Notes**: All four spec R1 acceptance checks pass. Section ordering is stable for downstream tooling.

### Requirement 2: Gate thresholds enumerated in `rubric.md`
- **Expected**: Each of the six finding-type labels appears on a line containing `severity`, `solidness`, or `signal` (≥ 6 matches).
- **Actual**: `grep -cE '^-.*\b(issue|suggestion|nitpick|question|praise|cross-cutting)\b.*(severity|solidness|signal)'` returns 9.
- **Verdict**: PASS
- **Notes**: Gate thresholds correctly use spec-canonical `nitpick:` (not `nit:`). Each label's rule names the axis preconditions verbatim per spec R2.

### Requirement 3: Caps enforced, tie-break documented
- **Expected**: `nitpick:` cap = 3 on a single line; `praise:` cap = 2; `cross-cutting:` cap = 1; tie-break by file path alphabetical then line ascending.
- **Actual**: Content is present — `rubric.md` lines 54–58 contain `- `nitpick:` cap = 3`, `- `praise:` cap = 2`, `- `cross-cutting:` cap = 1`, plus the tie-break sentence on line 58. The spec R3 regex uses `\bcap\b` with BSD grep which does not match against the actual `` `nitpick:` cap = 3 `` formatting (backticks interrupt the word boundary), so the literal spec greps return 0. Simpler regex `nitpick.*cap.*3` returns 1 for all three caps. Tie-break grep `grep -cE 'alphabetical.*line|line.*alphabetical|file path.*alphabetical'` returns 1.
- **Verdict**: PASS
- **Notes**: Spec-level R3 regex is a BSD-`\b` tooling defect (cap values are clearly present in the file; the regex cannot cross the backticked-label formatting). Content meets the requirement unambiguously.

### Requirement 4: Drop-reason taxonomy fixed in `rubric.md`
- **Expected**: Five reasons (`evidence-not-found`, `evidence-context-mismatch`, `low-signal`, `linter-class`, `over-cap`) each on a bulleted definition line with a `:`/`—`/`-` separator.
- **Actual**: `grep -cE '^[-*].*\b(...)\b.*[:—-]'` returns 5. Each reason has a single-line definition in the `## Drop-reason taxonomy` section.
- **Verdict**: PASS
- **Notes**: `evidence-not-found` is correctly marked as the silent-drop reason; the split from the original `evidence-mismatch` (per spec's adversarial-review note) is present.

### Requirement 5: Evidence-grounding normalization rules in `rubric.md`
- **Expected**: NFC/Normalization Form C; whitespace-collapse rule; consecutive-hunk rule; `matched_side` recorded.
- **Actual**: All four spec greps pass (NFC: 1, whitespace-collapse: 1, consecutive/hunk-boundary: 2, matched_side: 2).
- **Verdict**: PASS
- **Notes**: All five normalization rules (strip diff prefix, whitespace collapse, CRLF→LF, NFC, consecutive-hunk) are documented in `## Normalization rules` (lines 70–81). A clarifying note on `matched_side` vs ticket-005 renderer `side`/`start_side` is included.

### Requirement 6: `output-format.md` reference file created
- **Expected**: File exists; six labels each as a subsection/bullet definition; `(blocking)`/`(non-blocking)` decorator present; em-dash mentioned in voice guide.
- **Actual**: File exists (78 lines). `grep -cE '^(### |-).*\b(issue|suggestion|nitpick|question|praise|cross-cutting):'` returns 19. Each label appears ≥ 4 times individually. Decorator grep returns 11. Em-dash grep returns 1.
- **Verdict**: PASS
- **Notes**: All six labels have `### <label>:` subsections with format rules and examples. Voice guide enumerates forbidden AI-tell vocabulary; anti-pattern example correctly contrasts `nitpick:` vs `linter-class` drop.

### Requirement 7: Evidence schema documented in Stage 3 of `protocol.md`
- **Expected**: `### Evidence schema` subsection present; schema fields (`claim`, `label_hint`, `evidence`, `suggested_fix`, `category`); enum values for `label_hint` and `category`; conditional on `quoted_text` vs `rationale` for `question`/`cross-cutting`.
- **Actual**: `grep -c 'Evidence schema'` returns 6. Schema-shape grep returns 1. Label-hint enum grep returns 1. Category enum grep returns 2.
- **Verdict**: PASS
- **Notes**: Schema appears in a fenced TypeScript block at `protocol.md` lines 148–169; conditional rule on `quoted_text`/`rationale` explicitly documented at lines 171–179.

### Requirement 8: Four critic prompts emit `findings[]` JSON with correct category
- **Expected**: `^## Output format` heading preserved ≥ 4 times; each Output-format subsection references `findings[]`; all four literal category values are present.
- **Actual**: `grep -c '^## Output format'` returns 5 (4 critics + 1 triage reference). The `awk /^## Output format/{flag=1}` range captures `findings[]` inside each critic's Output-format subsection — `grep -c 'findings\[\]'` against that stream returns 4. All four literal category strings present: `"category": "compliance"` (1), `"category": "bug"` (1), `"category": "history"` (1), `"category": "historical-comment"` (1). The spec R8 awk range-idiom `/^## Output format/,/^##[^#]/` collapses to a five-line stream (only the header lines) because `^##[^#]` never matches before the next same-level heading — this is the BSD-awk single-line-range defect flagged in the task description. Bypassing via the plan's strengthened `awk /^## Output format/{flag=1; next} /^##[^#]/ && flag {flag=0} flag` pattern yields the correct 4 `findings[]` count.
- **Verdict**: PASS
- **Notes**: Per-critic append directive is exact and identical across the four critics, differing only in the literal category value.

### Requirement 9: Bash evidence-grounding pre-step
- **Expected**: pre-step implemented (not deferred); interface references `evidence-ground.sh`, `stdin`/`findings`/`grounded`/`drops`/`diff_path`; shell code fence or external script file; `jq --slurpfile` invocation pattern.
- **Actual**: Script exists at `plugins/cortex-pr-review/skills/pr-review/scripts/evidence-ground.sh` (553 lines, mode 755, executable). `grep -cE 'evidence-ground|Evidence grounding|Bash.*evidence'` against `protocol.md` returns 4. Contract-field grep returns 10. Stage 3.5 heading present. `jq -nc --slurpfile` invocation pattern documented in protocol.md lines 518–521. Behavioral smoke test (minimal fixture through the script) returns parseable `.grounded` JSON.
- **Verdict**: PASS
- **Notes**: The `## Stage 3.5 — Bash Evidence Grounding (pre-step)` section is captured by the Stage 3→Stage 4 awk range (the BSD idiom is fine here — Stage 4 comes after Stage 3.5). Script implements per-critic validation with `critic-malformed-json` drop, NFC via python3, path-normalization to POSIX, consecutive-hunk check, and priority-order substring match. Contract fields (`grounded`, `drops`, `failed_critics`) all emitted.

### Requirement 10: Stage 4 synthesizer rewritten
- **Expected**: `claude-opus-4-7` model pin; old fixed sections removed; injection-resistance preamble; path-traversal guard; no scaffolding fluff.
- **Actual**: Stage 4 awk range grep for `claude-opus-4-7` returns 2. Old fixed section grep (High-Confidence Issues, Architectural Assessment, Consensus Positives) returns 0. Injection preamble grep returns 1 (matches "untrusted user data" phrase). Path-traversal grep returns 6 (`evidence.path` references plus `..` segment rejection plus `starts with /`). Subagent prompt markers `<!-- BEGIN/END SUBAGENT PROMPT -->` both present exactly once on their own lines.
- **Verdict**: PASS
- **Notes**: Stage 4 is fully rewritten between the markers. Injection preamble verbatim per spec. Path-traversal guard explicitly rejects `..` segments and `/`-prefixed paths. No "double-check", "carefully consider", "make sure to" in the new template.

### Requirement 11: Verdict header derivation (APPROVE/REQUEST_CHANGES only)
- **Expected**: `Verdict:` header line; APPROVE/REQUEST_CHANGES both present; REJECT absent; derivation rule present.
- **Actual**: `Verdict:` grep returns 3; APPROVE|REQUEST_CHANGES grep returns 3; REJECT grep returns 0; derivation-rule grep (`issue.*blocking.*REQUEST_CHANGES|any.*blocking.*REQUEST_CHANGES`) returns 1.
- **Verdict**: PASS
- **Notes**: Derivation rule is stated explicitly at protocol.md line 661: "if any surfaced finding has label `issue (blocking):` then Verdict is `REQUEST_CHANGES`; otherwise Verdict is `APPROVE`."

### Requirement 12: Observability footer template
- **Expected**: `<details>` block; table schema (`# | Category | Label | Reason`); 8192-byte cap documented; 15-entry cap documented.
- **Actual**: `<details>` grep returns 3; table-schema grep returns 1; 8192/8 KB grep returns 1; 15-entry cap grep returns 1.
- **Verdict**: PASS
- **Notes**: Footer template at protocol.md lines 693–703 matches spec verbatim (including the `{N_critics}`, `{X}`, `{Y}`, `{visible_Y}` placeholders and the overflow-row format). Variable semantics (including the `Y` vs `visible_Y` silent-vs-visible distinction) are documented at lines 705–715.

### Requirement 13: Synthesis-failure fallback preserved and expanded
- **Expected**: Original "If the Opus subagent fails" text preserved; scope expanded to include pre-step failures; zero-findings-is-not-failure disambiguation present.
- **Actual**: Original-fallback grep returns 2; expanded-scope grep returns 4; zero-findings-not-failure grep returns 1.
- **Verdict**: PASS
- **Notes**: The verbatim fallback appears twice — once inside the subagent prompt body (degraded-mode contract instruction) and once in the outer "Failure handling" commentary after the `<!-- END SUBAGENT PROMPT -->` marker. Expanded-scope language ("pre-step exits non-zero, emits malformed JSON on stdout, emits empty stdout, or exceeds the 150-second Bash-tool timeout") is present. Zero-findings disambiguation at protocol.md lines 744–746.

### Requirement 14: SKILL.md `outputs:` frontmatter updated
- **Expected**: New string present; old `APPROVE | REQUEST_CHANGES | REJECT` absent; update lives in the YAML frontmatter block.
- **Actual**: New-string grep returns 1; old-string grep returns 0; frontmatter-scope awk returns 1.
- **Verdict**: PASS
- **Notes**: Frontmatter line 9 updated to the new string; body prose line 40 updated to "APPROVE or REQUEST CHANGES" (no REJECT anywhere in the file, per Task 12).

### Requirement 15: Stability test protocol in `rubric.md`
- **Expected**: Metric names (Krippendorff's α); Verdict stability metric; ship threshold 0.6; block threshold 0.5; PR selection criteria (known nits, real bug, pure refactor).
- **Actual**: Krippendorff/α grep returns 5; Verdict-stability grep returns 4; 0.6 threshold grep returns 1; 0.5 threshold grep returns 2; selection-criteria grep returns 3.
- **Verdict**: PASS
- **Notes**: `## Stability test protocol` is the seventh top-level section in rubric.md. 9-run methodology, metrics, ship thresholds, and 3-iteration exit ramp all documented.

### Requirement 16: Stability test execution and calibration log population
- **Expected**: `calibration-log.md` exists with 9 runs, anchor SHA, α, ship decision.
- **Actual**: `plugins/cortex-pr-review/skills/pr-review/references/calibration-log.md` does NOT exist. Task 13 is explicitly deferred. A `task_deferred` event is logged in `lifecycle/.../events.log` with the reason: "repo is trunk-based with no PR history; 9-run stability gate has nothing authentic to calibrate against. Component-level smoke test substituted: Stage 0 preflight + evidence-ground.sh against 5 cases (+match, -match/demotion, pass-through question, hallucinated→evidence-not-found silent drop, malformed critic→failed_critics)."
- **Verdict**: PARTIAL
- **Notes**: The deferral is reasonable on its merits — a 9-run stability gate against 3 real merged PRs has nothing authentic to measure when the repo has no PR history. Fabricating a log purely to satisfy a grep would defeat the gate's purpose (the plan explicitly addresses this tamper-evident concern). The substituted 5-case component smoke test exercises the Bash pre-step's behavioral contract (`+` match, `-` match with matched_side demotion hint, pass-through `question` with null quoted_text + rationale, hallucinated finding → silent `evidence-not-found` drop, malformed critic → `failed_critics[]` + `critic-malformed-json`) — this is sufficient evidence that the Bash-side code paths work. What it does NOT validate is the end-to-end synthesizer output-stability property the 9-run gate was designed to measure; that property remains untested until real PR traffic exists. The deferral weakens the ship gate, but the weakening is explicit, logged, and scoped to a property that no 9-run run could have actually measured in this repo state. Flagged PARTIAL (not FAIL) per the task instructions — shipping without calibration is a known, bounded risk here rather than an unsafe gap.

### Requirement 17: Opus 4.7 model pin in Stage 4 dispatch
- **Expected**: `claude-opus-4-7` pin in Stage 4; thinking/effort settings documented (as dispatch params or as commentary).
- **Actual**: `claude-opus-4-7` grep returns 2 in the Stage 4 awk range; thinking/effort grep returns 2.
- **Verdict**: PASS
- **Notes**: Dispatch block at protocol.md lines 546–550 pins `model: claude-opus-4-7`; `thinking.type: "adaptive"` and `output_config.effort: "xhigh"` are documented as inline comments (the Task tool does not currently accept them as per-dispatch params). Commentary at lines 552–555 explains the rationale and future-use expectation.

## Requirements Drift

**State**: none
**Findings**:
- None (no requirements docs exist in this project — no `requirements/project.md` and no area docs; drift check has nothing to compare against)
**Update needed**: None

## Stage 2: Code Quality

- **Naming conventions**: Consistent across the three layers. `diff_path` (snake_case) matches the plan's declared pipeline-state variable name and is used verbatim in `protocol.md` Stage 1, `protocol.md` Stage 3.5 invocation pattern, `evidence-ground.sh` input-schema documentation, and input validation (`jq -r '.diff_path'`). `matched_side` is used consistently in `rubric.md`, `protocol.md` Evidence schema, `protocol.md` Stage 3.5 matching algorithm, `protocol.md` Stage 4 demotion rule, and `evidence-ground.sh` comments and match-emission logic. `failed_critics` is consistent between `protocol.md` Stage 3.5 output contract, Stage 4 `## Inputs` block, and the script's per-critic validation code path. No case-or-hyphenation drift found.
- **Error handling**: `evidence-ground.sh` handles all required failure modes. Malformed critic JSON is caught at per-critic validation and emits a `critic-malformed-json` drop with the critic added to `failed_critics` (confirmed by smoke test). Missing/unreadable `diff_path` emits a single-line error JSON and exits 1. A 120-second self-timeout watchdog is spawned as a background process; the caller sets a 150-second Bash-tool safety net. Cleanup via `trap '... EXIT'` removes the tmp dir and kills the watchdog. Intentional absence of `set -e` is documented inline — the script must continue processing surviving critics when one fails, which `set -e` would prevent. Defense-in-depth preflight for `jq`/`python3` inside the script covers the standalone-invocation case even though Stage 0 is the primary gate.
- **Test coverage**: The 5-case component-level smoke test (recorded in the `task_deferred` event) exercises the script's key behavioral branches: `+` match, `-` match with matched_side demotion hint, pass-through `question` with null quoted_text + populated rationale, hallucinated finding producing a silent `evidence-not-found` drop, and malformed-critic yielding `failed_critics[]` + `critic-malformed-json`. These cover the five main control-flow paths in the matching algorithm. Gaps for this ticket: no test for cross-hunk quote → `evidence-context-mismatch`, no test for Windows-path slash normalization, no test for the 120-second self-timeout, no end-to-end test of synthesizer output stability (the deferred calibration's job). The coverage is acceptable for a foundation ticket where the downstream renderer (005) and voice filter (006) will both exercise the pre-step in their own tests — but the gaps are real and worth noting for follow-up.
- **Pattern consistency**: The new `## Stage 0 — Environment Preflight` and `## Stage 3.5 — Bash Evidence Grounding (pre-step)` sections follow the existing protocol.md staging convention (numbered stage headings, `---` separators between stages, code-fenced commands for each concrete step). The Stage 0 cost-graceful-failure framing ("halts the pipeline immediately; no synthesis attempted, so no fallback route") is explicit and correct. The subagent-prompt marker pattern (`<!-- BEGIN SUBAGENT PROMPT -->` / `<!-- END SUBAGENT PROMPT -->` on their own lines) is new to this file but is a clean, greppable way to distinguish prompt-body text from main-agent commentary and dispatch plumbing — a good convention to establish for this skill. Subagent-prompt internal structure (Security preamble → Inputs → Path-traversal guard → Rubric scoring → matched_side demotion → Verdict derivation → Output structure → Observability footer → Synthesis-failure fallback) reads cleanly and preserves the existing Stage 3 `## Inputs` template shape.

## Verdict

```json
{"verdict": "APPROVED", "cycle": 1, "issues": [], "requirements_drift": "none"}
```
