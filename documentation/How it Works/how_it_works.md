# How SAILL Works

This document covers the mechanics: how context gets into a session, how teams and model preferences are loaded, how the acting model resolves a team name and executes roles, and how scope cascades across a folder hierarchy.

---

## How Claude Code Loads Context

When a Claude Code session starts in a directory, the harness assembles the system prompt by walking the directory tree from the user-global location down to the current working directory (CWD) and loading every `CLAUDE.md` it finds along the way.

### The loading chain

```
~/.claude/CLAUDE.md          (user-global — all sessions for this OS user)
  <root>/CLAUDE.md           (project root)
    <subfolder>/CLAUDE.md    (any intermediate directory between root and CWD)
      <CWD>/CLAUDE.md        (innermost — most specific)
```

All files load. They stack. Outermost first, innermost last. This means a session started in a deeply nested project folder inherits every layer above it.

A `CLAUDE.local.md` alongside any `CLAUDE.md` is also loaded. These are machine-local overrides (local paths, API keys, personal conventions) and must not be committed to version control.

### @-imports

The `@file` syntax inside a `CLAUDE.md` is a harness directive. The harness resolves it before the conversation starts, reading the referenced file and inlining its bytes directly into the system prompt.

**Critical properties:**

1. **Unconditional loading.** There is no lazy or conditional loading. Once a file is `@-imported`, it is always in context — every session, every time. Any prose on the same line as the `@` directive (e.g., "only load if needed") is inlined content, not a condition the harness evaluates.

2. **Only CLAUDE.md triggers harness resolution.** An `@file` line inside `agents.md` is passed to the model as plain text — the harness does not inline the referenced file. Only `CLAUDE.md` (and `CLAUDE.local.md`) files trigger harness `@-import` resolution.

3. **Recursive resolution.** If `CLAUDE.md` imports `agents.md`, and `agents.md` contains `@-references`, those are resolved further — because the resolution context started from a `CLAUDE.md`.

> **Note (June 2026):** In practice, Claude Code does not reliably honor `@-imports` inside `agents.md`. Files that `agents.md` intends to import should be `@-imported` directly in `CLAUDE.md` to guarantee loading. This is a known limitation observed during initial testing.

### The right pattern for optional content

If you want a file available only when the model explicitly needs it, do not `@-import` it. Write a prose pointer instead:

```
If you need the full flag catalog, read ./agent_team_flags.md.
```

That one line costs a handful of tokens in always-loaded context. The referenced file costs nothing until the model reads it mid-session.

---

## How agent_teams.md and model_prefs.md Get Loaded

These files are loaded via `@-imports` in `CLAUDE.md`:

```
# CLAUDE.md
@agent_teams_flags.md   # flag vocabulary
@agent_teams.md         # team definitions
@model_prefs.md         # model group config
```

Once loaded, the model has the team definitions and model group mappings in context for the entire session. No further reads are needed to invoke a team by name.

`model_prefs.local.md` (gitignored) is the file where actual model IDs are placed. It is `@-imported` alongside `model_prefs.md` to supply the group membership and task-class routing the base file defines as empty.

---

## How the Acting Model Resolves a Team and Spawns Roles

There is no resolver running in the background at invocation time. The team definitions are already in the system prompt. When you say "send the investigate-and-fix team," the acting model:

1. Looks up the team by name in the loaded `agent_teams.md` content.
2. Reads each role's model group (e.g., `#midcost`).
3. Consults `model_prefs.md` to resolve the group to an actual model ID.
4. Spawns each role in order as a sub-agent on the resolved model, passing the role's charter as its prompt.
5. Chains each role's output to the next role as input.

SAILL flags (like `if needed`, `parallel`, `Loop`) in the team definition modify this execution: skipping roles, running them concurrently, or looping back with feedback.

This is purely in-context instruction — the reliability of execution depends on the acting model following the loaded instructions.

---

## How Scope Cascade Works

Team definitions and model preferences cascade from the least-specific scope to the most-specific. The most-specific definition for a given team name or model group wins.

### Cascade order (least to most specific)

```
<root>/agent_teams.md                     (shipped base — do not edit)
  <root>/local.agent_teams.md             (root-level overrides and custom teams)
    <project>/local.agent_teams.md        (project-level overrides)
      <brain>/local.agent_teams.md        (brain-level overrides)
        <subfolder>/local.agent_teams.md  (subfolder-level overrides)
```

**Same-name overrides:** a team defined at a more-specific scope replaces the same-named team from a less-specific scope. New names are unioned in — they are added without removing anything from above.

**How it activates:** Each `agents.md` in the hierarchy can `@-import` its sibling `local.agent_teams.md`. When that `agents.md` is in the load chain for the current CWD, the override is active.

The same cascade applies to `model_prefs.local.md` for model group membership and routing rules.

---

## Summary of the Data Flow

```
Session start
  → harness walks <root> → CWD collecting CLAUDE.md files
  → resolves all @-imports (recursively from CLAUDE.md only)
  → system prompt = stacked layers, outermost first
      includes: agent_teams.md, agent_team_flags.md, model_prefs.md, local overrides
  → session runs; user invokes a team by name
  → acting model reads team def from loaded context
  → resolves #group → model ID via model_prefs
  → spawns roles in order, honoring SAILL flags
  → chains output through the role sequence
  → returns result
```
