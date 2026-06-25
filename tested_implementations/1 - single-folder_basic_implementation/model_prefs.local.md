# Model Preferences Local Overrides

Override or extend model preferences defined in the base `model_prefs.md`.
Fill in model IDs for each group. This file is gitignored — it is machine-local.

---

## Per-Session Slot Preferences

### Spawned Agent Model
Unset

### Sub-Agent Override
Unset

---

## Model Groups

Add members to named groups. List in preference order; first runnable member wins.

### #lowcost

### #midcost

### #highcap

### #investigate

### #debug

### #fast

---

## Task-Class Routing

Define task-class to model-group mappings here. Most specific class wins on conflict.

---

## Scope Precedence

Local overrides win. Same merge rules as base file.
