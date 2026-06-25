# Tested Implementation 3 — Environment Variables in Agent Team Files

This implementation demonstrates using environment variables as `@-import` paths in `agents.md`, enabling a single project to load SAILL definitions from different locations without editing files. Team definitions are resolved at session start based on environment variable expansion.

---

## What This Demonstrates

- Using shell environment variables in `@-import` paths to reference SAILL files
- Loading agent teams and flags from a configurable location instead of a fixed relative path
- Switching between different team-definition sets by changing an environment variable
- Portable, shareable agent team definitions across multiple projects

---

## The Problem It Solves

In a standard setup, `agents.md` imports files by relative path:

```markdown
@agent_teams.md
@agent_teams_flags.md
```

This works when files are co-located with the project, but it couples the project tightly to a specific folder structure. If you want to:

- **Share team definitions** across multiple projects without copying or symlinking
- **Centrally manage** team definitions that all projects reference
- **Switch between environments** (e.g., local teams vs. organization-wide teams) without editing project files
- **Deploy to different machines** where the definition location differs

...then hardcoded relative paths require editing files in every affected project.

---

## How It Works

The project's `agents.md` references files via an environment variable:

```markdown
@'$ENV_VAR_LOC1/agent_teams.md'
@'$ENV_VAR_LOC1/agent_teams_flags.md'
```

Before starting a session, set the environment variable to point at a location:

```bash
export ENV_VAR_LOC1=/path/to/Location_1
# or
export ENV_VAR_LOC1=/path/to/Location_2
# or any other location containing these files
```

The session harness resolves `$ENV_VAR_LOC1` at load time and inlines the files from that location into the session context. The project's `agents.md` stays unchanged.

### In This Implementation

- **Location_1/** and **Location_2/** are two example locations that `$ENV_VAR_LOC1` might point to
- Both contain identical `agent_teams.md` and `agent_teams_flags.md` files
- **Your_actual_project_folder/** is the project that uses the environment variable to load from either location
- Switching between locations requires only changing `ENV_VAR_LOC1` and starting a new session

---

## Folder Structure

```
3 - environment_variables/
├── README.md                          ← You are here
├── Location_1/
│   ├── README.md
│   ├── agents.md
│   ├── agent_teams.md
│   ├── agent_teams_flags.md
│   └── CLAUDE.md
├── Location_2/
│   ├── README.md
│   ├── agents.md
│   ├── agent_teams.md
│   ├── agent_teams_flags.md
│   └── CLAUDE.md
└── Your_actual_project_folder/
    ├── CLAUDE.md
    ├── agents.md                     ← Uses $ENV_VAR_LOC1 in @-imports
    └── agent_teams.md                ← Local fallback copy
```

---

## When to Use This Pattern

**Small teams sharing definitions without a project hierarchy:**
Set a shared environment variable pointing to a central location (e.g., a shared network folder or a team repository). All team members set `ENV_VAR_LOC1` locally and point their projects at the same shared files.

**Switching between definition sets:**
Maintain multiple locations with different `agent_teams.md` definitions for different use cases (e.g., staging teams vs. production teams). Switch by changing `ENV_VAR_LOC1` and starting a new session.

**Organizational deployments:**
Push environment variables to all machines as part of onboarding or infrastructure setup. All projects reference the same variable; updating the files at that location propagates to all new sessions.

**Avoiding hierarchical structures:**
If your organization is flat or ad-hoc, environment variables avoid the overhead of a rigid multi-folder hierarchy while still achieving centralized definition management.

---

## Important Caveat: Context Freezes at Session Start

Environment variable changes take effect **only at session start**, not mid-session.

The harness resolves all `@-imports` (including variable expansion) once when building the system prompt at session initialization. A running session continues with the context it started with, even if you change `ENV_VAR_LOC1` in your shell.

**Example:**
1. Start a session with `ENV_VAR_LOC1=/location/A`
2. Mid-session, change it: `export ENV_VAR_LOC1=/location/B`
3. The current session still uses definitions from `/location/A`
4. The next session (in a new shell window or after closing this one) uses `/location/B`

This is by design — it prevents context drift mid-conversation and ensures consistent behavior within a session.

---

## How to Test This Implementation

1. **Set the environment variable:**
   ```bash
   export ENV_VAR_LOC1=$(pwd)/Location_1
   ```

2. **Start a session in `Your_actual_project_folder/`:**
   Open the project in your editor or start a session there. The harness loads `agents.md`, expands `$ENV_VAR_LOC1`, and inlines the files from `Location_1/`.

3. **Verify the import resolved correctly:**
   Check your session's loaded context. The agent teams and flags from `Location_1/agent_teams.md` and `Location_1/agent_teams_flags.md` should be available.

4. **Switch locations (in a new session):**
   ```bash
   export ENV_VAR_LOC1=$(pwd)/Location_2
   ```
   Start a new session. You'll now be using definitions from `Location_2/`.

5. **Confirm the difference:**
   If `Location_1/` and `Location_2/` have different `agent_teams.md` content, you'll see the difference in the new session's available teams.

---

## Relationship to Other Patterns

**vs. Multi-Folder Hierarchy (Implementation 2):**
- Hierarchy is better for organizations with a stable folder structure where inheritance and per-scope overrides are valuable.
- Environment variables are better for flat structures, small teams, or when definition location needs to be configurable without editing files.
- Both can be combined: a hierarchical setup can use environment variables at any level.

**vs. Literal Relative Paths:**
- Relative paths are simpler for self-contained projects with no shared definitions.
- Environment variables add complexity but solve the portability and centralization problems.

---

## Edge Cases and Limitations

- **Variable not set:** If `ENV_VAR_LOC1` is not defined, the import will fail. Set a sensible default or provide setup documentation.
- **Path format:** Use quotes in the `@-import` path when the variable contains spaces or special characters: `@'$ENV_VAR_LOC1/agent_teams.md'`
- **Non-POSIX shells:** On Windows without a POSIX shell, use the appropriate syntax for your environment (e.g., PowerShell `$env:VAR_NAME` or `set VAR_NAME=value` in cmd.exe).
- **Reliability note:** As of June 2026, transitive imports through CLAUDE.md have shown unreliability. The safer approach is to import directly in `agents.md` and rely on context instructions to reference files if needed.
