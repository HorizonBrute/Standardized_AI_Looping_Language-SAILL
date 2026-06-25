# SAILL Example Loops

Eleven self-contained team definitions illustrating the full primitive set. Each is a copy-paste starting point — drop into `agent_teams.md` or `local.agent_teams.md` and invoke by name.

Each example includes a **Human prompt** — the natural-language request that this SAILL definition encodes. This is what you might type or process regularly enough to define an agent team and call it by name — "Send the XYZ team" — saving token cost and giving your team a shared, standard way to invoke the same workflow.

Each example also includes a **Prompt change after implementation** section showing how the invocation shrinks once the team is defined.

Model groups reference the standard tiers: `#lowcost`, `#midcost`, `#highcap`, `#investigate`. Replace role labels and charters to fit your workflow; the control-flow structure stays portable. Add or remove models/groups in `model_prefs.md` using the appropriate skill.

---

## 1. Diagnose & Patch

Minimal two-role sequential flow. No conditions, no retries — the simplest shareable unit.

> **Human prompt:** "Look into why the auth service is crashing and fix it."

```
### Diagnose & Patch

1. Diagnoser (#midcost) — identifies root cause and hands a precise diagnosis to Patcher.
2. Patcher (#lowcost) — applies the fix described by Diagnoser and confirms resolution.
```

**Prompt change after implementation:**
> "Send the Diagnose & Patch team at the auth service crash."

---

## 2. Draft, Review, Publish — with User Gate

Three-role sequence with an `ask user` gate before the final step. Useful whenever publishing or deploying is irreversible and should not happen automatically.

> **Human prompt:** "Write a release announcement, have someone check it, then post it — but ask me before you actually publish."

```
### Draft, Review, Publish

1. Drafter (#lowcost) — produces the initial draft from the provided brief.
2. Reviewer (#midcost) — audits the draft for correctness and completeness; hands edits to Drafter if needed.
3. Publisher (#lowcost, ask user) — waits for user approval, then executes the publish step.
```

**Prompt change after implementation:**
> "Send the Draft, Review, Publish team for the v2.4 release notes."

---

## 3. Audit & Fix with Retry Loop

Implement/validate pattern with a three-iteration retry loop. The Validator feeds specific failures back to the Implementer; the cap surfaces any unresolved issues rather than silently proceeding.

> **Human prompt:** "Apply the linting fixes and keep retrying until the suite is clean — give up after 3 attempts and tell me what's still broken."

```
### Audit & Fix

1. Implementer (#lowcost) — applies the change.
2. Validator (#midcost) — runs the audit suite against the change.
   **Loop:** on failure, return specific findings to "Implementer" and re-run from there;
   repeat until the Validator passes clean or 3 iterations, then stop and report
   any outstanding failures.
```

**Prompt change after implementation:**
> "Send the Audit & Fix team at the current diff."

---

## 4. Parallel Recon, then Synthesize

Fan out across multiple sources concurrently, then merge findings. The `wait` on the Recon box syncs before Synthesizer runs. Add or remove crawlers inside `Recon[ … ]` without touching the rest of the definition.

> **Human prompt:** "Pull data from the API, the database, and the UI traces all at once, then give me one combined report."

```
### Parallel Recon & Synthesize

1. Orchestrator (#highcap) — defines the investigation scope and fans out.
2. Recon[ SourceA (#investigate, parallel),
          SourceB (#investigate, parallel),
          SourceC (#investigate, parallel) ] (wait)
3. Synthesizer (#midcost) — merges all findings into a single actionable report.
```

**Prompt change after implementation:**
> "Send the Parallel Recon & Synthesize team across the payment service sources."

---

## 5. Context-Driven Loop with Variable Cap

Loop where both the pass condition and the iteration cap come from the invocation context rather than being hardcoded. Keeps the team definition generic and reusable across projects with different quality bars.

> **Human prompt:** "Keep refining this output until it meets the bar we discussed — you decide when it's good enough, but don't go forever."

```
### Iterative Refine

1. Producer (#lowcost) — generates the artifact from the provided brief.
2. Critic (#midcost) — evaluates the artifact against -context:pass criteria-.
   **Loop:** on failure, return specific feedback to "Producer" and re-run from there;
   repeat until -context:pass criteria- or -context:cap- iterations, then report failures.
```

**Prompt change after implementation:**
> "Send the Iterative Refine team — pass criteria: no hallucinations, cap: 4."

---

## 6. Compound Loop Exit — Pass, User Override, or Cap

Loop that exits on whichever comes first: a clean pass, the user choosing to stop early, or hitting the iteration cap. Useful for long-running quality cycles where the user may want to accept partial results mid-flight.

> **Human prompt:** "Run quality checks and keep fixing until it passes, but I want to be able to stop it early if I decide good enough is good enough. Hard cap at 5 tries."

```
### Iterative Quality Pass

1. Implementer (#lowcost) — applies changes per the task brief.
2. Quality-Checker (#midcost) — runs the full quality suite.
   **Loop:** on failure, return findings to "Implementer" and re-run from there;
   repeat until clean or ask user or 5 iterations, then report any outstanding failures.
```

**Prompt change after implementation:**
> "Send the Iterative Quality Pass team at the current branch."

---

## 7. Failure Handler with Skill Escalation

Build loop with an `if fail` handler: if the loop exhausts its cap without passing, a named skill is invoked instead of stopping silently. Suitable for CI-style pipelines where failures need to be triaged by a separate process.

