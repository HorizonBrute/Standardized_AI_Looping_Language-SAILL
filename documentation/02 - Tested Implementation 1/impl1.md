# Tested Implementation 1 — Single-Folder Basic Setup

This implementation demonstrates the minimal SAILL setup: all files in one directory, loaded by a single `CLAUDE.md`.

**Location:** `tested_implementations/1 - single-folder_basic_implementation/`

---

## What This Implementation Demonstrates

- The minimum set of files needed to use agent teams and model preferences
- How `CLAUDE.md` wires the files together via `@-imports`
- How `agents.md` loads the SAILL files, and `CLAUDE.md` loads `agents.md`

---

## Files Involved

```
1 - single-folder_basic_implementation/
├── CLAUDE.md
├── agents.md
├── agent_teams.md
├── agent_team_flags.md
└── model_prefs.md
```

---

## What Each File Does

### CLAUDE.md

The harness entry point. Loads `agents.md`, which in turn loads the SAILL files via recursive `@-import` resolution.

```
@agents.md
**"Send an agent team"** resolves through the Agent Teams framework defined in
  agent_teams.md. Named variants select the matching team by name.
```

When `CLAUDE.md` `@-imports` `agents.md`, the harness resolves `agents.md`'s own `@-imports` recursively — so `agent_teams.md`, `agent_team_flags.md`, and `model_prefs.md` all load automatically. See [How it Works](../06%20-%20How%20it%20Works/how_it_works.md) for the resolution chain.

### agents.md

Loads team and model files into context and provides the natural-language invocation instruction:

```
@agent_teams.md        # Load Agent Teams definitions
@agent_team_flags.md  # Load flag definitions
**"Send an agent team"** resolves through the Agent Teams framework defined in
  agent_teams.md. Named variants select the matching team by name.
@./model_prefs.md      # Load Model Preferences
```

### agent_teams.md

Defines the four shipped starter teams: **Investigate & Fix**, **Full Team**, **Review & Fix**, **Explore & Summarize**. Also defines the loop syntax, conditional role syntax, sub-teams (boxes), and `-context-` values.

This is the file invoked when you say "send an agent team" in a session. The acting model looks up the requested team name in this file.

See [Agent Groups](../03%20-%20Agent%20Groups/agent_groups.md) for the full team definitions and format reference.

### agent_team_flags.md

The flag vocabulary catalog. Lists each SAILL primitive with its form (inline or annotation) and meaning. The acting model uses this to interpret flags in team definitions.

### model_prefs.md

Declares the model groups (`#lowcost`, `#midcost`, `#highcap`, `#investigate`, `#debug`, `#fast`) and the resolution order. Group membership is `Unset` in the base file — fill it in `model_prefs.local.md` (gitignored).

---

## How to Verify It Works

### 1. Confirm context loads

Run `/context-cost` from this directory. The output should show `CLAUDE.md`, `agent_teams.md`, `agent_team_flags.md`, and `model_prefs.md` in the loaded file list.

### 2. Inspect active teams

Run `/agent-teams` (bare). The skill runs `resolve_agent_teams.py` and displays the teams currently in effect, their roles, and their sources.

Expected output: all four shipped teams visible, sourced from `agent_teams.md`.

### 3. Test spawning

Run `/test-agent-teams` and choose a team. The skill spawns each role as a real sub-agent and verifies:
- Each role echoed the nonce (proof of actual execution)
- Each role ran on a model consistent with its assigned `#group`

> Note: `/test-agent-teams` spawns real agents — it is a deliberate, possibly costly integration test.

### 4. Invoke a team manually

In a session with this directory as CWD:

> "Send an investigate-and-fix team to look at the failing test."

The acting model should spawn an Investigator on `#midcost`, then a Fixer on `#lowcost`.

