# SAILL Implementation 1: Single-Folder Basic Setup

This implementation demonstrates the minimal set of files needed to define and invoke SAILL agent teams and model preferences. All files are in a single directory and wired together through `CLAUDE.md`.

---

## What This Implementation Demonstrates

- The minimum viable configuration for SAILL: one directory with five files
- How `CLAUDE.md` serves as the entry point and loads supporting files via `@-imports`
- The agent team definition format: team names, roles, model groups, and looping/conditional syntax
- The model preference system: named groups and resolution order
- A known limitation: `@-imports` declared inside `agents.md` are not reliably resolved by the harness; only direct `CLAUDE.md` imports work

---

## File Structure

```
1 - single-folder_basic_implementation/
├── CLAUDE.md                  # Harness entry point; loads all needed files
├── agents.md                  # Documentation and redundant imports
├── agent_teams.md             # Shipped team definitions
├── agent_team_flags.md        # SAILL flag vocabulary
└── model_prefs.md             # Model group routing rules
```

---

## File Purposes

### CLAUDE.md

**Role:** Harness entry point. The Claude Code harness reads this file and resolves all `@-import` lines.

**Contents:**
```
@agents.md
@agent_teams_flags.md
**"Send an agent team"** resolves through ...
```

**Why it exists:**
- The harness only resolves `@-imports` from `CLAUDE.md`, not from files imported by `agents.md`
- This file must explicitly `@-import` any file that needs to be in context for every session
- Inline prose like `**"Send an agent team"**` documents the user-facing command (natural language cue)

**Implementation note:** Lines prefixed with `#` are comments. Line 1 attempts `@agents.md` but the comment warns of unreliability.

### agents.md

**Role:** Documentation file; should (but does not reliably) re-import team and model files.

**Contents:**
- `@agent_teams.md` — references the team definitions
- `@agent_teams_flags.md` — references the flag vocabulary
- `@./model_prefs.md` — references the model groups
- Inline prose repeating the `**"Send an agent team"**` instruction

**Why it exists:**
- Intended to mirror `CLAUDE.md` for clarity and maintainability
- Serves as documentation of which files the system needs
- Currently unreliable: the harness does not honor `@-imports` declared inside `agents.md` (observed June 2026, Claude Code harness behavior)
- Workaround: all imports must be declared directly in `CLAUDE.md`

**Limitation:** This is a known issue during SAILL development. The intention is that `agents.md` would be the canonical source of imports, with `CLAUDE.md` simply aliasing it; in practice, only `CLAUDE.md` imports are resolved.

### agent_teams.md

**Role:** Defines the four shipped starter teams and the SAILL team syntax specification.

**Contents:**
1. **Investigate & Fix** — diagnose root cause, then apply the fix
   - Investigate (`#midcost`)
   - Fix (`#lowcost`)

2. **Full Team** — complete lifecycle for a sizable or ambiguous task (default)
   - Orchestrator (`#highcap`)
   - Log-reader (`#lowcost`, if needed)
   - Planner (`#highcap`)
   - Implementer (`#lowcost`)
   - Validator (`#midcost`, if asked) — with loop: return feedback to Implementer on failure, cap at 3 iterations

3. **Review & Fix** — audit a diff then apply findings
   - Reviewer (`#highcap`)
   - Fixer (`#lowcost`)

4. **Explore & Summarize** — fan out and distill findings
   - Explorer (`#investigate`)
   - Summarizer (`#lowcost`)

**Syntax reference included:**
- **Loops:** `Loop: on <condition>, return feedback to <role> and re-run from "<role name>"; repeat until <pass condition> or <max> iterations, then <action at cap>`
- **Conditional roles:** `(<group>, if needed)` or `(<group>, if asked)`
- **Sub-teams (boxes):** `[ … ]` to bundle roles into a unit
- **Context placeholders:** `-context-` or `-context:<name>-` for values drawn from context

**When invoked:**
- User says "send an agent team" (or a team name like "investigate-and-fix")
- The acting model looks up the team name in this file
- The model spawns each role in sequence on the appropriate model group

### agent_team_flags.md

**Role:** Catalog of SAILL primitives and flag vocabulary.

**Contents:**
- Flag form: inline (declared on a role line) or annotation (declared separately)
- Each flag's meaning and interaction with other flags
- Examples: `if needed`, `if asked`, `ask user`, `parallel`, `wait`, `Loop`, `if fail`

**When used:**
- The acting model consults this file when interpreting team definitions
- Users can extend with custom flags in `local.agent_team_flags.md` (gitignored)

**Usage:** Run `resolve_agent_teams.py --flags` to list all available flags.

### model_prefs.md

**Role:** Defines model groups and resolution priority for agent spawning.

