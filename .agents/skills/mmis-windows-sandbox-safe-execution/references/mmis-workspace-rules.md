# MMIS Workspace Rules

Use these rules when working in `C:\FTS\NovoRepo_FOITS\DOWNLOADS\FTS-OITS\MMIS-URA`.

## Canonical Paths

Active code roots:

- `.branch-work/MarketMasterApi/`
- `.branch-work/MarketMasterERP/`

Supporting workspace paths:

- `Docs/`
- `.agents/skills/` for workspace-local skill definitions
- workspace-local temp helper files only when necessary

Secondary or non-default roots:

- `MarketMasterApi-dev/`
- `MarketMasterERP-dev/`
- sibling folders outside this workspace
- archived snapshots under analysis or zipped areas

Do not run implementation from one copy and validation from another.

## Canonical Command Pattern

Always run from the owning repo root first.

Backend examples from `.branch-work/MarketMasterApi`:

```powershell
npm test
npm run build
npm start
npx prisma migrate deploy
npx prisma db seed
```

Frontend examples from `.branch-work/MarketMasterERP`:

```powershell
npx tsc --noEmit --pretty false
npm run build
npm run dev -- --host 127.0.0.1 --port 5173
```

If a dev server or worker launch hits `spawn EPERM`, treat it as a sandbox execution issue first, not an app-code issue.

## Known Causes

- `spawn EPERM`: blocked child-process orchestration such as Vite, nodemon, browser, or worker launch.
- split-root wrapper refusal on patch: nested protected path or wrapper limitation on existing-file edits.
- ACL `Access is denied`: protected workspace subtree such as `.agents\skills`.
- Git failures: sandbox is guarding `.git` writes.
- network, TLS, DB, or auth failures during tools: command needs escalation.

## Stable Fixes

- Use PowerShell-native runners and repo-local commands.
- Keep generated reports, env files, docs, and temp helpers inside this workspace.
- Use escalated PowerShell writes for protected subtrees when necessary.
- Escalate for Git metadata writes, browser or PDF launch, network or database access, or blocked subprocesses that cannot be replaced with a simpler path.

## Verification Habit

After any workaround:

1. rerun the exact intended command through the stable path
2. inspect the produced diff, log, API response, or report
3. remove temporary helper artifacts if they are no longer needed
4. write down the stable path in docs if it closes a recurring blocker
