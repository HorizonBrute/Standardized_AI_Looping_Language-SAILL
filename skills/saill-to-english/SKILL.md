---
name: saill-to-english
description: Translate a SAILL agent-team definition into plain English — render every primitive, loop, box, flag, and -context- placeholder as natural language a non-technical reader can follow. Use when the user types /saill-to-english, says "translate this SAILL", "explain this team definition", "what does this loop do", or "describe this in plain English".
tools: Read
---

# Skill: /saill-to-english

**Model preference:** `#lowcost` (translation is mechanical; no reasoning about the codebase required).

Turn a SAILL agent-team block into fluent, plain English — a step-by-step description of what the workflow does, in what order, under what conditions, and what happens when things go wrong. The output should be readable by someone who has never seen SAILL.

---

## When to invoke

`/saill-to-english <SAILL block>`, or the user pastes a team definition and asks what it does / how to explain it.

---

## Step 1 — Load the primitive vocabulary

1.1 Read `working_copy/documentation/SAILL Language Guide/saill_guide.md` for the authoritative meaning of every primitive.
1.2 Do not invent meanings. If a token is not in the vocabulary, flag it as an unrecognized extension.

---

## Step 2 — Parse the SAILL block

Work through the definition in order:

| SAILL construct | What to identify |
|---|---|
| `### Name` | The team's name and a one-sentence purpose |
| Numbered roles `N. Label (#group)` | Each step; its label, capability tier, and charter |
| `(#group, if needed)` | Conditional — model decides at runtime |
| `(#group, if asked)` | Conditional — only runs when the user explicitly requests it |
| `(#group, ask user)` | Hard gate — workflow pauses for user input before this step |
| `(parallel)` / `(wait)` | Concurrent roles; the `wait` is the sync point |
| `Box[ … ]` | Named or anonymous sub-team; note what runs inside |
| `**Loop:**` | Retry block — extract: trigger, loop-back target, pass condition, cap, cap action |
| `or ask user` in a Loop | User can stop the loop early by choice |
| `-context:<name>-` | Runtime placeholder — note what must be supplied at invocation |
| `if fail <action>` | Failure handler — what runs if the loop cap is hit unmet |
| `/skill-name` in a charter | Named skill the role invokes as its work |

---

## Step 3 — Render to English

3.1 **Opening line** — one sentence naming the team and its top-level purpose.

3.2 **Step-by-step narration** — for each role or box, one paragraph:
  - What the role does (its charter, in plain English)
  - Any condition on whether it runs at all (`if needed` / `if asked` / `ask user`)
  - If it is part of a parallel group: name the other roles running alongside it and note they all run at the same time, then sync before the next step
  - If it has a Loop: explain in plain English — "If the result fails, the workflow goes back to [Role] with specific feedback and tries again. This can repeat up to [cap] times. If it still hasn't passed by then, [cap action]." If `or ask user` is present: "At any point, you can choose to stop early and accept the current result."

3.3 **Placeholders** — list each `-context:<name>-` and what the user or invocation must supply for it.

3.4 **Failure handlers** — if any `if fail` actions are present, describe what triggers them and what they do.

---

## Step 4 — Output format

```
**[Team Name]**
<One-sentence summary of what the team accomplishes.>

**Steps:**
1. **[Role]** — <plain-English charter>. [Condition note if any.]
2. **[Role A]** and **[Role B]** run at the same time. Once both finish, the workflow continues.
   - **[Role A]** — <charter>
   - **[Role B]** — <charter>
3. **[Role]** — <charter>. If validation fails, the workflow loops back to step N and tries again,
   up to X times. If it still hasn't passed, <cap action>. [User-stop note if applicable.]

**What you need to supply at run time:**
- `-context:pass criteria-` — the quality bar that counts as a passing result
- `-context:cap-` — how many retry attempts to allow

**Failure handling:**
- If the loop hits its cap without passing, `/skill-name` is called to triage the failure.
```

Keep the output tight. One paragraph per step is the ceiling — the goal is clarity, not exhaustiveness.

---

## Notes for the executing agent

1. Translate faithfully — do not add or remove behavior implied by the SAILL.
2. A `#group` token is a model-group reference, not a literal model name. To resolve what is actually in a group, read the active model-preferences config (`horizon_aios_model_prefs.md` and any in-scope `model_prefs.local.md`). Render the group as its resolved name(s) when that adds clarity; otherwise render it as "the `#group-name` model group" — never invent a human label from memory.
3. If the input is ambiguous or uses an unrecognized token, note it clearly rather than guessing.
4. This skill only translates — it does not run, save, or modify the team definition.