**Contents:**
- **Per-Session Slots:** "Spawned Agent Model" and "Sub-Agent Override" (both Unset in base file)
- **Model Groups:** `#lowcost`, `#midcost`, `#highcap`, `#investigate`, `#debug`, `#fast` (membership undefined in base file)
- **Task-Class Routing:** (undefined in base file)
- **Merge Rules:** how local overrides combine with base files across scopes

**Resolution order** (when spawning an agent):
1. Named group from the prompt (e.g., "on `#highcap`")
2. Task-class routing match
3. Sub-agent override (if set)
4. Spawned agent model (if set)
5. Harness or provider default

**Base vs. local:**
- Base file defines the schema and rules; group membership is "Unset"
- Edit `model_prefs.local.md` (gitignored) to fill in group members (e.g., `#lowcost: claude-haiku, claude-3-5-sonnet`)

---

## How Files Work Together

### Loading Order

When Claude Code opens a session in this directory:

1. **Harness reads `CLAUDE.md`** and begins resolving `@-imports`
2. **Loads `agents.md`** (line 1: `@agents.md`) — but its own `@-imports` are NOT resolved
3. **Loads `agent_teams_flags.md`** (line 2: `@agent_teams_flags.md`)
4. All three files are now in context

### What Gets Into Context

- `CLAUDE.md` itself (entry point)
- `agents.md` (imported directly)
- `agent_teams_flags.md` (imported directly)
- `agent_teams.md` — **NOT automatically loaded** (imported only inside `agents.md`, which is unreliable)
- `model_prefs.md` — **NOT automatically loaded** (imported only inside `agents.md`, which is unreliable)

### Workaround: Reference on Demand

The `CLAUDE.md` comment says:
> Save context, you can instruct the agent to reference the file if it needs.

When an agent team is invoked:
- The user (or the acting model) says "send an Investigate & Fix team"
- The acting model is instructed to look up the team in `agent_teams.md` (which is NOT in context by default)
- The model reads the file explicitly if needed, reducing baseline context load

---

## Known Limitation: @-imports in agents.md

**Finding (June 2026):**
The Claude Code harness resolves `@-imports` only from `CLAUDE.md`. If `agents.md` contains:
```
@agent_teams.md
@./model_prefs.md
```

These imports are **not resolved**. The files are not loaded into context.

**Why it matters:**
- `agent_teams.md` and `model_prefs.md` are large reference files; having them always in context is expensive
- The intention was to keep `agents.md` as the canonical import manifest, with `CLAUDE.md` simply aliasing it
- This limitation forces `CLAUDE.md` to repeat imports or omit them entirely

**Current workaround:**
- Import only small, essential files in `CLAUDE.md` (e.g., `agents.md`, `agent_teams_flags.md`)
- Document other imports in `agents.md` (as done here)
- Instruct the acting model to reference files on demand when interpreting team definitions

**Future:** This limitation is expected to be resolved in a later harness version.

---

## How to Verify It Works

### 1. Confirm files load correctly

Run `/context-cost` from this directory. Output should list:
- `CLAUDE.md`
- `agents.md`
- `agent_teams_flags.md`

(And **not** automatically list `agent_teams.md` or `model_prefs.md`, which load on demand.)

### 2. List available teams

Run `/agent-teams` (bare command). Output shows:
- All four shipped teams
- Their roles and model-group assignments
- Source file: `agent_teams.md`

### 3. Test team execution

Run `/test-agent-teams` and select a team. The skill:
- Spawns each role as a real sub-agent
- Each role echoes a test nonce (proof of execution)
- Verifies each role ran on a model matching its `#group`

> Warning: `/test-agent-teams` spawns real agents and may incur API cost.

### 4. Invoke a team manually

In a session with this directory as working directory:

> "Send an investigate-and-fix team to look at the failing integration test."

The model spawns:
1. **Investigator** role on the `#midcost` model group
2. **Fixer** role on the `#lowcost` model group

Each role receives the request, executes its charter, and passes results to the next role.

---

## File Listing

| File | Lines | Purpose |
|------|-------|---------|
| `CLAUDE.md` | ~5 | Harness entry point; imports essential files |
| `agents.md` | ~6 | Documentation of intended imports and command invocation |
| `agent_teams.md` | ~112 | Team definitions (Investigate & Fix, Full Team, Review & Fix, Explore & Summarize) and loop/box syntax |
| `agent_team_flags.md` | (reference) | SAILL flag vocabulary and meanings |
| `model_prefs.md` | ~53 | Model group schema, resolution order, and merge rules |
| `README.md` | (this file) | Implementation guide and known limitations |

---

## Next Steps

- **To extend:** Edit `local.agent_teams.md` (gitignored) to add custom teams or override shipped ones
- **To customize models:** Edit `model_prefs.local.md` (gitignored) to assign model groups (e.g., `#lowcost: haiku, 3.5-sonnet`)
- **To learn more:** See the documentation index at `../documentation/` for deeper dives into loops, conditional roles, and box syntax

