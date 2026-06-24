---
name: mmis-windows-sandbox-safe-execution
description: 'Prevent and route around Windows sandbox-wrapper failures in the MMIS workspace. Use when work is happening in C:\FTS\NovoRepo_FOITS\DOWNLOADS\FTS-OITS\MMIS-URA and commands or edits fail with spawn EPERM, split-root wrapper write errors, blocked .git metadata operations, protected subtree ACL issues, or unstable multi-process runner behavior in the split repos under .branch-work.'
---

# MMIS Windows Sandbox Safe Execution

Use this skill to keep MMIS work inside the execution pattern that is already proven in this workspace.

## Core Rules

1. Keep implementation and testing inside the active workspace roots only:
   - `.branch-work/MarketMasterApi/`
   - `.branch-work/MarketMasterERP/`
   - `Docs/`
   - workspace-local temp helpers only when needed
2. Do not switch active work back to stale sibling folders, archived snapshots, or `*-dev` copies unless the user explicitly asks for them.
3. Prefer PowerShell-native entrypoints for orchestration on Windows.
4. Treat Git metadata writes, protected subtree rewrites, browser launches, PDF generation, network or database access, and blocked subprocess launches as normal escalation cases.

## Execution Pattern

- Start from the owning repo root under `.branch-work/`.
- Keep generated env files, reports, docs, temp scripts, and collections inside this workspace.
- Run validation through repo-local commands from the correct repo root.
- Prefer direct PowerShell command paths over layered shell wrappers.

For the current MMIS-specific path rules and command patterns, read [references/mmis-workspace-rules.md](references/mmis-workspace-rules.md).

## Failure Routing

### `spawn EPERM`

- Assume the Windows sandbox wrapper blocked a child-process chain.
- Stop trying to fix the application code first.
- Replace the entrypoint with a PowerShell-native path when possible.
- If the task genuinely needs the blocked child process, rerun with escalation instead of stacking more wrappers.

### `apply_patch` or edit failures on existing files

- Confirm the target file is inside this workspace.
- Prefer `apply_patch` first when it works.
- If the wrapper refuses the edit on an existing file or protected subtree, use a narrow PowerShell rewrite inside the workspace and verify the resulting diff immediately.

### Git write failures

- Do not fight the sandbox for `checkout`, `commit`, `rebase`, `merge`, or `push`.
- Request escalation directly for Git metadata operations.

### Protected subtree ACL failures

- Treat `.agents\skills` or similar ACL-guarded paths as environment constraints, not repo bugs.
- Use an escalated PowerShell write path when the task truly needs that subtree.

## What To Avoid

- Node entrypoints that spawn `cmd.exe` and then other CLIs.
- Mixed checkouts where code is edited in one repo and tested in another.
- Temp scripts or reports written outside this workspace.
- Assuming the sandbox can be fixed from repository code. The repo can only avoid the wrapper's bad paths.

## Standard Response Pattern

When this skill triggers:

1. Identify whether the failure is file-write, child-process, Git metadata, ACL, or network/sandbox restriction.
2. Move to the canonical workspace-local repo path.
3. Replace ad hoc shell chains with the PowerShell-native path.
4. Escalate only for the classes of actions that the sandbox is expected to guard.
5. Record the stable path in repo docs if a new recurring failure mode was found.
