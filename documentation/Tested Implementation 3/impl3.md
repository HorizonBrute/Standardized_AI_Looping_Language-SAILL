# Tested Implementation 3 — Environment Variables in Agent Team Files

This implementation demonstrates using environment variables as `@-import` paths in `CLAUDE.md` and `agents.md`, so the actual location of SAILL definition files can be changed at the shell level without editing the markdown files.

**Location:** `tested_implementations/3 - environment_variables/`

---

## What This Implementation Demonstrates

- Referencing SAILL files by environment variable path instead of literal relative paths
- Pointing a project's `agents.md` at a centrally-managed or shared set of team definitions
- Changing the active team definitions by updating an environment variable rather than a file

---

## The Problem It Solves

In a standard setup, `agents.md` references files by relative path:

```
@agent_teams.md
@model_prefs.md
```

This works when the files are co-located with the project. But if you want to:
- Point multiple projects at the same shared set of teams without copying files
- Let a team or organization push out team definition updates to all members centrally
- Switch between different team-definition sets (e.g., different environments or use cases)

...then hard-coded relative paths require editing files in each project.

---

## How It Works

`agents.md` uses a shell environment variable in the `@-import` path:

```
@'$ENV_VAR_LOC1/agent_teams.md'        # Load Agent Teams definitions
@'$ENV_VAR_LOC1/agent_teams_flags.md   # Load flag definitions
```

`$ENV_VAR_LOC1` is set in the shell environment before launching the session. The harness resolves the variable at load time and inlines the file from that location.

The `Location_1/` and `Location_2/` directories in the implementation represent two different locations that `$ENV_VAR_LOC1` might point to — for example, a local repo vs. a shared network location vs. a different team's definitions.

---

## When to Use This Pattern

**Small teams sharing definitions without a project hierarchy:**
Set a shared environment variable pointing to a central repo location. All team members point their `agents.md` at the same shared files without needing a hierarchical `CLAUDE.md` structure.

**Switching between definition sets:**
Change `ENV_VAR_LOC1` to switch which `agent_teams.md` the project uses. The current shell session will use the new variable on the next session start (not the current session — context is fixed at session start).

**Organizational deployments:**
Push environment variables to all machines. All `agents.md` files reference the same variable. Updating the files at that location propagates to all sessions that start after the update.

---

## Important Caveat

Environment variable changes take effect at **session start**, not mid-session. The harness resolves `@-imports` (including variable expansion) once when building the system prompt. A running session continues using the context it started with.

If you change `ENV_VAR_LOC1` mid-session, the current session will not pick it up. The change affects the next session.

---

## Folder Structure

```
3 - environment_variables/
├── README.md
├── Location_1/                         ← one possible target location
├── Location_2/                         ← another possible target location
└── Your_actual_project_folder/
    ├── CLAUDE.md
    ├── agents.md                        ← @-imports via $ENV_VAR_LOC1
    └── agent_teams.md                  ← local fallback copy
```

`agents.md` in the project folder:
```
@'$ENV_VAR_LOC1/agent_teams.md'
@'$ENV_VAR_LOC1/agent_teams_flags.md
**"Send an agent team"** resolves through the Agent Teams framework defined in
  agent_teams.md. Named variants select the matching team by name.
```

---

## Relationship to the Multi-Folder Hierarchy Pattern

Environment variables and the multi-folder hierarchy (Implementation 2) are complementary, not alternatives:

- The hierarchy pattern is better for organizations with a stable folder structure where inheritance and per-scope overrides are valuable.
- The environment variable pattern is better for small teams, flat structures, or cases where the definition location needs to be configurable without changing file content.

Both can be combined: a hierarchical setup can use environment variables at any level to point at external definition files.
