# SAILL Language Guide

SAILL (Standard AI Looping Language) is a small, terse, vendor-neutral vocabulary for expressing how a team of agents runs. It is the notation layer inside agent team definitions — the part that says not just who is in the team, but when and how each role executes.

---

## Why a Language, Not Just Prose

A natural-language team definition like "run the Investigator, then have the Fixer apply the findings, and if validation fails try again up to three times" works. But it is verbose, ambiguous across models, and not shareable in a standard way.

SAILL is the compact version of that. One team definition reads the same everywhere. A human, the acting model, and tooling all understand it identically. It is to agent workflows what SQL is to data queries — a narrow, regular notation that expresses rich meaning in very little space.

> SAILL is the compact notation. Natural language is always its lossless fallback translation.

---

## The Primitive Set

SAILL is a **closed set of atomic primitives**. Each primitive does exactly one thing. Expressive power comes from composing them, not from adding bespoke flags for every use case.

| Primitive | Form | Meaning |
|---|---|---|
| `if needed` | inline | Run only if the acting model judges it adds value; else skip |
| `if asked` | inline | Run only when the user explicitly requests it; else skip |
| `ask user` | inline | Pause and wait for user input, decision, or approval before continuing |
| `parallel` | inline | Run concurrently with adjacent `parallel`-flagged roles |
| `wait` | inline | Sync point: wait for the preceding parallel group to finish before continuing |
| `Loop` | annotation | Re-run an earlier role with feedback until a pass condition or iteration cap |

**Inline primitives** appear inside a role's model group parenthetical:
```
Log-reader (#lowcost, if needed)
Explorer (#investigate, parallel)
```

**Annotation primitives** introduce their own clause on the following line:
```
Validator (#midcost)
**Loop:** on failure, return feedback to "Implementer" and re-run from there;
repeat until clean or 3 iterations, then report failures.
```

---

## Naming Rules for Primitives

A SAILL primitive:

1. Reads as English in place — `Validator (#midcost, if asked)` reads naturally
2. Is one or two words. Lowercase for inline modifiers (`if needed`, `parallel`, `wait`); Capitalized for annotation primitives that introduce a clause (`Loop`)
3. Names the behavior, not the mechanism
4. Is self-translating — a human renders it to plain English without opening the registry
5. Is stable and distinct — never collides with another primitive or a `#model-group` name

Compose existing primitives before inventing new ones. Add a new primitive only for a genuinely new control-flow idea — never to encode one team's specific work logic.

---

## Sub-teams and Nesting — Boxes

A **box** `[ … ]` bundles one or more roles (or other boxes) into a single node. Boxes set the order of operations and group related work.

Name a box to declare an inline, ephemeral sub-team:
```
Recon[ APICrawler (#investigate, parallel), DBReader (#investigate, parallel) ]
```

All SAILL flags apply inside a box — no new operators. Boxes nest without limit.

**Example 1 — Parallel recon, then plan:**
```
Orchestrator (#highcap) ->
Recon[ APICrawler (#investigate, parallel), DBReader (#investigate, parallel) ] (wait) ->
Planner (#highcap)
```

**Example 2 — Gated build with an implement/validate loop:**
```
Planner (#highcap, ask user) ->
[ Implementer (#lowcost), Validator (#midcost)
  **Loop:** to "Implementer" until pass or 3 ]
```

**Example 3 — Two sub-teams looping in parallel, then integrate:**
```
[ Frontend[ Impl (#lowcost), Val (#midcost) Loop: to "Impl" until pass or 3 ] (parallel),
  Backend[  Impl (#lowcost), Val (#midcost) Loop: to "Impl" until pass or 3 ] (parallel) ]
(wait) -> Integrator (#midcost)
```

---

## Context Values — `-context-`

Wherever a SAILL parameter would take a literal — a loop's pass condition, a cap, a scope, a target — write `-context-` to mean "resolve this from the current context": the user's invocation, the conversation, or runtime state.

Qualify the placeholder:
```
-context:pass criteria-
-context:cap-
-context:source data-
```

