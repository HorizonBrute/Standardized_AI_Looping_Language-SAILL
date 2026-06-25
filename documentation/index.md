# SAILL Documentation Index

**SAILL (Standard AI Looping Language)** is a compact, vendor-neutral notation for defining reusable multi-agent workflows. Define a workflow once, name it, invoke it by name — or share it with anyone using a compatible agent harness. Like SQL for data queries, SAILL is the standard notation for agent-loop definitions.

---

## Table of Contents

| # | Document | Description |
|---|---|---|
| 01 | [Overview and Getting Started](01%20-%20Overview%20and%20Getting%20Started/overview.md) | What SAILL is, why it exists, key concepts, prerequisites, getting started path, and verification steps |
| 02 | [How it Works](02%20-%20How%20it%20Works/how_it_works.md) | Context loading mechanics, @-import rules, scope stacking, and how teams execute |
| 03 | [SAILL Language Guide](03%20-%20SAILL%20Language%20Guide/saill_guide.md) | The full primitive set, naming rules, boxes/nesting, -context- values, failure handlers, and worked examples |
| 04 | [Agent Groups](04%20-%20Agent%20Groups/agent_groups.md) | What agent teams are, the four shipped teams, conditions, loops, custom teams, and scope cascade |
| 05 | [Model Preferences](05%20-%20Model%20Preferences/model_preferences.md) | Model groups, member grammar, per-session slots, task-class routing, scope cascade, and skill callouts |
| 06 | [Example Loops](06%20-%20Example%20Loops/example_loops.md) | Eleven annotated copy-paste team definitions covering every SAILL primitive, each with a human-prompt origin, with a quick-reference table |
| 07 | [Helper Utilities](07%20-%20Helper%20Utilities/helper_utilities.md) | Reference for all AIOS system utilities across bin, sbin, and usrbin — names, descriptions, and file links |
| 08 | [Evaluating Context Cost](08%20-%20Evaluating%20Context%20Cost/context_cost.md) | What context cost is, which files load, the unconditional loading rule, token economy rules, and the Terseness Contract |
| 09 | [Tested Implementation 1](09%20-%20Tested%20Implementation%201/impl1.md) | Single-folder basic setup: files, wiring, known @-import limitation, and how to verify |
| 10 | [Tested Implementation 2](10%20-%20Tested%20Implementation%202/impl2.md) | Multi-folder context inheritance: hierarchy, what stacks at each level, and override mechanics |
| 11 | [Tested Implementation 3](11%20-%20Tested%20Implementation%203/impl3.md) | Environment variable paths: pointing @-imports at a variable location for flexible or shared deployments |
