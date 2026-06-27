# MMIS URA Health Smoke Run Report

Date: 2026-06-24
Workspace: `C:\FTS\NovoRepo_FOITS\MMIS-URA-FTS-DEV`

## Run Verdict

The reusable local health-smoke harness is now proven through the current port-configurable PowerShell runner path.

Current strongest proof is the latest successful run against a clean workspace-owned backend instance on `http://127.0.0.1:5053`.

Verified outputs:

- Newman run: `passed`
- Hoppscotch CLI run: `passed`
- JUnit exports: `present`
- summary JSON: `present`
- launcher state file: `present`

Overall verdict:

`REPEATABLE LOCAL RUNNER HARNESS IS PROVEN WHEN THE WORKSPACE OWNS THE TARGET PORT AND JWT CONTEXT`

## Harness Used

PowerShell harness:

- [run_mmis_ura_health_smoke.ps1](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/scripts/run_mmis_ura_health_smoke.ps1)

Backend launcher:

- [start_mmis_api_mock_5051.ps1](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/scripts/start_mmis_api_mock_5051.ps1)

Supporting env refresher:

- [write_mmis_ura_health_smoke_envs.js](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/scripts/write_mmis_ura_health_smoke_envs.js)

Collections used:

- [MMIS_URA_HEALTH_SMOKE_2026-06-24.postman_collection.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/URA-EFRIS-CODEX-DOCS-REPORTS/MMIS_URA_HEALTH_SMOKE_2026-06-24.postman_collection.json)
- [MMIS_URA_HEALTH_SMOKE_2026-06-24.postman_environment.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/URA-EFRIS-CODEX-DOCS-REPORTS/MMIS_URA_HEALTH_SMOKE_2026-06-24.postman_environment.json)
- [MMIS_URA_HEALTH_SMOKE_2026-06-24.hoppscotch.collection.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/URA-EFRIS-CODEX-DOCS-REPORTS/MMIS_URA_HEALTH_SMOKE_2026-06-24.hoppscotch.collection.json)
- [MMIS_URA_HEALTH_SMOKE_2026-06-24.hoppscotch.env.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/URA-EFRIS-CODEX-DOCS-REPORTS/MMIS_URA_HEALTH_SMOKE_2026-06-24.hoppscotch.env.json)

## Command Executed

Launcher:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/start_mmis_api_mock_5051.ps1 -Port 5053
```

Smoke run:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/run_mmis_ura_health_smoke.ps1 -Port 5053
```

## Observed Result

Newman:

- request: `GET http://127.0.0.1:5053/api/ura/health`
- HTTP status: `200`
- assertion: `status 200`
- result: passed

Hoppscotch CLI:

- request: `GET http://127.0.0.1:5053/api/ura/health`
- HTTP status: `200`
- test: `URA health returns 200`
- result: passed

Launcher state:

- port: `5053`
- JWT source: `workspace-local-dev-secret`
- state file: [mmis-api-5053.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/state/mmis-api-5053.json)

## Generated Reports

Saved under:

- [\.runner-tools\reports](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports)

Artifacts:

- [mmis-ura-health-smoke.newman.junit.xml](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke.newman.junit.xml)
- [mmis-ura-health-smoke-5053.newman.junit.xml](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke-5053.newman.junit.xml)
- [mmis-ura-health-smoke.hopp.junit.xml](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke.hopp.junit.xml)
- [mmis-ura-health-smoke-5053.hopp.junit.xml](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke-5053.hopp.junit.xml)
- [mmis-ura-health-smoke.summary.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke.summary.json)
- [mmis-ura-health-smoke-5053.summary.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/reports/mmis-ura-health-smoke-5053.summary.json)

## Important 5051 Caveat

Port `5051` is not currently trustworthy by default.

The launcher now records whether a live listener matches the last workspace-owned launcher state. Current evidence on `5051` shows drift:

- `alreadyListening=true`
- `listenerMatchesKnownLauncher=false`
- warning: `Port 5051 is already listening, but it does not match the last workspace-owned launcher state. JWT or runtime drift may cause false smoke failures.`

Reference file:

- [mmis-api-5051.json](/C:/FTS/NovoRepo_FOITS/MMIS-URA-FTS-DEV/.runner-tools/state/mmis-api-5051.json)

This means a `401` on `5051` should not be treated as a code regression until listener ownership is confirmed.

## What This Proves

- runner installation is real
- collection execution is real
- the workspace can produce both latest and immutable per-port machine-readable test reports
- the backend wrapper surface is reachable through both Newman and Hoppscotch CLI
- the smoke path is no longer tied to one contested fixed port

## What It Does Not Yet Prove

- real cloned-UAT database flows
- login or business workflows against persisted data
- full-surface generated collection success
- staged or shared-dev MMIS success
- real URA sandbox credentialed success

## Next Step

Use the same launcher plus smoke pattern for:

1. a DB-backed auth smoke,
2. a vendor or supplier onboarding smoke,
3. a PRN generation smoke in mock mode,
4. then staged or shared-dev execution after a real target URL and credentials are provided.


