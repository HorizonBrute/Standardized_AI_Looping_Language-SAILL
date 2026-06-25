# Helper Utilities

These utilities support AIOS framework operations across three privilege tiers: bin (available to all users), sbin (system administration only), and usrbin (user-owned copies of sbin utilities).

## bin

| Utility | Description |
|---------|-------------|
| [context_cost.py](../../../../horizon_system/bin/context_cost.py) | Walks the harness auto-load chain (CLAUDE.md, agents.md, @-imports) for a given path and reports context overhead in bytes, words, and estimated tokens. |
| [monitor_status.py](../../../../horizon_system/bin/monitor_status.py) | Checks whether horizon_aios_monitor.py is currently running and prints "running" or "stopped" (cross-platform: Win32_Process CIM query on Windows, pgrep on Unix). |
| [resolve_agent_teams.py](../../../../horizon_system/bin/resolve_agent_teams.py) | Walks the scope cascade from a given path up to the AIOS root to resolve all Agent Team definitions in effect, merging shipped defaults with machine-local overrides; supports --json, --flags, and per-team role/model/flag output. |
| [resolve_sound.py](../../../../horizon_system/bin/resolve_sound.py) | Resolves an AIOS event name to an absolute sound-file path by checking per-project and system-level sounds.map and mute config, printing nothing (and exiting 0) when muted or unmapped. |
| [statusline.sh](../../../../horizon_system/bin/statusline/statusline.sh) | Cross-platform statusline dispatcher that reads Claude Code JSON from stdin and routes it to the platform-appropriate statusline script (PowerShell on Windows, Bash on macOS/Linux). |
| [statusline-command.sh](../../../../horizon_system/bin/statusline/statusline-command.sh) | Bash statusline script for Linux/macOS that extracts cwd, git branch, model, and context-usage percentage from Claude Code JSON and emits a formatted status line with threshold audio alerts. |
| [statusline-context-alerts.ps1](../../../../horizon_system/bin/statusline/statusline-context-alerts.ps1) | PowerShell statusline script for Windows that parses Claude Code JSON and outputs context-usage percentage and cost, with configurable threshold alerts read from the nearest aios_statusline.conf. |

## sbin

