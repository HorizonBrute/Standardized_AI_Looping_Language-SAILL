# SAILL Skills Index

| Skill | Description |
|---|---|
| [agent-teams](agent-teams/SKILL.md) | Create or edit agent-team definitions in local.agent_teams.md at any scope (OS, project, brain, or subfolder) — define the agent chain, each role's model-group preference, and loop/retry constructs. |
| [convert-prompt-to-saill](convert-prompt-to-saill/SKILL.md) | Convert a natural-language prompt into a SAILL agent-team flow — roles with model groups, control-flow flags (parallel/wait/Loop/if needed/if asked/ask user), boxes for sub-teams and nesting, and -context- for run-time values. |
| [model-catalog-refresh](model-catalog-refresh/SKILL.md) | Fetch CURRENT model information from live provider docs (Anthropic, OpenAI, Google Gemini, Ollama) and return a structured catalog the user can use to populate or validate the Horizon AIOS model-preference config. |
| [model-prefs](model-prefs/SKILL.md) | Configure or inspect the in-context model-preference layer — model groups (incl. local/BYO models), per-session slots, and task-class routing — by editing the gitignored extend file. |
| [model-prefs-assign](model-prefs-assign/SKILL.md) | Audit AIOS skills for their model-preference group callout and assign or refresh it — add a "Model preference" line to skills that lack one, change the group where behavior should differ, and keep the skill indexes' Model-group columns in sync. |
| [model-prefs-test](model-prefs-test/SKILL.md) | Test which model the model-preference config actually resolves to per group, and (in live mode) spawn small test agents by group name to confirm the spawn honors it. |
| [saill-to-english](saill-to-english/SKILL.md) | Translate a SAILL agent-team definition into plain English — render every primitive, loop, box, flag, and -context- placeholder as natural language a non-technical reader can follow. |
| [test-agent-teams](test-agent-teams/SKILL.md) | End-to-end self-test of the Agent Teams system — walk every defined team (or one named team) and spawn each role so it echoes a self-generated nonce, its role, what it was told to do (its charter), and the model it actually ran as. |
