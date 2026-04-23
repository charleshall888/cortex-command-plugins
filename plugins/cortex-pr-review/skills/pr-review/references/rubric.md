# PR Review Rubric

Scoring and gating rules for the pr-review synthesizer. Apply every finding against the three axes below, then route it through the gate thresholds to decide whether it survives and which Conventional Comments label it takes.

## Philosophy

- Findings are grounded by evidence or they are not findings — there is no "observation" category. If a claim cannot cite a concrete diff hunk, file path, or behavior trace, drop it. Affirmed by the `/ultrareview` verification philosophy: speculation without grounding is noise.

## Axes

Score every candidate finding on three orthogonal axes. All three must be assigned before the finding reaches the gate.

### severity

Buckets: `must-fix | should-fix | nit | unknown`

- `must-fix` — correctness, security, data loss, or contract break. Merging as-is ships a defect.
- `should-fix` — quality, maintainability, or clarity regression that a reasonable reviewer would block on in most teams.
- `nit` — style, taste, or micro-optimization. Ignorable without harm.
- `unknown` — the reviewer cannot classify severity without author input (ambiguous intent, missing context). Routes to `question:`, never to `issue:`.

### solidness

Buckets: `solid | plausible | thin`

- `solid` — evidence is unambiguous AND the finding names a concrete next action (specific edit, file, or behavior change). `solid` is the only bucket that carries an actionable fix; a finding without a concrete next action cannot be `solid` no matter how strong the evidence.
- `plausible` — evidence supports the finding but the fix shape is uncertain, or the finding is a well-posed question rather than a prescription.
- `thin` — evidence is weak, indirect, or relies on pattern-matching without a grounded quote. Drop unless re-grounded.

### signal

Buckets: `high | medium | low`

- `high` — the finding changes the reviewer's decision or the author's next commit. Worth the reader's attention even in a dense review.
- `medium` — useful context or a real improvement, but would not block merge on its own.
- `low` — true but trivial; the reader gains little by seeing it called out.

## Gate thresholds

One rule per Conventional Comments label. A finding that does not match any rule below is dropped.

- `issue (blocking):` — severity = `must-fix` AND solidness ≥ `plausible`. Blocking by construction.
- `suggestion:` — severity = `should-fix` AND solidness = `solid` AND signal ≥ `medium`. Non-blocking by default.
- `nitpick (non-blocking):` — severity = `nit` AND solidness = `solid` AND signal = `high`. Spec-canonical label is `nitpick:`, never `nit:`.
- `question:` — severity = `unknown` AND solidness ≥ `plausible`. Never blocking, never paired with a prescriptive fix.
- `praise:` — solidness = `solid` AND signal = `high`. Orthogonal to severity; praise is earned by strong evidence and real impact, not by tone.
- `cross-cutting:` — solidness ≥ `plausible` AND signal = `high`. Bypasses locality (may reference multiple files or the PR as a whole).
- Everything else → drop.

## Caps and tie-break

Per-label ceilings on the number of findings that may survive after gating. Apply caps after the gate thresholds; findings beyond the cap are dropped with reason `over-cap`.

- `nitpick:` cap = 3
- `praise:` cap = 2
- `cross-cutting:` cap = 1

Tie-break for `nitpick:` overflow: sort surviving candidates by file path alphabetical, then by line ascending; keep the first three. The same tie-break applies to any label where more candidates survive than the cap allows.

## Drop-reason taxonomy

Every dropped finding carries exactly one reason from the list below. Reasons determine whether the drop is silent or visible in the footer.

- `evidence-not-found` — silent drop (hallucinated evidence); not shown in footer. The `quoted_text` does not match any line in the diff on any side.
- `evidence-context-mismatch` — visible drop; the quoted text is present in the diff but on the wrong side (e.g., quoted as added but only exists on the `-` side) or the multi-line quote spans a hunk boundary.
- `low-signal` — rubric-gate drop; the finding fails a threshold such as a nit without `signal = high`, or a suggestion without `signal ≥ medium`.
- `linter-class` — style, formatting, or linter-enforced issue filtered by design so the review does not duplicate tooling output.
- `over-cap` — exceeded the per-label cap after tie-break; the finding was otherwise eligible.

## Normalization rules

Evidence-grounding normalization applied before matching `quoted_text` against diff lines. Stage 4's synthesizer relies on these rules to compare against the evidence-grounding pre-step output.

- Apply NFC (Normalization Form C) Unicode normalization to both the `quoted_text` and each candidate diff line before comparison.
- Strip the leading diff prefix `^[+\- ]` from each diff line (the `+`, `-`, or space marker) before comparison.
- Normalize `\r\n` line endings to `\n` (CRLF → LF) in both inputs.
- Collapse whitespace runs (spaces, tabs) to a single space in both inputs before comparison; this whitespace-collapse step runs after CRLF normalization.
- For a multi-line `quoted_text`, split on `\n` and require each line to match a consecutive diff line within the same hunk. Reject cross-hunk quotes (quotes that would span a hunk boundary) with `evidence-context-mismatch`.
- Record the matched side (`+`, `-`, or ` `) in `evidence.matched_side` on the finding.

Note: `matched_side` is a rubric-input diagnostic consumed by the Stage 4 synthesizer to decide demotion (e.g., a `-` side match typically demotes an `issue:` to a `question:` or drops it). It is distinct from ticket 005's renderer `side` / `start_side` fields, which map to GitHub's Reviews API request payload.
