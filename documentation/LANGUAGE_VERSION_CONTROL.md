# SAILL Language Version History

> Version increments on merge to main. Update this file and VERSION.md before merging.

This document tracks language-level changes to SAILL (Standard AI Looping Language): primitives added, removed, or modified; syntax changes; and breaking changes. It does not track implementation changes (tooling, harness, skill scaffolding, etc.).

---

## 0.9 (pre-release)

**Status:** Initial language definition.

First public pre-release of SAILL. Establishes the core primitive set and syntax conventions.

### Core primitives shipped

| Primitive | Description |
|-----------|-------------|
| `parallel` | Execute roles/steps concurrently |
| `wait` | Block until a prior parallel step completes |
| `Loop` | Repeat a step or block (with optional condition) |
| `if needed` | Conditional execution — run only when the condition is required |
| `if asked` | Conditional execution — run only when explicitly requested by the caller |
| `ask user` | Pause flow and surface a question to the human |
| `-context- values` | Runtime-bound named values injected into a flow at invocation |
| `model groups` | Role-level model preference declarations |
| `scope cascade` | Inheritance of settings/context from outer scope to inner |
| `nested boxes` | Sub-team / sub-flow blocks nested inside a parent flow |

### Notes

- Syntax is intentionally terse: cusp of human- and machine-readable, modeled on SQL/ELK query style.
- No breaking changes (initial release).

---

## 1.0

_Placeholder — to be filled before 1.0 merge to main._

<!--
Sections to complete:
- Breaking changes (if any from 0.9)
- New primitives
- Modified primitives
- Removed primitives
- Migration notes
-->
