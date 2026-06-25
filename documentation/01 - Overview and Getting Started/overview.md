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
| Context loading | The mechanism by which `agents.md` and imported files are assembled into a session's system prompt |

---

## Prerequisites

**A compatible agent harness** — any harness that:
- Reads `agents.md` from the working directory
- Resolves file imports declared in `agents.md` (or `CLAUDE.md`) and loads them into context
- Passes that assembled context to the model before the session starts

Examples: Claude Code, OpenAI Codex CLI, or any similar tool that reads a context manifest and sends it to the model. The model does all interpretation — no SAILL-specific tooling is required in the harness itself.

**What you supply:**
- `@-import` lines in your own `agents.md` or `CLAUDE.md` pointing at the SAILL files (`agent_teams.md`, `agent_team_flags.md`, `model_prefs.md`)
- A `model_prefs.local.md` (gitignored) with your actual model IDs filled into each group

Once those files are in place, reload your harness context so it picks up the imports. After that, SAILL is functional.

---

## Getting Started

The fastest path is the [single-folder basic implementation](../02%20-%20Tested%20Implementation%201/impl1.md) — a complete, tested setup you can run directly.

1. Clone or download the SAILL repository.
2. In your terminal, navigate to `tested_implementations/1 - single-folder_basic_implementation/`.
3. Open your agent harness (Claude Code, Codex, or similar) with that directory as the working directory.
4. Connect to your provider of choice.
5. Tell the harness to invoke the agent tests directly from the file: `run /test-agent-teams from agent_teams.md`.

This validates SAILL is functional in your environment before you integrate it into your own project.

**To integrate into your own project:**
1. Copy `agent_teams.md`, `agent_team_flags.md`, and `model_prefs.md` into your project directory (or point at them via environment variable paths — see [Tested Implementation 3](../11%20-%20Tested%20Implementation%203/impl3.md)).
2. Add `@-import` lines for those files in your `agents.md` or `CLAUDE.md`.
3. Create `model_prefs.local.md` in the same directory and fill in your model IDs for each group.
4. Reload your harness context.
5. Verify with the steps below.

---

## Verifying It Works

### 1. Confirm context loads

Run `/context-cost` from the directory. The output should show `CLAUDE.md` (or `agents.md`), `agent_teams.md`, `agent_team_flags.md`, and `model_prefs.md` in the loaded file list.

### 2. Inspect active teams

Run `/agent-teams` (bare). The skill displays all teams currently in effect, their roles, model groups, and source files.

Expected output: all four shipped teams visible, sourced from `agent_teams.md`.

### 3. Test spawning

Run `/test-agent-teams` and select a team. The skill spawns each role as a real sub-agent and verifies each role echoed the test nonce (proof of actual execution) and ran on a model consistent with its assigned `#group`.

> Note: `/test-agent-teams` spawns real agents — it incurs API cost.

### 4. Invoke a team manually

> "Send an investigate-and-fix team to look at the failing test."

The acting model should spawn an Investigator on `#midcost`, then a Fixer on `#lowcost`, chaining the diagnosis through.

---

## Where to Go Next

| Topic | File |
|---|---|
| Working single-folder example | [Tested Implementation 1](../02%20-%20Tested%20Implementation%201/impl1.md) |
| Defining and invoking agent teams | [Agent Groups](../03%20-%20Agent%20Groups/agent_groups.md) |
| Example loops (copy-paste team definitions) | [Example Loops](../04%20-%20Example%20Loops/example_loops.md) |
| Keeping context overhead low | [Evaluating Context Cost](../05%20-%20Evaluating%20Context%20Cost/context_cost.md) |
| How context loads, @-imports, scope stacking | [How it Works](../06%20-%20How%20it%20Works/how_it_works.md) |
| The full SAILL primitive set, boxes, loops, -context- | [SAILL Language Guide](../07%20-%20SAILL%20Language%20Guide/saill_guide.md) |
| Model groups, routing, per-session slots | [Model Preferences](../08%20-%20Model%20Preferences/model_preferences.md) |
| Multi-folder inheritance example | [Tested Implementation 2](../09%20-%20Tested%20Implementation%202/impl2.md) |
| Environment variable paths example | [Tested Implementation 3](../10%20-%20Tested%20Implementation%203/impl3.md) |
| Helper utilities reference | [Helper Utilities](../11%20-%20Helper%20Utilities/helper_utilities.md) |
