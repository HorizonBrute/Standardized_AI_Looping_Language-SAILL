# SAILL Language Version History

> Version increments on merge to main. Update this file and VERSION.md before merging.

This document tracks language-level changes to SAILL (Standard AI Looping Language): primitives added, removed, or modified; syntax changes; and breaking changes. It does not track implementation changes (tooling, harness, skill scaffolding, etc.).

---

## 0.9 (pre-release)

**Date:** 2026-06-25
**Status:** Initial pre-release

First public pre-release of SAILL. Establishes the core primitive set and syntax conventions. First tested implementation validated and documented.

### Core primitives shipped

| Primitive | Description |
|-----------|-------------|
| `parallel` | Execute roles/steps concurrently |
| `wait` | Block until a prior parallel step completes |
| `Loop` | Repeat a step or block (with optional condition) |
| `if needed` | Conditional execution — run only when the acting model judges it adds value |
| `if asked` | Conditional execution — run only when explicitly requested by the caller |
| `ask user` | Pause flow and surface a question to the human |
| `-context:<value>-` | Runtime-bound named values injected into a flow at invocation; value in angle brackets names what is being sought (e.g., `-context:dirs-`) |
| `#lowcost`, `#midcost`, `#highcap`, `#investigate` | Model groups — named capability tiers mapped to actual model IDs via `model_prefs.md` |
| scope cascade | Inheritance of team definitions and model preferences from outer scope to inner; most-specific scope wins |
| `[ … ]` (nested boxes) | Sub-team / sub-flow blocks nested inside a parent flow; name a box to declare an inline sub-team |
| `/skill-name` invocation | A role's charter may invoke a named skill directly (e.g., `if fail /triage`) |

### Notes

- Syntax is intentionally terse: cusp of human- and machine-readable, modeled on SQL/ELK query style.
- No breaking changes (initial release).

---

## 1.0.0

**Date:** 2026-06-25
**Status:** Stable

First stable release of SAILL. No breaking changes from 0.9.

### Changes from 0.9

- No primitives added, removed, or modified.
- Primitive set, syntax, and model group conventions are unchanged.
- Documentation completed and indexed across all 13 sections.
- Hash integrity baseline established.

### Migration notes

No migration required from 0.9. All 0.9 team definitions are valid 1.0.0 definitions.
