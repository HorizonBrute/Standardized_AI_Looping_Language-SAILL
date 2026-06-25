# SAILL — Major Features

SAILL (Standard AI Looping Language) is a compact, vendor-neutral notation for defining reusable multi-agent workflows. These are its defining capabilities.

---

## 1. Shareable Loops — Standard Interoperability

**Define a workflow once. Share it as a markdown snippet. Anyone with a compatible harness can use it immediately.**

Before SAILL, sharing an agentic workflow meant sharing a wall of natural-language prompt text — verbose, ambiguous, and tied to the author's context and phrasing. Two people using the "same" workflow were running two different things.

SAILL is the standard notation that makes workflows genuinely portable. A team definition is a small, self-contained markdown block built from a closed set of primitives. It reads the same way everywhere — to a human, to the acting model, and to any compatible agent harness. Drop it into a `local.agent_teams.md`, invoke it by name, and it runs.

```
### Deep Explore

1. Orchestrator (#highcap) — defines scope and fans out.
2. Recon[ APICrawler (#investigate, parallel), DBReader (#investigate, parallel),
          UITracer (#investigate, parallel) ] (wait)
3. Synthesizer (#midcost) — integrates findings into a single report.
```

Share this on a forum, in a README, in a community library. The recipient pastes it in and runs `Run the Deep Explore team against -context:target-`. No translation, no re-prompting, no loss of intent.

This is the core value of SAILL: it is to agent workflows what SQL is to data queries — a narrow, regular, shareable notation that any compatible system executes identically.

> See [Agent Groups](../03%20-%20Agent%20Groups/agent_groups.md) and [Example Loops](../04%20-%20Example%20Loops/example_loops.md).

---

## 2. SAILL Inside Skills — Skill Compression

**A verbose skill can be replaced with a compact SAILL block.**

A skill that spawns a multi-role agent team traditionally describes each role in prose — what to do, in what order, with what output. That prose re-explains things the acting model already knows from the SAILL vocabulary. The SAILL-in-skill pattern eliminates that redundancy.

Write the team as a SAILL block inside the skill body. Close with one dispatch line:

```
**Send the <Team Name> team against -context-**
```

The model executes the block directly. Same roles, same execution order, same output contract — at roughly 20% of the token cost.

**Tested result:** a 4-role, ~50-line prose skill reduced to ~10 lines with no loss of behavior.

> See [SAILL Inside Skills](../12%20-%20SAILL%20Inside%20Skills/saill_in_skills.md) — full pattern, authoring rules, live before/after comparison.

---

## 3. Define-Once, Use-Many

**Name a team once. Invoke it by name anywhere.**

A team defined in `agent_teams.md` (or `local.agent_teams.md`) is available by name in any prompt, skill, or agent instruction in that scope. Reuse is a reference, not a copy. Shared workflows stay in sync automatically.

> See [Agent Groups](../03%20-%20Agent%20Groups/agent_groups.md) and [Example Loops](../04%20-%20Example%20Loops/example_loops.md).

---

## 4. Vendor-Neutral Model Groups

**Route by capability tier, not by provider.**

SAILL roles reference model groups (`#lowcost`, `#midcost`, `#highcap`, `#investigate`) rather than specific model IDs. The group resolves to whatever model is configured for that tier in the current environment. Swap providers or upgrade models without touching a single team definition.

```
Investigator (#investigate, parallel)
Builder (#lowcost)
Reviewer (#highcap, ask user)
```

> See [Model Preferences](../08%20-%20Model%20Preferences/model_preferences.md).

---

## 5. Context-Light Runtime Values

**Structure is fixed. Specifics arrive at runtime.**

`-context:<qualifier>-` placeholders let a team definition remain fully generic. The acting model resolves placeholders from the user's invocation, the conversation, or runtime state. One team definition covers every variation of a task.

```
1. Scanner (#lowcost) — globs target files under -context:root-; emits list as -context:dirs-.
2. Reader (#lowcost, parallel) per -context:dirs-
3. Synthesizer (#midcost) — collects results; writes to -context:output path-.
```

> See [SAILL Language Guide — Context Values](../07%20-%20SAILL%20Language%20Guide/saill_guide.md#context-values----context-).

---

## 6. Native Parallelism and Control Flow

**Concurrency, loops, gates, and failure handlers — in one or two words each.**

The full SAILL primitive set:

| Primitive | What it does |
|---|---|
| `parallel` | Run concurrently with adjacent parallel-flagged roles |
| `wait` | Sync point — wait for the preceding parallel group |
| `Loop` | Re-run with feedback until pass condition or iteration cap |
| `if needed` | Run only if the model judges it adds value |
| `if asked` | Run only when the user explicitly requests it |
| `ask user` | Pause for user input or approval |
| `[ … ]` box | Group roles into a named sub-team |
| `if fail <action>` | Failure handler — run on cap-exhausted loop or role failure |

Every construct composes with every other. Complexity comes from composition, not from bespoke flags.

> See [SAILL Language Guide](../07%20-%20SAILL%20Language%20Guide/saill_guide.md) for the full reference.

---

## 7. Scope Cascade

**Teams, model preferences, and flags inherit and override by directory depth.**

SAILL configuration stacks: OS-level → project-level → brain-level → subfolder. Each layer can add or override what's above it. A brain gets the full default vocabulary from every enclosing scope plus its own local customizations, with no duplication.

> See [Tested Implementation 2](../09%20-%20Tested%20Implementation%202/impl2.md) and [How it Works](../06%20-%20How%20it%20Works/how_it_works.md).
