# Location_1 — Example Team Definition Location

**Location_1** is one possible target location that the environment variable `$ENV_VAR_LOC1` could point to.

---

## What This Folder Represents

This folder contains a complete set of SAILL agent team definitions and flag definitions. It serves as an example of what a centralized or shared definition location might look like.

When `ENV_VAR_LOC1` is set to point here, projects using the pattern in `Your_actual_project_folder/` will load their team definitions from this location.

---

## What Gets Loaded

When a session starts in `Your_actual_project_folder/` with `ENV_VAR_LOC1=$(pwd)/Location_1`:

```markdown
@'$ENV_VAR_LOC1/agent_teams.md'       # Resolves to Location_1/agent_teams.md
@'$ENV_VAR_LOC1/agent_teams_flags.md' # Resolves to Location_1/agent_teams_flags.md
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

## How This Differs from Location_2

Both Location_1 and Location_2 are identical in this demonstration. In a real-world scenario, they might differ in:

- **Different team definitions** (e.g., one has aggressive looping, the other conservative)
- **Different flags or role modifiers** (e.g., different available role flags)
- **Different scope precedence rules** (e.g., per-scope model preferences)
- **Location_1 for development**, Location_2 for production

For this tested implementation, they serve as proof that the environment variable pattern allows choosing between multiple locations without editing the project.

---

## Testing from This Location

To test with Location_1:

```bash
cd Your_actual_project_folder
export ENV_VAR_LOC1=$(cd ../Location_1 && pwd)
# Start a session here
```

Your session will load definitions from Location_1.