| Utility | Description |
|---------|-------------|
| [bootstrap.ps1](../../../../horizon_system/sbin/bootstrap.ps1) | Idempotent PowerShell bootstrap that configures a new Windows machine with all required Horizon AIOS environment variables, shell profile entries, symlinks, and scheduled tasks. |
| [bootstrap.sh](../../../../horizon_system/sbin/bootstrap.sh) | Idempotent Bash bootstrap that configures a new machine (Windows Git Bash, macOS, or Linux) with all required Horizon AIOS environment variables, shell profile entries, and symlinks. |
| [bootstrap_docker.ps1](../../../../horizon_system/sbin/bootstrap_docker.ps1) | PowerShell wrapper that runs the standard bootstrap in non-interactive Docker mode, skipping shell-profile instructions and sync-schedule setup for container builds. |
| [bootstrap_docker.sh](../../../../horizon_system/sbin/bootstrap_docker.sh) | Bash wrapper that runs the standard bootstrap in non-interactive Docker mode, automatically executed during docker build to configure the AIOS inside a container image. |
| [horizon_aios_backup_user_data.py](../../../../horizon_system/sbin/horizon_aios_backup_user_data.py) | Force-adds gitignored user data paths (memory, handoffs, objectives) to a temporary commit and pushes them to the owner's personal remote without touching the framework .gitignore. |
| [horizon_aios_brain_credential.py](../../../../horizon_system/sbin/horizon_aios_brain_credential.py) | Stores and retrieves brain OS account passwords in the native OS keystore (Windows Credential Manager, macOS Keychain, or Linux Secret Service) via the keyring library. |
| [horizon_aios_brain_logon_rights.py](../../../../horizon_system/sbin/horizon_aios_brain_logon_rights.py) | Grants, revokes, or queries a Windows LSA logon right on a brain account to enable opt-in automation tiers (e.g. batch logon, service logon) for scheduled brain execution. |
| [horizon_aios_create_brain.py](../../../../horizon_system/sbin/horizon_aios_create_brain.py) | Provisions a new brain by creating the OS user account, the shared brains group and a per-brain group, the workspace folder with correct ACLs, and a login shell profile with the auto-generated password stored in the keystore. |
| [horizon_aios_doctor.py](../../../../horizon_system/sbin/horizon_aios_doctor.py) | Health-check utility that verifies the AIOS installation state (env vars, paths, permissions, git config); run with --post-setup to additionally validate sound, statusline, and commit signing. |
| [horizon_aios_harden.py](../../../../horizon_system/sbin/horizon_aios_harden.py) | Applies the authoritative brains-group ACL model to $HORIZON_SYSTEM (deny brains on sbin/skills_sbin/logs; allow on bin/skills_bin), enforcing security_invariants.md §2/§3/§5 independently of brain creation. |
| [horizon_aios_maintain_logs.py](../../../../horizon_system/sbin/horizon_aios_maintain_logs.py) | Prunes AIOS log files older than the configured retention period and rotates any log file that exceeds the configured size limit. |
| [horizon_aios_maintain_logs_runner.ps1](../../../../horizon_system/sbin/horizon_aios_maintain_logs_runner.ps1) | PowerShell wrapper that resolves the Python interpreter and delegates to horizon_aios_maintain_logs.py; intended for periodic Task Scheduler or cron execution. |
| [horizon_aios_monitor.py](../../../../horizon_system/sbin/horizon_aios_monitor.py) | Filesystem integrity monitor that watches the AIOS layer for unexpected file creates, modifies, deletes, and moves and logs each event as a JSON line to $HORIZON_SYSTEM/logs/horizon_aios_monitor/. |
| [horizon_aios_monitor_analyze.py](../../../../horizon_system/sbin/horizon_aios_monitor_analyze.py) | Reads horizon_aios_monitor.py JSON-line logs, checks for file-change events and monitor uptime gaps, and writes a security summary to horizon_aios_security.log with optional OS syslog/Event Log alerts. |
| [horizon_aios_monitor_analyze_runner.ps1](../../../../horizon_system/sbin/horizon_aios_monitor_analyze_runner.ps1) | PowerShell wrapper for horizon_aios_monitor_analyze.py intended for periodic Task Scheduler or cron execution; passes through --syslog and --days arguments. |
| [horizon_aios_monitor_runner.ps1](../../../../horizon_system/sbin/horizon_aios_monitor_runner.ps1) | PowerShell launcher for horizon_aios_monitor.py for manual use or as a service wrapper; forwards all arguments to the Python script. |
| [horizon_aios_redirect_memory.py](../../../../horizon_system/sbin/horizon_aios_redirect_memory.py) | Redirects the harness's per-project state directory (~/.claude/projects/) into $HORIZON_ROOT/memory/ via a symlink so all conversation transcripts and agent memory are centrally governed by AIOS rules. |
| [horizon_aios_register_user_skills.py](../../../../horizon_system/sbin/horizon_aios_register_user_skills.py) | Idempotently aggregates the owner's skill view by symlinking brain-tier (skills_bin) and machine-local skills into skills_sbin so the owner sees all skill tiers in one place. |
| [horizon_aios_relocate.py](../../../../horizon_system/sbin/horizon_aios_relocate.py) | Rewrites all machine-local AIOS root pointers (env vars, ~/.claude/CLAUDE.md, skills symlink, settings.json) when the AIOS tree is moved to a new absolute path. |
| [horizon_aios_remove_brain.py](../../../../horizon_system/sbin/horizon_aios_remove_brain.py) | Deprovisions a brain by reversing horizon_aios_create_brain.py — removes the OS user account, per-brain group, workspace folder, shell profile, and stored credential while leaving the shared brains group intact. |
| [horizon_aios_setup_monitor_service.py](../../../../horizon_system/sbin/horizon_aios_setup_monitor_service.py) | Installs the AIOS filesystem monitor and its log analyzer as scheduled services (Windows Task Scheduler tasks at logon and daily, or Unix cron jobs). |
| [horizon_aios_setup_sync_schedule.py](../../../../horizon_system/sbin/horizon_aios_setup_sync_schedule.py) | Installs the AIOS auto-sync task as a Windows Task Scheduler entry or Unix cron job so the framework syncs from the remote on a schedule. |
| [horizon_aios_switch.py](../../../../horizon_system/sbin/horizon_aios_switch.py) | Rewrites the five machine-global AIOS pointers (env vars, ~/.claude/CLAUDE.md, skills symlink, settings.json, and the AIOS registry) to point at a different AIOS root, switching which AIOS the machine uses. |
| [horizon_aios_sync.py](../../../../horizon_system/sbin/horizon_aios_sync.py) | Syncs the AIOS framework from the upstream remote into the local install; lives in sbin and must not be exposed to brain users. |
| [horizon_aios_sync_runner.ps1](../../../../horizon_system/sbin/horizon_aios_sync_runner.ps1) | Thin PowerShell launcher for Windows Task Scheduler that resolves the Python interpreter and delegates to horizon_aios_sync.py. |
| [horizon_aios_update.py](../../../../horizon_system/sbin/horizon_aios_update.py) | Mirrors the Horizon_AI_OS upstream into the owner's personal fork via a bare clone, then pulls the result into the local install. |
| [horizon_aios_verify_isolation.py](../../../../horizon_system/sbin/horizon_aios_verify_isolation.py) | Brain-isolation test (Criterion #5) that proves a brain OS account can read $HORIZON_BIN but is denied $HORIZON_SYSTEM/sbin, either by static ACL inspection (default) or live impersonation (--live). |
| [uninstall.ps1](../../../../horizon_system/sbin/uninstall.ps1) | Idempotent PowerShell uninstall script that reverses all bootstrap.ps1 changes on a Windows machine without destroying user content. |
| [uninstall.sh](../../../../horizon_system/sbin/uninstall.sh) | Idempotent Bash uninstall script that reverses all bootstrap.sh changes on macOS, Linux, or Windows Git Bash without destroying user content. |

## usrbin

| Utility | Description |
|---------|-------------|
| [horizon_aios_update.py](../../../../horizon_system/usrbin/horizon_aios_update.py) | Mirrors the Horizon_AI_OS upstream into the owner's personal fork via a bare clone, then pulls the result into the local install (machine-local copy, gitignored, never synced). |
