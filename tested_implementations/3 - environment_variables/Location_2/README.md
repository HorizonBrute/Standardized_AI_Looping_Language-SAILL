# Location_2 — Example Team Definition Location

**Location_2** is another possible target location that the environment variable `$ENV_VAR_LOC1` could point to.

---

## What This Folder Represents

This folder contains a complete set of SAILL agent team definitions and flag definitions. It serves as an example of an alternative location that could be used instead of Location_1.

When `ENV_VAR_LOC1` is set to point here, projects using the pattern in `Your_actual_project_folder/` will load their team definitions from this location instead.

---

## What Gets Loaded

When a session starts in `Your_actual_project_folder/` with `ENV_VAR_LOC1=$(pwd)/Location_2`:

```markdown
@'$ENV_VAR_LOC1/agent_teams.md'       # Resolves to Location_2/agent_teams.md
@'$ENV_VAR_LOC1/agent_teams_flags.md' # Resolves to Location_2/agent_teams_flags.md
```

The session context includes:

- **agent_teams.md** — SAILL agent team definitions (Investigate & Fix, Full Team, Review & Fix, Explore & Summarize)
- **agent_teams_flags.md** — Flag definitions for loops, conditionals, and role directives

---

## Files in This Location

- **agents.md** — Standard import file; includes `agent_teams.md` and `agent_teams_flags.md`
- **agent_teams.md** — Core SAILL team definitions
- **agent_teams_flags.md** — Flag syntax and reference for role modifiers (loops, conditionals, etc.)
- **CLAUDE.md** — Session initialization file (imports `agents.md`)

---

## How This Differs from Location_1

Both Location_1 and Location_2 are identical in this demonstration. In a real-world scenario, they might differ in:

- **Different team definitions** (e.g., Location_1 for rapid prototyping with aggressive retries, Location_2 for production with conservative iteration limits)
- **Different role configurations** (e.g., Location_1 uses `#highcap` models, Location_2 uses `#midcost`)
- **Different available teams** (e.g., Location_1 includes experimental teams, Location_2 only stable ones)
- **Different flag vocabularies** (e.g., Location_1 supports additional custom flags)

For this tested implementation, Location_1 and Location_2 are functionally equivalent to demonstrate that the environment variable approach allows switching locations without project file changes.

---

## Testing from This Location

To test with Location_2:

```bash
cd Your_actual_project_folder
export ENV_VAR_LOC1=$(cd ../Location_2 && pwd)
# Start a session here
```

Your session will load definitions from Location_2.

To switch back to Location_1 in a new session:

```bash
export ENV_VAR_LOC1=$(cd ../Location_1 && pwd)
# Start a new session
```
