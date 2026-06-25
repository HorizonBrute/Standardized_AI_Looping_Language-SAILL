# SAILL — Standard AI Looping Language

A compact, vendor-neutral notation for defining reusable multi-agent workflows.
Define a workflow once, name it, invoke it by name — or share it with anyone using a compatible agent harness.
Think of it as SQL for agent-loop definitions.
fff
---

## The Problem

Running multiple AI agents in a coordinated workflow today means writing a long natural-language prompt every time, or building bespoke tooling that hard-codes the flow. Both approaches are non-portable, expensive in tokens, and not shareable. Changing the workflow means rewriting the prompt or the code.

SAILL solves this by letting you define the workflow once as a named team, then invoke it in one line. The same definition is shareable with anyone on a compatible harness — no SAILL-specific tooling required.

In practice this means: define your multi-agent loop once, reference it by name in prompts or in skills. Teams defined in skills can reduce repeated-prompt token cost by 80% or more compared to writing the full workflow out each session. (Proof: [before/after skill comparison](skills/test_saill_in_skill/example_conversion.md) vs. [`skills/test_saill_in_skill/SKILL.md`](skills/test_saill_in_skill/SKILL.md) — documented in [SAILL Inside Skills](documentation/12%20-%20SAILL%20Inside%20Skills/saill_in_skills.md).)

---

## Quick Examples

### Example 1 — Simple retry loop

**Human prompt:** "Apply the linting fixes and keep retrying until the suite is clean — give up after 3 attempts."

```
### Audit & Fix

1. Implementer (#lowcost) — applies the change.
2. Validator (#midcost) — runs the audit suite against the change.
   **Loop:** on failure, return specific findings to "Implementer" and re-run from there;
   repeat until the Validator passes clean or 3 iterations, then stop and report
   any outstanding failures.
```

**Invoke:** `"Send the Audit & Fix team at the current diff."`

---

### Example 2 — Parallel fan-out with synthesis

**Human prompt:** "Pull data from the API, the database, and the UI traces all at once, then give me one combined report."

```
### Parallel Recon & Synthesize

1. Orchestrator (#highcap) — defines the investigation scope and fans out.
2. Recon[ SourceA (#investigate, parallel),
          SourceB (#investigate, parallel),
          SourceC (#investigate, parallel) ] (wait)
3. Synthesizer (#midcost) — merges all findings into a single actionable report.
```

**Invoke:** `"Send the Parallel Recon & Synthesize team across the payment service sources."`

---

Both definitions are portable: drop either into any project's `agent_teams.md`, swap the role charters for your domain, keep the control-flow structure. The model does all interpretation — no SAILL-specific tooling is required in the harness.

---

## Key Concepts

| Concept | Description |
|---|---|
| **Agent team** | A named, ordered list of roles with model groups and control-flow flags |
| **Role** | A single unit of work: label + model group + one-line charter |
| **Model group** | A named capability tier (`#lowcost`, `#midcost`, `#highcap`, `#investigate`) mapped to your actual model IDs — defined once, referenced everywhere |
| **Model preferences** | A gitignored local file (`model_prefs.local.md`) that maps each model group to your actual model IDs. Filled in once per environment; never committed. The shipped `model_prefs.md` is a conservative template — all groups intentionally set to `Unset` until you configure them. |
| **SAILL primitive** | A control-flow keyword: `if needed`, `if asked`, `parallel`, `wait`, `Loop`, `ask user`, `-context:<value>-` |
| **Scope cascade** | Team and model definitions stack from a root directory down to the current folder; most-specific wins |

---

## Technical Summary

SAILL works by loading team definitions into the model's context via standard `@-import` files (`agent_teams.md`, `agent_team_flags.md`, `model_prefs.md`). The mechanism is the `CLAUDE.md` → `agents.md` hierarchy: `CLAUDE.md` `@-imports` `agents.md`, which in turn `@-imports` the SAILL files — assembling the full context before any session begins. When you invoke a team by name, the model interprets the SAILL expression — spawning roles, applying control-flow primitives, and routing to model groups — using only the already-loaded context. No additional tooling is required at runtime.

