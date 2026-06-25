# Evaluating Context Cost

**Context cost** is the fixed token overhead paid on every session. Every file loaded into the system prompt at session start is billed unconditionally — even if the model never references that content during the session.

SAILL's file design choices directly affect this overhead. Understanding context cost is essential to keeping sessions efficient.

---

## What Gets Loaded and Why

The harness assembles the system prompt by walking from the user-global `~/.claude/` directory down to the current working directory, loading every `CLAUDE.md` it finds, plus all their `@-imports` (resolved recursively).

**Files that are always loaded when in the chain:**

- `CLAUDE.md` (and `CLAUDE.local.md`) at every ancestor scope from user-global to CWD
- Every file `@-imported` from those `CLAUDE.md` files, recursively

**For a typical SAILL setup, that includes:**
- `agents.md`
- `agent_teams.md`
- `agent_team_flags.md`
- `model_prefs.md`
- `model_prefs.local.md`
- `local.agent_teams.md` (if `@-imported`)
- `local.agent_team_flags.md` (if `@-imported`)

All of these are the **always-loaded chain** — the set of files whose bytes enter every session.

---

## The Unconditional Loading Rule

There is no lazy loading, conditional loading, or on-demand loading for `@-imported` files.

Once a file is referenced with `@` in a `CLAUDE.md`, it is loaded for every session in that scope. Any prose on the `@-import` line — including "only load if needed" or "optional" — is inlined as content, not interpreted as a condition by the harness.

**Implication for file design:** Put only truly always-needed content in `@-imported` files. For optional reference material, use a prose pointer:

```
If you need the extended flag glossary, read ./extended_flags_reference.md.
```

The pointer costs a few tokens. The referenced file costs nothing until Claude reads it mid-session.

---

## Token Economy Rules

1. Keep the always-loaded total **under 1000 tokens** if possible.
2. At **2000 tokens**, review and trim before adding more content.
3. Every sentence that can be removed without losing a rule should be removed.
4. Do not put lengthy explanations, examples, or reference material in `@-imported` files.
5. No commented-out content — dead lines still cost tokens.

---

## The Terseness Contract

The Terseness Contract is the authoring standard for every file in the always-loaded chain. Because each byte is billed every session, verbosity is a cost imposed on every interaction.

A file in the always-loaded chain passes the Terseness Contract if it satisfies all seven criteria:

| # | Criterion |
|---|---|
| 1 | Every line earns its keep — removing it would break the file's function |
| 2 | Instructions are imperative, not discursive — tell the model what to do |
| 3 | No rationale that belongs in documentation elsewhere |
| 4 | No inline examples when a prose pointer to an example file suffices |
| 5 | No redundancy with sibling always-loaded files |
| 6 | No commented-out content — dead lines still cost tokens |
| 7 | `@-imports` only for files that must be present in every session |

Apply these criteria when authoring `agents.md`, `model_prefs.md`, `agent_teams.md`, `agent_team_flags.md`, and any local override files.

---

## How to Measure Context Overhead

### context_cost.py

A Python script that walks the ancestor chain from a target path to the filesystem root, collecting every `CLAUDE.md`, `CLAUDE.local.md`, and `agents.md` the harness would load, plus all `@-imports`. Reports KB, words, and estimated token count per file and as a total.

```
python context_cost.py [path]
python context_cost.py [path] --json
```

`path` defaults to CWD. `--json` emits machine-readable output.

Run `context_cost.py` after adding or modifying any `CLAUDE.md` or `@-import`. Confirm overhead stayed in budget before committing.

### /context-cost skill

Runs `context_cost.py --json`, formats the output as a table, and flags thresholds:

- `[NOTE]` — >= 1000 tokens: moderate context load
- `[WARN]` — >= 2000 tokens: high context load; review before proceeding

---

## Best Practices

**What to `@-import` (always-loaded):**
- `agent_teams.md` — needed every session to resolve team names
- `agent_team_flags.md` — needed every session to interpret team flags
- `model_prefs.md` + `model_prefs.local.md` — needed every session to resolve `#group` names

**What to leave as a prose pointer (on-demand):**
- Extended flag glossaries
- Example libraries
- Reference documentation
- Implementation notes and background rationale

**Format discipline:**
- Use tables instead of prose lists where the content is tabular
- Keep role charters to one line
- Keep team descriptions to one sentence
- Keep routing rules to one line each

**Scope discipline:**
- Add files at the most-specific scope where they are actually needed
- A file `@-imported` at the org root is loaded for every project in the org
- A file `@-imported` at the project level is loaded only in that project's sessions