> **Human prompt:** "Build and certify this artifact — retry up to 4 times on failure. If it still doesn't pass, escalate to the triage process instead of just stopping."

```
### Build & Certify

1. Builder (#lowcost) — builds the artifact.
2. Certifier (#midcost) — runs the certification suite.
   **Loop:** on failure, return findings to "Builder" and re-run from there;
   repeat until certified or 4 iterations, then report failures.
   if fail /escalate_cert_failure
```

**Prompt change after implementation:**
> "Send the Build & Certify team."

---

## 8. Nested Parallel Sub-teams, then Integrate

Two independent workstreams each running their own implement/validate loop in parallel. The outer `wait` holds the Integrator until both complete. Structure scales to any number of parallel tracks.

> **Human prompt:** "Work on the frontend and backend at the same time — each one should self-correct until it passes. Once both are done, merge them."

```
### Parallel Tracks & Integrate

1. Orchestrator (#highcap) — divides work across the two tracks and defines integration criteria.
2. [ Track-A[ Implementer (#lowcost), Validator (#midcost)
              **Loop:** to "Implementer" until pass or 3 ] (parallel),
     Track-B[ Implementer (#lowcost), Validator (#midcost)
              **Loop:** to "Implementer" until pass or 3 ] (parallel) ] (wait)
3. Integrator (#midcost) — merges both track outputs and verifies the combined result.
```

**Prompt change after implementation:**
> "Send the Parallel Tracks & Integrate team — Track-A: frontend, Track-B: backend."

---

## 9. Security Audit with Conditional Log Scan

Optional role (`if needed`) that the acting model skips when there is no runtime evidence to gather. The Security-Reviewer invokes a named skill as its charter. Use as a drop-in security gate on any diff.

> **Human prompt:** "Run a security review on this diff — grab logs first if there are any. Let me decide whether to keep going if issues are found."

```
### Security Gate

1. Log-Scanner (#lowcost, if needed) — gathers runtime evidence; skipped if no logs are available.
2. Security-Reviewer (#highcap) — run /security-review on -context:source data-;
   on findings **Loop:** return findings to "Log-Scanner" until clean or ask user or -context:cap-.
```

**Prompt change after implementation:**
> "Send the Security Gate team at the current diff."

---

## 10. Full Research-to-Deliverable Pipeline

End-to-end: orchestrate, explore in parallel, plan (user-gated), implement with retry, optional validation, and publish behind a final user gate. Demonstrates all SAILL primitives in one definition.

> **Human prompt:** "Research this topic across both domains in parallel, put together a plan and show it to me first, then build it out — validate if I ask. Don't publish without my say-so."

```
### Research to Deliverable

1. Orchestrator (#highcap) — breaks the objective into scoped tasks and fans out to Recon.
2. Recon[ Domain-A (#investigate, parallel),
          Domain-B (#investigate, parallel) ] (wait)
3. Planner (#highcap, ask user) — synthesizes findings into a delivery plan; pauses for approval.
4. Implementer (#lowcost) — executes the approved plan.
5. Validator (#midcost, if asked) — verifies correctness and completeness.
   **Loop:** on failure, return feedback to "Implementer" and re-run from there;
   repeat until clean or 3 iterations, then report outstanding failures.
6. Publisher (#lowcost, ask user) — waits for final approval, then delivers the output.
```

**Prompt change after implementation:**
> "Send the Research to Deliverable team — Domain-A: regulatory landscape, Domain-B: competitor analysis."

---

## 11. Index, Audit & Document

Parallel indexing, then a sequential audit/write pipeline with a cleanup step. Demonstrates a named parallel box with wait, followed by a chain of three sequential roles. No loops or gates — a straight pipeline where each stage hands off to the next.

> **Human prompt:** "Send a haiku agent to index skills and index bin in parallel. Send a sonnet agent to view all skill files and point any that reference a utility to point to the utility in bin — that agent should also summarize each utility in documentation/Helper Utilities/glob.txt. Then send a haiku agent to write the documentation file and delete glob.txt when it's done. Then update the indexes."

```
### Index, Audit & Document

1. Indexers[ Skills-Indexer (#lowcost, parallel),
             Bin-Indexer    (#lowcost, parallel) ] (wait)
2. Skill-Auditor (#midcost) — reads all skill SKILL.md files; updates any utility
   references to point to the resolved path in bin; writes a one-line summary of
   each referenced utility to documentation/Helper Utilities/glob.txt.
3. Doc-Writer (#lowcost) — writes the documentation file from glob.txt into
   documentation/Helper Utilities/; deletes glob.txt on completion.
4. Index-Updater (#lowcost) — refreshes any indexes affected by the above changes.
```

**Prompt change after implementation:**
> "Send the Index, Audit & Document team."

---

## Quick-Reference: Primitives Used

| Primitive | Examples above |
|---|---|
| Sequential roles | 1, 2, 3, 7, 11 |
| `if needed` | 9 |
| `if asked` | 5 (Validator), 10 |
| `ask user` | 2, 6, 9, 10 |
| `parallel` / `wait` | 4, 8, 10, 11 |
| `Loop` | 3, 5, 6, 7, 8, 9, 10 |
| Compound loop exit (`or ask user`) | 6, 9 |
| `-context-` values | 5, 6 (cap), 9 |
| `if fail <skill>` | 7 |
| Skill call in charter | 9 |
| Nested boxes | 8, 11 |