Scope cascade means these files can live at any directory level; definitions in a subdirectory's `agent_teams.md` override or extend the root-level definitions, so different projects or folders get different team behavior without touching the shared base.

Key mechanics: named teams + SAILL primitives (`parallel`, `wait`, `Loop`, `if needed`, `if asked`, `ask user`, `-context:<value>-`) + model groups (`#lowcost`, `#midcost`, `#highcap`, `#investigate`) + scope cascade (`CLAUDE.md` → `agents.md` → SAILL files, root → current directory, most-specific wins).

**Harness compatibility:** `agents.md` and `CLAUDE.md` have become the de facto standard context manifest format for agentic developer tools. Claude Code and OpenAI Codex CLI support this natively — open the working directory and the context loads automatically. Any other harness can implement it with a straightforward wrapper: walk the directory tree, collect `agents.md` / `CLAUDE.md` files, resolve `@-imports`, and assemble the system prompt before session start. No SAILL-specific tooling is required. The lego blocks fit.

**Tested environments:** Claude (multiple models and versions), OpenAI Codex CLI, Ollama (local models), Ollama configured as a wrapper around Claude, and Ollama pointed at web-hosted models — all confirmed working.

> **Note on model selection:** SAILL execution is fully in-context — the acting model reads team definitions and follows them using its own reasoning. Testing has focused on large-context, capable models. Smaller or less capable models may not reliably execute complex multi-role team structures. Test your target model in your environment before deploying.

---

## Getting Started

### First run — verify SAILL works

**Prerequisite:** Clone this repo.

1. Open your context-aware harness (Claude Code, Codex CLI, or any compatible harness) with `tested_implementations/1 - single-folder_basic_implementation/` as the working directory.
2. Run `/test-agent-teams`.

The skill spawns each role as a real sub-agent, verifies each role echoed the test nonce, and reports which model each role ran on. You'll see multi-role chaining live in your environment.

> `/test-agent-teams` spawns real agents — it incurs API cost.

Full walkthrough: [Overview and Getting Started](documentation/01%20-%20Overview%20and%20Getting%20Started/overview.md)

### Integrate into your own project

1. Copy `agent_teams.md`, `agent_team_flags.md`, and `model_prefs.md` into your project directory (or point at them via environment variable paths — see [Tested Implementation 3](documentation/10%20-%20Tested%20Implementation%203/impl3.md)).
2. Add `@-import` lines for those files in your `CLAUDE.md` (recommended) or `agents.md`:
   ```
   @'./agent_teams.md'
   @'./agent_team_flags.md'
   @'./model_prefs.md'
   ```
3. Create `model_prefs.local.md` alongside `model_prefs.md` and fill in your actual model IDs for each group. The shipped `model_prefs.md` is a conservative template — all groups set to `Unset` by default; `model_prefs.local.md` is required to activate model routing.
4. Reload your harness context.
5. **Create and test your first team** using the included skills:
   - `/convert-prompt-to-saill` — convert a natural-language workflow description into a SAILL team definition
   - `/try-my-prompt` — converts a prompt and returns a ready-to-paste `agent_teams.md` entry plus a plain-English version
   - `/test-agent-teams` — end-to-end test: spawns each role as a real sub-agent and verifies execution
6. Verify context loading with `/context-cost` (lists every file loaded and estimated token cost).

---

## Documentation

