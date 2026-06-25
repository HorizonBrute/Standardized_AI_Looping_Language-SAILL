# Model Preferences

The model-preference layer directs which model is used for **spawned agents and delegated tasks**. It does not change the model you are talking to in the current session — that is set by the harness at launch.

The mechanism is in-context instruction, not enforcement. The acting model reads the loaded `model_prefs.md` and follows it when spawning agents. Reliability is best-effort and depends on the acting model honoring the instructions.

---

## What Model Preferences Govern

| What | Governed? |
|---|---|
| Your interactive session model | No — set by harness/provider at launch |
| Spawned agent model | Yes |
| Sub-agent model (within spawned agents) | Yes, via Sub-Agent Override |
| Task-class routing (automatic routing by category of work) | Yes |

---

## The Shipped Model Groups

Named capability tiers. Teams reference groups, not model IDs, so team definitions are portable across providers and model generations.

| Group | Intended use |
|---|---|
| `#lowcost` | Routine, mechanical work — status checks, summaries, formatting |
| `#midcost` | Standard development and writing tasks |
| `#highcap` | Complex reasoning, cross-document synthesis, architecture decisions |
| `#investigate` | Research-heavy tasks requiring deep comprehension |
| `#debug` | Debugging passes, root cause analysis |
| `#fast` | Latency-sensitive operations where speed matters more than depth |

The system selects the **first runnable member** of the group in the current runtime. Members that don't match the runtime are silently skipped — no errors.

---

## Member Grammar

```
bare model-id           (any runtime)
runtime:model-id        (runtime-qualified)
```

Examples:
```
claude:haiku
claude:sonnet
claude:opus
claude:fable
ollama:llama3.2
```

Anthropic aliases (`claude:haiku`, etc.) are preferred over pinned full model IDs unless you need a specific version — aliases track the current best model in that tier. `ollama:` members that aren't pulled locally silently no-op until `ollama pull`.

---

## Per-Session Slots

Two override slots adjust the default model for all spawned work in a session:

| Slot | Effect |
|---|---|
| **Spawned Agent Model** | Override the default model for all spawned agents |
| **Sub-Agent Override** | Override specifically for sub-agents |

Both default to `Unset`. Set them in `model_prefs.local.md` to activate.

---

## Task-Class Routing

Directs entire categories of work to the right tier automatically, without naming a group in each prompt:

```markdown
## Task-Class Routing
- pr status checks, inbox triage, summaries -> #lowcost
- full code review, security audit          -> #highcap
- debugging, root cause analysis            -> #debug
```

More-specific task class wins on conflict.

---

## Resolution Order

When the acting model needs to pick a model for spawned work:

1. Named group from the prompt (e.g., `#highcap` in a team role)
2. Task-Class Routing match — first runnable member of the mapped group
3. Sub-Agent Override (sub-agents only, if set)
4. Spawned Agent Model (if set)
5. Harness/provider default

---

## Scope Cascade

Model preferences cascade from least to most specific. Most-specific scope wins on conflict.

```
OS-global model_prefs.md / model_prefs.local.md
  project-root model_prefs.local.md
    brain-root model_prefs.local.md
      subfolder model_prefs.local.md   (most specific)
```

**Merge rules:**
- Slots: more-specific wins if not `Unset`
- Groups: membership combined across scopes
- Routing: more-specific scope wins on conflict

Override files are `@-imported` from the scope's `agents.md` — never from `CLAUDE.md`.

---

## Setup

1. Copy the tracked template (`model_prefs.md`) — do not edit it.
2. Create `model_prefs.local.md` in the same directory (gitignored).
3. Run `/model-catalog-refresh` to get current model IDs and pricing from provider documentation.
4. Fill in group members using model IDs from the catalog:
   ```markdown
   ### #lowcost
   claude:haiku

   ### #midcost
   claude:sonnet

   ### #highcap
   claude:opus
   ```
5. Add task-class routing rules as needed.
6. Ensure `agents.md` `@-imports` `./model_prefs.local.md`.

All user choices go in the gitignored local file. Never put model IDs in the tracked base file.

---

## Skill Model Group Callouts

Every skill declares which model group its work should run on, as a one-line callout in the skill body:

```
**Model preference:** `#midcost` (per `model_prefs.md`; overridable by a prompt directive).
```

Properties:
- Must be in the skill body — frontmatter is stripped before the model sees it
- It is documentation the acting model reads, not enforcement
- A prompt directive (`"use #highcap"`) always overrides the callout
- It directs work the skill delegates to agents; it does not change the session model running the skill

**Group assignment heuristics for skills:**
- `#highcap` — security-sensitive, privileged, or destructive changes; deep judgment
- `#investigate` — research, live fetching, cross-source analysis
- `#midcost` — structured authoring/summarization with moderate judgment
- `#lowcost` — mechanical, read-only, or report-only work
- `#fast` — trivial single actions where latency matters
- `#debug` — step-by-step debugging or coding tasks

When two fit, prefer the cheaper unless a mistake would be costly.

---

## Related Skills

| Skill | Purpose |
|---|---|
| `/model-prefs` | Configure-ramp for model preferences: define groups, set slots, add routing, inspect |
| `/model-prefs-assign` | Audit and assign model-group callouts in skill files |
| `/model-prefs-test` | Verify that groups resolve and spawned agents actually run on the intended model |
| `/model-catalog-refresh` | Fetch live model IDs and pricing from provider documentation |
