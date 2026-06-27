# MMIS Onboarding Safe Path Runner Guide

Date: 2026-06-26

## Purpose

This guide documents the executable runner for the June 26 onboarding safe-path collections.

It is the operational bridge between:

- `MMIS_ONBOARDING_SAFE_PATH_UAT_POSTMAN_COLLECTION_2026-06-26.json`
- `MMIS_ONBOARDING_SAFE_PATH_UAT_HOPPSCOTCH_COLLECTION_2026-06-26.json`

and actual Newman / Hoppscotch CLI execution.

## Scripts

### Env refresh

- `scripts/write_mmis_onboarding_safe_path_envs.js`

What it does:

- reads the June 26 Postman and Hoppscotch env templates
- applies `MMIS_*` runtime variables if present
- attempts seeded admin, vendor, and supplier logins when credentials exist
- writes runtime env files
- writes a refresh summary

Generated files:

- `MMIS_ONBOARDING_SAFE_PATH_UAT_POSTMAN_ENV_RUNTIME_2026-06-26.json`
- `MMIS_ONBOARDING_SAFE_PATH_UAT_HOPPSCOTCH_ENV_RUNTIME_2026-06-26.json`
- `.runner-tools/reports/mmis-onboarding-safe-path.env-refresh.json`

### Runner

- `scripts/run_mmis_onboarding_safe_path_uat.ps1`

Supported modes:

- `contract`
- `auth`
- `full`

What it does:

- refreshes runtime env files
- filters the collections to the selected folder scope
- checks unresolved placeholders before execution
- if ready, runs Newman and Hoppscotch CLI
- writes per-run summary and JUnit paths

## Current Verified Runner State

Verified command:

- `powershell -NoProfile -ExecutionPolicy Bypass -File "scripts/run_mmis_onboarding_safe_path_uat.ps1" -Mode full -Port 5053`

Current result:

- runner logic works
- filtered temporary collections are generated
- runtime env files are generated
- execution stops correctly before Newman / Hoppscotch because required inputs are still missing

Evidence:

- `.runner-tools/reports/mmis-onboarding-safe-path.env-refresh.json`
- `.runner-tools/reports/mmis-onboarding-safe-path.summary.json`
- `.runner-tools/reports/mmis-onboarding-safe-path-full-5053.summary.json`

## Current Missing Inputs

From the latest `full` summary:

- supplier email
- supplier market id
- supplier tax id
- vendor email
- vendor market id
- admin token or seeded admin credentials
- vendor token or seeded vendor credentials
- supplier token or seeded supplier credentials
- vendor tax id number
- supplier tax id number
- TIN document path

Equivalent Hoppscotch JWT and TIN/document placeholders are also still unresolved.

## Recommended Execution Order

1. Provide seeded admin, vendor, and supplier credentials through `MMIS_*` env vars.
2. Provide real market IDs, TINs, and the TIN evidence document path.
3. Run `contract` mode first.
4. Run `auth` mode next.
5. Run `full` mode only after DB-backed runtime is confirmed stable.

## Relationship To Other Packs

- Use `MMIS_LOCAL_MOCK_SMOKE_EXECUTION_ADDENDUM_2026-06-26.md` for the already-proven URA health smoke path.
- Use `MMIS_CURRENT_COLLECTION_EXECUTION_MAP_2026-06-26.md` for the bigger collection hierarchy.
- Use `MMIS_CURRENT_IMPLEMENTED_API_UAT_AND_SELF_EVAL_VERDICT_2026-06-26.md` for the broader platform verdict and remaining blockers.