This keeps team definitions generic and reusable. The structure is fixed; the specifics arrive at run time.

**Examples:**

Loop until a context-defined pass condition, capped at a literal 3:
```
[ Implementer (#lowcost), Validator (#midcost)
  Loop: to "Implementer" until -context:pass criteria- or 3 ]
```

Both pass and cap from context:
```
Loop: to "Implementer" until -context:pass- or -context:cap-
```

Scope a role from context:
```
Investigator (#midcost) — audit -context:source data-
```

---

## Failure Handlers, Skill Calls, and Compound Loop Exits

### Failure handlers — `if fail <action>`

A failure handler on a role or box runs `<action>` if the role/box fails or a loop hits its cap unmet, instead of stopping silently.

```
Build[ Implementer (#lowcost), Validator (#midcost) ] if fail /build_fail_triage_report
```

`<action>` is typically a skill invocation.

### Skill calls

A role's charter may invoke a named skill by its slash name. The skill is the role's work (charter), not a control-flow flag:

```
Security-Reviewer (#highcap) — run /security-review on -context:source data-
```

### Compound loop exits

A Loop may list several `or`-separated exit conditions — a pass, `ask user` (user can approve or stop early), and a cap:

```
Security-Reviewer (#highcap) — audit -context:source data-;
  on findings Loop: to "Build" until clean or ask user or -context:cap-
```

---

## Worked Examples

### Minimal — two-role sequential team
```
### Investigate & Fix

1. Investigate (#midcost) — diagnoses root cause; hands diagnosis to Fix.
2. Fix (#lowcost) — applies the change and verifies resolution.
```

### With conditional and gate
```
### Full Team

1. Orchestrator (#highcap) — breaks the task down and coordinates the rest.
2. Log-reader (#lowcost, if needed) — gathers runtime evidence before planning.
3. Planner (#highcap, ask user) — designs the approach; presents plan and waits for approval.
4. Implementer (#lowcost) — writes the code.
5. Validator (#midcost, if asked) — verifies work and checks for regressions.
   **Loop:** on failure, return feedback to "Implementer" and re-run from there;
   repeat until clean or 3 iterations, then report outstanding failures.
```

### Parallel exploration, then synthesize
```
### Deep Explore

1. Orchestrator (#highcap) — defines scope and fans out.
2. Recon[ APICrawler (#investigate, parallel), DBReader (#investigate, parallel),
          UITracer (#investigate, parallel) ] (wait)
3. Synthesizer (#midcost) — integrates findings into a single report.
```

### Loop with failure handler
```
### Build & Certify

1. Builder (#lowcost) — builds the artifact.
2. Certifier (#midcost) — runs the certification suite.
   **Loop:** on failure, return findings to "Builder" and re-run from there;
   repeat until certified or -context:cap- iterations, then report failures.
   if fail /escalate_cert_failure
```

---

## Extending the Vocabulary

The shipped flag vocabulary lives in `agent_team_flags.md`. To add a new primitive for your project or organization:

1. Add a row to `local.agent_team_flags.md` (gitignored):
   ```
   | <flag> | inline|annotation | <meaning> |
   ```
2. Verify with `resolve_agent_teams.py --flags` — the new flag should appear.

The resolver parses flags generically, so a new term works the moment it appears in the registry and is used in a team definition.

**Extension rule:** Add a new primitive only for a genuinely new control-flow concept. Never encode role-specific work logic or team-specific details as a primitive.

---

## Community Goal

SAILL is designed to be a shared, open language for agentic loops — adopted and extended across harnesses and projects. The defining properties that make it portable:

- **Vendor-neutral** — groups reference capability tiers, not provider model IDs
- **Context-light** — rich meaning in one or two words; no large schema required
- **Human-readable** — reads as near-English in place; self-documenting
- **Composable** — a small closed set of primitives; complexity comes from composition
- **Define-once, share freely** — the same team definition works anywhere the primitives are understood

A team definition is shareable as a markdown snippet. Anyone with a compatible agent harness can drop it in and invoke it by name.
