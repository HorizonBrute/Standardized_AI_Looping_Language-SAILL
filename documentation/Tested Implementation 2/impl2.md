# Tested Implementation 2 — Multi-Folder Context Inheritance

This implementation demonstrates how SAILL files stack across a directory hierarchy. Each level of the tree inherits everything from above it, and can override or extend it with local files.

**Location:** `tested_implementations/2 - multi-folder_inheritance_implementation/`

---

## What This Implementation Demonstrates

- How SAILL files cascade from an organization root down through groups, teams, and projects
- That an agent working in a leaf project folder inherits all definitions from every ancestor
- How `local.agent_teams.md` and `model_prefs.local.md` at each scope override without affecting other scopes
- How flag customizations (`local.agent_team_flags.md`) can be scoped

---

## The Folder Hierarchy

```
Org_Root/
├── CLAUDE.md
├── agents.md
├── agent_teams.md
├── agent_team_flags.md
├── model_prefs.md
├── Group_A/
│   ├── CLAUDE.md
│   ├── agents.md
│   ├── agent_teams.local.md        ← Group A's team overrides/additions
│   ├── agent_team_flags.local.md   ← Group A's flag additions
│   ├── model_prefs.local.md        ← Group A's model choices
│   ├── GrpA_Team_1/
│   │   ├── CLAUDE.md
│   │   ├── agents.md
│   │   ├── agent_teams.local.md
│   │   ├── agent_team_flags.local.md
│   │   ├── model_prefs.local.md
│   │   ├── GrpA_T1_project_1/
│   │   │   ├── CLAUDE.md
│   │   │   ├── agents.md
│   │   │   ├── agent_teams.local.md
│   │   │   ├── agent_team_flags.local.md
│   │   │   └── model_prefs.local.md
│   │   └── GrpA_T1_project_2/
│   │       └── (same structure)
│   └── GrpA_Team_2/
│       └── (same structure)
└── Group_B/
    ├── CLAUDE.md
    ├── agents.md
    └── (subfolders — some with local overrides, some without)
```

---

## What Gets Inherited at Each Level

Context stacks. When a session starts in `GrpA_T1_project_1/`, the harness loads:

1. `Org_Root/CLAUDE.md` → imports `agent_teams.md`, `agent_team_flags.md`, `model_prefs.md`
2. `Org_Root/Group_A/CLAUDE.md` → imports `agent_teams.local.md`, `model_prefs.local.md`, `agent_team_flags.local.md`
3. `Org_Root/Group_A/GrpA_Team_1/CLAUDE.md` → imports its local override files
4. `Org_Root/Group_A/GrpA_Team_1/GrpA_T1_project_1/CLAUDE.md` → imports its local override files

All four layers are in the system prompt. The project-level agent has the full context of every ancestor.

---

## How local.agent_teams.md Overrides Work

At each level, `local.agent_teams.md` can:
- **Override** a shipped team by defining a team with the same name — the local definition wins
- **Add** a custom team by defining a team with a new name — unioned with inherited teams

Only the most-specific definition of a given team name is active. If `Group_A` defines a custom "Deploy & Verify" team and `GrpA_Team_1` also defines "Deploy & Verify" with different roles, `GrpA_Team_1`'s version wins for sessions in that subtree.

Teams defined at higher scopes but not overridden at lower scopes remain available everywhere below them.

---

## How model_prefs.local.md Overrides Work

Each scope's `model_prefs.local.md` can:
- **Set per-session slots** — Spawned Agent Model, Sub-Agent Override (more-specific wins if not `Unset`)
- **Add group members** — membership is combined across scopes
- **Add routing rules** — more-specific scope wins on conflict

Example: Org_Root assigns `claude:haiku` to `#lowcost`. Group_A adds `claude:sonnet` to `#lowcost`. At Group_A scope, `#lowcost` tries `claude:haiku` first, then `claude:sonnet`.

---

## Practical Implications

**Group_B observed behavior:** The structure shows that Group_B teams and some of its projects do not have `local.agent_team_flags.md` or `model_prefs.local.md`. Those scopes simply inherit from `Org_Root` without any local customization — which is valid and expected. Local override files are optional at every level.

**"this_project_inherits_+has_custom_teams.txt" and "this_project_has_inherits_flags_and_teams.txt"** — these annotation files in the implementation mark which projects demonstrate team addition vs. flag addition, confirming that both types of extension are tested in the same hierarchy.

---

## How to Verify Inheritance

From any project-level folder (e.g., `GrpA_T1_project_1/`):

1. Run `/context-cost` — confirm files from all ancestor levels appear in the load list.
2. Run `/agent-teams` — confirm teams from `Org_Root` are visible alongside any local additions.
3. Run `resolve_agent_teams.py` from the project folder — it walks the cascade and shows which definition of each team wins and where it came from.

The `--json` flag on `resolve_agent_teams.py` shows the full source chain per team.
