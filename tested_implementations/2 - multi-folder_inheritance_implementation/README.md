# Tested Implementation 2: Multi-Folder Context Inheritance

## Overview

This implementation demonstrates how SAILL agent definitions and model preferences cascade and stack across a hierarchical directory structure. Each folder level inherits all configuration from its parents, while maintaining the ability to override or extend with scope-local definitions.

**Key concept:** An agent spawned from a leaf project folder automatically has access to all agent teams, flags, and model preferences defined at every ancestor level in the hierarchy.

---

## Folder Hierarchy

```
Org_Root/                                    ← Organization root scope
├── CLAUDE.md
├── agents.md
├── agent_teams.md                           ← Base team definitions
├── agent_team_flags.md                      ← Base flag definitions
├── model_prefs.md                           ← Base model preferences
│
├── Group_A/                                 ← Group scope (inherits all Org_Root)
│   ├── CLAUDE.md
│   ├── agents.md                            ← @-imports local overrides
│   ├── agent_teams.local.md                 ← Group A: add/override teams
│   ├── agent_team_flags.local.md            ← Group A: custom flags
│   ├── model_prefs.local.md                 ← Group A: model choices
│   │
│   ├── GrpA_Team_1/                         ← Team scope (inherits Org_Root + Group_A)
│   │   ├── CLAUDE.md
│   │   ├── agents.md
│   │   ├── agent_teams.local.md             ← Team 1: add/override teams
│   │   ├── agent_team_flags.local.md        ← Team 1: custom flags
│   │   ├── model_prefs.local.md             ← Team 1: model choices
│   │   │
│   │   ├── GrpA_T1_project_1/               ← Project scope (inherits all ancestors)
│   │   │   ├── CLAUDE.md
│   │   │   ├── agents.md
│   │   │   ├── agent_teams.local.md         ← Project-specific teams
│   │   │   ├── agent_team_flags.local.md    ← Project-specific flags
│   │   │   └── model_prefs.local.md         ← Project-specific models
│   │   │
│   │   ├── GrpA_T1_project_2/
│   │   │   └── (annotated as inherits-only, no local extensions)
│   │   │
│   │   └── model_prefs.local.md             ← Team 1 model overrides
│   │
│   ├── GrpA_Team_2/                         ← Second team under Group A
│   │   └── (similar structure)
│   │
│   └── model_prefs.local.md                 ← Group A model overrides
│
├── Group_B/                                 ← Alternative group (inherits Org_Root)
│   ├── CLAUDE.md
│   ├── agents.md
│   │   (no local team/flag files at group level)
│   │
│   ├── GrpB_Team_1/
│   │   └── (similar structure)
│   │
│   └── GrpB_Team_2/
│       └── (similar structure)
│
└── model_prefs.md                           ← Org_Root model preferences
```

---

## What Gets Inherited at Each Level

When a session starts in a project folder, the harness loads context in this order:

### 1. Organization Root (`Org_Root/`)
- `CLAUDE.md` → @-imports `agents.md`
- `agents.md` → @-imports `agent_teams.md`, `agent_team_flags.md`, `model_prefs.md`
- Base definitions of all standard teams and model groups

### 2. Group Level (e.g., `Group_A/`)
- `CLAUDE.md` → @-imports `agents.md`
- `agents.md` → @-imports `agent_teams.local.md`, `agent_team_flags.local.md`, `model_prefs.local.md`
- Group-scoped overrides and additions to teams, flags, and model groups

### 3. Team Level (e.g., `GrpA_Team_1/`)
- `CLAUDE.md` → @-imports `agents.md`
- `agents.md` → @-imports its local override files
- Team-scoped overrides and additions

### 4. Project Level (e.g., `GrpA_T1_project_1/`)
- `CLAUDE.md` → @-imports `agents.md`
- `agents.md` → @-imports its local override files
- Project-scoped overrides and additions

**Result:** An agent at project level sees the union of all inherited definitions, with the most-specific (deepest) scope winning on conflicts.

---

## How local.agent_teams.md Overrides Work

Each `local.agent_teams.md` can:

- **Add new teams** — Teams with novel names are unioned with inherited teams from parent scopes.
- **Override existing teams** — Redefine a team with the same name; the local definition replaces the inherited one at that scope and below.

**Scope precedence:** More specific (deeper) scope always wins. If `Group_A` defines "Deploy" and `GrpA_Team_1` also defines "Deploy", the Team 1 version is active in that subtree. Teams defined at higher scopes but not overridden remain available everywhere below.

**Example:** Org_Root provides "Investigate & Fix" and "Review & Fix" teams. Group_A adds a custom "Parallel Deploy" team. A project in Group_A sees all three teams.

---

## How agent_team_flags.local.md Extensions Work

Custom role flags and loop conditions can be defined at each scope via `local.agent_team_flags.md`.

- **Base flags** (defined in `Org_Root/agent_team_flags.md`): `if needed`, `if asked`, `ask user`, `parallel`, `wait`, loop syntax
- **Local flags** (added at any scope): Extend the vocabulary for teams at that scope and below

**Scope precedence:** Flags from all ancestor scopes are visible. Local flags are unioned; no override semantics.

---

## How model_prefs.local.md Overrides Work

Each `model_prefs.local.md` can configure:

