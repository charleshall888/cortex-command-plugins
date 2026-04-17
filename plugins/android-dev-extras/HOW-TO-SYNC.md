# HOW-TO-SYNC

How to refresh the skills vendored in this plugin against their upstream source.

## Upstream source

- Repository: [github.com/android/skills](https://github.com/android/skills) (Apache License 2.0).
- License: copied verbatim as `./LICENSE` in this plugin directory.
- Contributions: upstream does not accept public PRs. Curate locally; open issues upstream if you want changes.

## Path mapping (normative)

Upstream uses a category taxonomy; this plugin flattens it. Category prefix is dropped:

| Upstream | Vendored here |
|----------|---------------|
| `performance/r8-analyzer/SKILL.md` | `skills/r8-analyzer/SKILL.md` |
| `performance/r8-analyzer/references/**` | `skills/r8-analyzer/references/**` |
| `system/edge-to-edge/SKILL.md` | `skills/edge-to-edge/SKILL.md` |
| `<category>/<skill>/SKILL.md` | `skills/<skill>/SKILL.md` (for any future-added skill) |
| `<category>/<skill>/references/**` | `skills/<skill>/references/**` |

If two upstream skills ever collide on skill name after the category prefix is dropped, stop and resolve by keeping the category prefix in the vendored name (e.g., `skills/performance-x/` vs `skills/navigation-x/`). This has not occurred as of the initial vendoring (2026-04-17).

## Sync procedure

This is an AI-guided workflow, not an automated script. Next time you want to refresh:

1. Open a Claude Code session in this plugin directory.
2. Ask Claude: "sync android skills per HOW-TO-SYNC.md."
3. Claude should:
   - Fetch the upstream tree: `gh api "repos/android/skills/git/trees/main?recursive=1"`.
   - For each currently-vendored skill, fetch upstream SKILL.md and `references/**`, compare against vendored copy.
   - Surface a diff summary: unchanged / content-modified / references added/removed / skill removed upstream.
   - For new skills in upstream that are NOT vendored here: list them and ask whether to curate any in. Do not add them silently.
   - Update `LICENSE` if upstream `LICENSE.txt` has changed (verbatim copy).
   - Update `NOTICE` if the upstream copyright year or attribution has changed.
4. Run validation locally: `python3 scripts/validate-skill.py plugins/android-dev-extras/skills` (from the cortex-command-plugins repo root). Must exit 0 with no warnings.
5. Commit via the `/commit` skill. Never use raw `git commit` or `git -C`.
6. Push to GitHub remote — the marketplace source is `https://github.com/charleshall888/cortex-command-plugins.git`, so pushing is required for other project sessions to pick up changes.

## Apache 2.0 attribution obligation

Apache 2.0 § 4 requires distributing derivative works with:
- The original license text (→ `LICENSE` here).
- The original NOTICE file if one exists upstream (→ `NOTICE` here — authored when upstream ships no NOTICE; update verbatim if upstream adds one).
- Retention of copyright headers in modified files (only modify SKILL.md content if strictly necessary — prefer verbatim vendoring).

## Curation decisions worth revisiting

The initial vendoring included only `r8-analyzer` and `edge-to-edge` — both read-only and non-destructive. Deferred candidates:
- `build/agp/agp-9-upgrade` — destructive (modifies gradle files); only relevant when upgrading AGP major version.
- `jetpack-compose/migration/migrate-xml-views-to-jetpack-compose` — destructive, large cross-file edits.
- `navigation/navigation-3` — only relevant if actively migrating to Nav3.
- `play/play-billing-library-version-upgrade` — not applicable for CFA (no in-app billing).
- `system/edge-to-edge` — already included.
- `performance/r8-analyzer` — already included.

Add any of these by running steps 1–2 above and asking Claude to curate a specific skill in.
