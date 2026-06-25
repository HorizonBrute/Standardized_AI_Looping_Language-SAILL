# SAILL — Overview and Getting Started

**SAILL (Standard AI Looping Language)** is a compact, vendor-neutral notation for defining reusable multi-agent workflows. If SQL is a standard language for querying data, SAILL is a standard language for expressing how a team of AI agents runs.

---

## The Problem

When you want multiple AI agents to work on a task in sequence — one investigates, another fixes, a third validates — you have two options today:

1. Write a long natural-language prompt every time, explaining who does what, in what order, under what conditions.
2. Build bespoke tooling that hard-codes the workflow.

Both approaches are non-portable, expensive in tokens, and not shareable with others. Changing the workflow means rewriting the prompt or the code.

SAILL solves this by letting you define the workflow once, name it, and invoke it by name — or share the definition with anyone else using a compatible agent harness.

---

## What SAILL Is

SAILL has three layers:

**1. Agent Teams** — named, reusable workflows. A team is an ordered list of roles. Each role has a label, a model-capability group, an optional condition, and a one-line charter (what it does). You invoke a team by name in plain language:

> "Send the investigate-and-fix team at the auth bug."

The acting model looks up the team, spawns each role on its assigned model group, and chains their outputs.

**2. The SAILL language** — a small set of control-flow primitives that express how roles run: conditionally, in parallel, with loops, or with a human gate. These primitives appear inline in team definitions. They are terse by design — rich meaning in one or two words.

**3. Model Preferences** — a separate config layer that maps capability-group names (`#lowcost`, `#highcap`, etc.) to actual model IDs. Teams reference groups, not model IDs, so the same team definition works whether you're running on Anthropic, a local model, or a future provider.

---

## Key Concepts

| Concept | One-line description |
|---|---|
| Agent team | A named, ordered list of roles with model groups and control-flow flags |
| Role | A single unit of work: label + model group + charter (what it does) |
| Model group | A named capability tier (`#lowcost`, `#midcost`, `#highcap`, etc.) mapped to actual models |
| SAILL primitive | A control-flow keyword: `if needed`, `if asked`, `parallel`, `wait`, `Loop`, `ask user` |
| Scope cascade | Team and model definitions stack from a root directory down to the current folder; most-specific wins |
| Context loading | The mechanism by which `CLAUDE.md` and `agents.md` files are assembled into a session's system prompt |

---

## Hello World — Simplest Agent Team Setup

Three files in one directory:

**CLAUDE.md**
```
@agent_teams_flags.md
@agent_teams.md
"Send an agent team" resolves through the Agent Teams framework defined in agent_teams.md. Named variants select the matching team by name.
@model_prefs.md
```

**agent_teams.md** — defines one team:
```markdown
### Investigate & Fix
Diagnose a problem then apply the fix.

1. Investigate (#midcost) — diagnoses root cause; hands a precise diagnosis to Fix.
2. Fix (#lowcost) — applies the change and verifies the issue is resolved.
```

**model_prefs.md** — declares capability groups (members filled in `model_prefs.local.md`):
```markdown
### #lowcost
### #midcost
### #highcap
```

With these three files in place, you can say:

> "Send an investigate-and-fix team at the failing login test."

The acting model spawns an Investigator on a `#midcost` model, then a Fixer on a `#lowcost` model, chaining the diagnosis through.

> **Note (June 2026):** Claude Code's harness resolves `@-imports` only from `CLAUDE.md` files, not from `agents.md`. Put your `@-imports` directly in `CLAUDE.md` for reliable loading. See [How it Works](../How%20it%20Works/how_it_works.md) for details.

---

## Where to Go Next

| Topic | File |
|---|---|
| How context loads, @-imports, scope stacking | [How it Works](../How%20it%20Works/how_it_works.md) |
| Defining and invoking agent teams | [Agent Groups](../Agent%20Groups/agent_groups.md) |
| The full SAILL primitive set, boxes, loops, -context- | [SAILL Language Guide](../SAILL%20Language%20Guide/saill_guide.md) |
| Model groups, routing, per-session slots | [Model Preferences](../Model%20Preferences/model_preferences.md) |
| Keeping context overhead low | [Evaluating Context Cost](../Evaluating%20Context%20Cost/context_cost.md) |
| Working single-folder example | [Tested Implementation 1](../Tested%20Implementation%201/impl1.md) |
| Multi-folder inheritance example | [Tested Implementation 2](../Tested%20Implementation%202/impl2.md) |
| Environment variable paths example | [Tested Implementation 3](../Tested%20Implementation%203/impl3.md) |