| # | Document | Description |
|---|---|---|
| 00 | [Major Features](documentation/00%20-%20Major%20Features/major_features.md) | Defining capabilities — SAILL-in-skill compression, define-once-use-many, vendor-neutral model groups, native parallelism, scope cascade |
| 01 | [Overview and Getting Started](documentation/01%20-%20Overview%20and%20Getting%20Started/overview.md) | What SAILL is, why it exists, key concepts, prerequisites, and verification steps |
| 02 | [Tested Implementation 1](documentation/02%20-%20Tested%20Implementation%201/impl1.md) | Single-folder basic setup: files, wiring, and how to verify |
| 03 | [Agent Groups](documentation/03%20-%20Agent%20Groups/agent_groups.md) | Defining and invoking teams, shipped examples, conditions, loops, custom teams, scope cascade |
| 04 | [Example Loops](documentation/04%20-%20Example%20Loops/example_loops.md) | Twelve annotated copy-paste team definitions covering every SAILL primitive |
| 05 | [Evaluating Context Cost](documentation/05%20-%20Evaluating%20Context%20Cost/context_cost.md) | What loads, the unconditional loading rule, token economy rules, and the Terseness Contract |
| 06 | [How it Works](documentation/06%20-%20How%20it%20Works/how_it_works.md) | Context loading mechanics, @-import rules, scope stacking, and execution flow |
| 07 | [SAILL Language Guide](documentation/07%20-%20SAILL%20Language%20Guide/saill_guide.md) | Full primitive set, naming rules, boxes/nesting, -context- values, failure handlers |
| 08 | [Model Preferences](documentation/08%20-%20Model%20Preferences/model_preferences.md) | Model groups, member grammar, per-session slots, task-class routing, scope cascade |
| 09 | [Tested Implementation 2](documentation/09%20-%20Tested%20Implementation%202/impl2.md) | Multi-folder context inheritance: hierarchy, what stacks at each level, override mechanics |
| 10 | [Tested Implementation 3](documentation/10%20-%20Tested%20Implementation%203/impl3.md) | Environment variable paths for flexible or shared deployments |
| 11 | [Helper Utilities](documentation/11%20-%20Helper%20Utilities/helper_utilities.md) | Reference for bin utilities — context_cost.py, resolve_agent_teams.py, and testing tools |
| 12 | [SAILL Inside Skills](documentation/12%20-%20SAILL%20Inside%20Skills/saill_in_skills.md) | Embedding SAILL team notation directly in a skill body — authoring rules and token economy |

Full index: [documentation/index.md](documentation/index.md)

---

## Contributing

SAILL is an early-stage open standard. The goal right now is simple: get it into people's hands so real usage can reveal where it works, where it breaks, and what's missing. That stress-testing is more valuable than any single code contribution at this point.

This is also my first public open-source project. I've worked with Git inside organizations, but this is my first time putting something in community space and genuinely asking for input. I'm open to it, and I'll do my best to engage.

Here's where contribution is most useful right now:

---

### 1. Submit a well-crafted example loop

The single highest-value contribution: a real, tested SAILL team definition that solves a workflow you actually use.

Good examples are the fastest way for new users to understand and adopt the language. They also stress-test the primitives in ways I haven't thought of.

**How to submit one:**

1. Fork the repo
2. Add your loop to `documentation/04 - Example Loops/`
3. Follow the format in the existing examples: team name, the human prompt it encodes, the SAILL expression, and a one-line invoke example
4. Open a PR with a short description of the workflow and why you built it

The bar is: does it use current primitives correctly, does it represent a genuinely useful workflow, and is it something someone else could pick up and adapt?

---

### 2. Propose a new primitive or flag

SAILL is deliberately minimal. The primitives are meant to stay small — these definitions get read into context on every session, so token cost is a real constraint. A new flag needs to carry its weight.

The question I'll ask about any proposal: **does this fit the primitives-first, low-context-overhead philosophy, or does it add complexity without adding expressive power that can't already be covered by combining existing primitives?**

If you think you've found something that genuinely can't be expressed today and belongs in the core language, open an issue or a PR and make the case. I'm not looking for a long list of edge-case flags — I'm looking for things that represent real innovation in how agents can be directed. I don't have all the answers here. If you've found one, I want to hear it.

---

### What I'm not looking for right now

SAILL is intentionally a primitive layer. I'd rather keep it tight and let the community build on top of it than try to cover every case in the spec itself. If you're looking to take SAILL in a new direction — fork it, improve it, build on it. That's the point of making it open.

---

Questions, ideas, or just want to share how you're using it — open an issue. I'm learning how this works too.