1. **Per-session slots** — Spawned Agent Model, Sub-Agent Override
   - Leave `Unset` to defer to the next precedence level
   - More-specific scope wins if not `Unset`

2. **Model groups** (e.g., `#lowcost`, `#midcost`, `#highcap`, `#investigate`, `#debug`, `#fast`)
   - Members are *combined* across scopes
   - Example: Org_Root adds `claude:haiku` to `#lowcost`; Group_A adds `claude:sonnet` to `#lowcost`
   - Result at Group_A and below: `#lowcost` tries both models in order

3. **Task-class routing** — Map task types to model groups
   - More-specific scope wins on class conflicts

**Merge behavior:** When a session loads context from multiple scopes:
- Slots: Local wins if not "Unset"
- Groups: Membership combined (union)
- Routing: Most specific class wins

---

## Practical Implications

### Inheritance Is Cumulative
An agent working in `GrpA_T1_project_1/` automatically has:
- All teams, flags, and model groups from `Org_Root/`
- All additions/overrides from `Group_A/`
- All additions/overrides from `GrpA_Team_1/`
- All additions/overrides from `GrpA_T1_project_1/`

No explicit "import parent configs" step is needed; the harness walks the ancestor chain automatically.

### Local Overrides Don't Leak Upward
Defining `local.agent_teams.md` at project level does not affect the Team or Group scope. Each scope is independently configured and only affects itself and its descendants.

### Sparse Configurations Are Valid
Not every folder needs every file. For example:
- `GrpA_T1_project_2/` has no `agent_teams.local.md` or `model_prefs.local.md`
  - It simply inherits from all ancestors
  - This is the expected pattern for leaf projects with no custom config
- `Group_B/` has no local overrides at the group level
  - Its teams inherit from `Org_Root/` and apply to all Group_B subfolders

### Annotation Files Track Test Coverage
Two marker files in the implementation confirm test coverage:
- `this_project_inherits_+has_custom_teams.txt` — Project with both inherited and custom teams
- `this_project_has_inherits_flags_and_teams.txt` — Project testing flag inheritance

---

## Key Files at Each Level

### Org_Root/
| File | Purpose |
|------|---------|
| `CLAUDE.md` | Root entrypoint; @-imports agents.md |
| `agents.md` | @-imports agent_teams.md, agent_team_flags.md, model_prefs.md |
| `agent_teams.md` | Shipped team definitions (Investigate & Fix, Full Team, Review & Fix, Explore & Summarize) |
| `agent_team_flags.md` | Base loop and conditional flag vocabulary |
| `model_prefs.md` | Base model groups and per-session slots |

### Group_A/ (and Group_B/)
| File | Purpose |
|------|---------|
| `CLAUDE.md` | Group entrypoint; @-imports agents.md |
| `agents.md` | @-imports local override files |
| `agent_teams.local.md` | Group-scoped team additions or overrides |
| `agent_team_flags.local.md` | Group-scoped flag extensions |
| `model_prefs.local.md` | Group-scoped model group customization and routing |

### Team Level (GrpA_Team_1/, GrpA_Team_2/, etc.)
Same structure as Group level, allowing team-specific customization.

### Project Level (GrpA_T1_project_1/, etc.)
Same structure as Team level, allowing project-specific customization.

---

## How to Verify Inheritance

### From a Project Folder (e.g., `GrpA_T1_project_1/`)

1. **Check context load** — Run the `/context-cost` command:
   - Should list files from all ancestor levels
   - Example output shows `Org_Root/agents.md`, `Group_A/agents.md`, `GrpA_Team_1/agents.md`, and the project's own `agents.md`

2. **Check available teams** — Run the `/agent-teams` command:
   - Displays teams from `Org_Root/agent_teams.md`
   - Plus any additions from `Group_A/agent_teams.local.md`
   - Plus any additions from `GrpA_Team_1/agent_teams.local.md`
   - Plus any additions from `GrpA_T1_project_1/agent_teams.local.md`
   - Overrides are shown with context about scope

3. **Use resolve_agent_teams.py** — Run from the project folder:
   ```
   resolve_agent_teams.py --json
   ```
   - Shows the full source chain for each team
   - Confirms which definition wins and where it comes from
   - Useful for debugging precedence issues

4. **Check model groups** — Model preferences cascade the same way; a custom model group added at Group_A level appears in all descendent projects alongside Org_Root groups.

---

## Testing Notes

- **Group_A and Group_B** provide parallel hierarchies to verify that group-level overrides are scope-isolated
- **Multiple projects per team** (project_1 and project_2) test both "with local overrides" and "inherit-only" patterns
- **Team-level model_prefs.local.md** at `GrpA_Team_1/` and `GrpA_Team_2/` test that intermediate levels (Team) can override independently of their Group and Projects

---

## Implementation Details

All `CLAUDE.md` files use `@agents.md` imports for reliability (tested during June 2026 — note that direct `@agent_teams.md` imports in `CLAUDE.md` proved inconsistent). The `agents.md` at each level handles the actual imports of override files.

Each scope's `agents.md` follows this pattern:
```
@agent_teams.local.md        # Load Agent Teams definitions
@agent_team_flags.local.md   # Load Looping Definitions
@model_prefs.local.md        # Load Model preferences local overrides
```

This ensures that override files are loaded in addition to (not replacing) inherited definitions.
