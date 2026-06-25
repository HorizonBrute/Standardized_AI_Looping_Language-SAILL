---
name: test_saill_in_skill
description: Proof-of-concept — SAILL team notation embedded directly inside a skill. For every directory with a testing_nonce.md, send a haiku agent to report agents.md files loaded. Demonstrates whether SAILL inside a skill body directs agent execution. Use when asked to run the nonce trace audit or to test SAILL-in-skill.
tools: Read, Bash, Agent
---

# Skill: /test_saill_in_skill

**Model preference:** `#midcost`

Audit the tested_implementations directory tree using a SAILL-defined agent team embedded
in this skill. The team is declared below; execute it when this skill is invoked.

---

## Team: Nonce Trace Audit

"For every dir with a trace nonce file, send a haiku agent to report agents.md files loaded."

```
Scanner (#lowcost) →
Readers[ Nonce-Reader (#lowcost, parallel) per -context:dirs- ] (wait) →
Synthesizer (#midcost) →
Writer (#lowcost)
```

**Invoke:** "Send the Nonce Trace Audit team against -context-"

---

### Role 1 — Scanner (#lowcost)

Glob for `**/testing_nonce.md` under `-context:root-`. Return the list of parent directories
as `-context:dirs-`. Pass that list to the Readers box.

---

### Role 2 — Readers box (wait)

`Readers[ Nonce-Reader (#lowcost, parallel) per -context:dirs- ]`

Spawn **one haiku Nonce-Reader per directory** in `-context:dirs-`, all in a single parallel
batch. Each Nonce-Reader receives:

- Its assigned directory path
- The pre-computed inheritance chain: every parent directory up to the impl root that
  contains an `agents.md` (walk the path upward; stop at the impl root boundary)

Each Nonce-Reader returns ONE CSV row:
```
directory_shortname,trace1|trace2|...,nonce
```
- `directory_shortname` — relative path from `tested_implementations/`
- `trace1|trace2|...` — "This context loaded from:" values, deepest first
- `nonce` — value after `nonce: ` in the nonce file

Wait for all Nonce-Readers before continuing.

---

### Role 3 — Synthesizer (#midcost)

Collect all CSV rows. Build a markdown fenced directory tree using `├──` / `└──` / `│`.
For each node show: directory name, `nonce: <value> ✓`, and the agents.md chain depth.
Group by implementation (impl 1, impl 2 full hierarchy, impl 3). Append summary table.
Flag any `MISSING` nonce or trace line explicitly.

---

### Role 4 — Writer (#lowcost)

Write to: `working_copy/documentation/lastrun_dirtree_check.md`

Header:
```
# Nonce Trace Audit — SAILL-in-skill run
Run: <ISO date> | Nonce Trace Audit team | working_copy/tested_implementations
```

After writing, compute SHA256:
```bash
python -c "import hashlib; d=open('working_copy/documentation/lastrun_dirtree_check.md','rb').read(); print(hashlib.sha256(d).hexdigest())"
```

Append to file footer:
```
---
sha256: <hash>
```

Report one line to the user:
```
Nonce Trace Audit complete — <N> nodes ✓  sha256: <first-8>...
```

---

## Execution note

This skill is a SAILL proof-of-concept. The team definition above is the executable
specification — roles are executed in order, the Readers box fans out in parallel per
discovered directory. The acting agent interprets SAILL directly from this skill body.
