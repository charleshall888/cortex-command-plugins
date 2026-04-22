---
schema_version: "1"
uuid: d2654183-a31e-471c-b984-aa3a033b386c
title: "Document plugin design intent + curation bar"
status: backlog
priority: high
type: chore
tags: [android-dev-extras-registry, docs]
parent: 1
created: 2026-04-22
updated: 2026-04-22
discovery_source: research/android-dev-extras-registry/research.md
---

## Value

Future curation decisions ("should we add skill X?") keep re-litigating the same design question because the plugin has no statement of its own motivation or curation bar. Codifying it once stops that drift and gives future curators (and reviewers) a principled anchor.

## Research context

The marketplace root `README.md` states the per-project-toggle motivation for the marketplace as a whole, but `plugins/android-dev-extras/` itself has no README and the commit message / `HOW-TO-SYNC.md` are silent on plugin-level motivation. The existing deferred-candidates list (`agp-9-upgrade`, `migrate-xml-views-to-jetpack-compose`, `navigation-3`, `play-billing-library-version-upgrade`) reveals an *implicit* curation bar — "non-destructive, non-version-coupled, cross-project-applicable" — that is visible only in retrospect.

See `research/android-dev-extras-registry/research.md` DR-3 and the "Skill-bar criteria" open question.

## What "done" might look like

A short motivation document (plugin-level README or an expanded preamble in HOW-TO-SYNC) that covers:

- Why this plugin exists as a separate per-project capsule rather than a global install doc.
- The curation bar: what gets in, what gets deferred, and on what principles.
- An invitation to interrogate those criteria rather than treat them as settled (the research flagged that they've never been explicitly examined).

Location and shape are open — consider which is easier to discover: a dedicated README vs. a section inside HOW-TO-SYNC.

## Out of scope

- Rewriting the curation bar itself beyond what the research already surfaced — this ticket codifies current intent, not a new policy.
- Changes to vendored skill contents.
