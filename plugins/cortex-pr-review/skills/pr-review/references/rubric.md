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
