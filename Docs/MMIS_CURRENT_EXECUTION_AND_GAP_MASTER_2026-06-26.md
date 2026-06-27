# MMIS Current Execution And Gap Master

Date: 2026-06-26

## Executive Status

`IMPLEMENTATION SURFACE IS BROAD AND CURRENT / REAL UAT IS STILL PARTIAL`

The current MMIS + URA-EFRIS workspace now has:

- fresh implemented API inventory
- fresh route security matrix
- fresh full-collection parity evidence
- a current OpenAPI skeleton
- one real passing Postman + Hoppscotch smoke lane
- one repeatable runtime dependency probe
- one repeatable onboarding safe-path runner

The current blocker is not missing API route coverage. It is still runtime readiness for DB-backed and sandbox-backed execution.

## Current Machine-Readable Sources

- `MMIS_IMPLEMENTED_API_ENDPOINT_INVENTORY_2026-06-26.json`
- `MMIS_IMPLEMENTED_API_ROUTE_SECURITY_MATRIX_2026-06-26.json`
- `MMIS_CURRENT_IMPLEMENTED_COLLECTION_PARITY_2026-06-26.json`
- `MMIS_IMPLEMENTED_API_OPENAPI_SKELETON_2026-06-26.json`
- `MMIS_CURRENT_RUNTIME_EXECUTION_MATRIX_2026-06-26.csv`
- `.runner-tools/reports/mmis-ura-health-smoke-5052.summary.json`
- `.runner-tools/reports/mmis-ura-runtime-dependency-probe.summary.json`
- `.runner-tools/reports/mmis-onboarding-safe-path-full-5053.summary.json`

## Current Verified API Surface

Live `MarketMasterApi` route inventory:

- total routes: `194`
- authenticated: `109`
- admin: `44`
- public: `27`
- superadmin: `14`
- validated: `103`
- unvalidated: `91`

Major domain counts:

- `payments_billing`: `34`
- `ura_efris`: `34`
- `vendor_supplier`: `33`
- `auth_onboarding`: `27`
- `gate_market_ops`: `16`
- `supply_delivery`: `13`

This confirms the implementation breadth aligns with the intended MMIS system domains, even though execution evidence is still uneven across them.

## Current Collection Truth

Fresh parity against the live inventory shows:

- full generated Postman collection: `194 / 194`
- full generated Hoppscotch collection: `194 / 194`
- June 26 onboarding safe-path Postman pack: intentionally narrow
- June 26 onboarding safe-path Hoppscotch pack: intentionally narrow

Meaning:

- full collection drift is not the active blocker
- execution readiness is the active blocker

## Real Execution Evidence Completed

### 1. Local URA health smoke

Executed successfully on:

- `http://127.0.0.1:5052`

Tools:

- Newman
- Hoppscotch CLI

Result:

- `GET /api/ura/health` passed in both tools

Evidence:

- `.runner-tools/reports/mmis-ura-health-smoke-5052.summary.json`

### 2. URA runtime dependency probe

Repeatable probe script:

- `scripts/probe_mmis_ura_runtime_dependencies.ps1`

Executed successfully on:

- `http://127.0.0.1:5052`

Result split:

- `mock_runnable`: `1`
- `db_blocked`: `4`
- `other_failure`: `0`

Exact route outcomes:

- `/api/ura/health` -> `200`
- `/api/ura/catalog/mda-listing` -> `503 Database connection failed`
- `/api/admin/ura/requests?limit=1` -> `503 Database connection failed`
- `/api/admin/ura/tin-verifications?limit=1` -> `503 Database connection failed`
- `/api/admin/ura/efris-outbox?limit=1` -> `503 Database connection failed`

Evidence:

- `.runner-tools/reports/mmis-ura-runtime-dependency-probe.summary.json`

## Current Onboarding Safe-Path Execution Status

Repeatable scripts now exist:

- `scripts/write_mmis_onboarding_safe_path_envs.js`
- `scripts/run_mmis_onboarding_safe_path_uat.ps1`

Verified runner behavior:

- runtime env files are generated
- mode-based temporary subsets are generated
- execution stops correctly before Newman / Hoppscotch when required inputs remain unresolved

Current blocking inputs:

- seeded admin credentials or token
- seeded vendor credentials or token
- seeded supplier credentials or token
- vendor market id
- supplier market id
- vendor tax id number
- supplier tax id / tax id number
- TIN evidence document path

Evidence:

- `.runner-tools/reports/mmis-onboarding-safe-path.env-refresh.json`
- `.runner-tools/reports/mmis-onboarding-safe-path-full-5053.summary.json`

## Important Code-Level Fix Already Applied

The earlier authenticated URA route crash was resolved by fixing:

- `MarketMasterApi/src/middleware/validate.middleware.js`

Effect:

- stop reassigning `req.query` under Express 5 request objects
- preserve segmented validation through `req.validated.query`
- unblock authenticated URA route execution for the local mock smoke lane

Validation after fix:

- backend `npm test`: passed
- backend `npm run build`: passed

## What Is Still Missing For The Full Objective

### DB-backed local UAT

Still missing:

- real backend `.env`
- real frontend `.env`
- valid `DATABASE_URL`

### Onboarding / identity UAT

Still missing:

- seeded admin, vendor, and supplier credentials or working tokens
- real market IDs
- real TIN values
- TIN evidence document path

### Staged/shared-dev UAT

Still missing:

- staged base URL
- any required staged auth/runtime inputs

### URA sandbox/provider-backed execution

Still missing:

- URA sandbox credentials
- runtime target for provider-backed calls

## UI / UX Self-Evaluation Use Now

What can be self-evaluated right now:

1. supplier signup field parity in the frontend
2. vendor and supplier dashboard `KYC` routing
3. admin URA page tabs including EFRIS outbox
4. frontend buildability and proxy-target behavior

What still cannot be honestly claimed without more runtime inputs:

1. complete DB-backed onboarding
2. real PRN / catalog / audit lifecycle execution
3. EFRIS outbox behavior against persisted records
4. staged dev proof
5. URA sandbox proof

## Best Next Execution Order

1. provide real local backend and frontend env files plus `DATABASE_URL`
2. rerun the onboarding safe-path runner in `contract`, then `auth`, then `full`
3. rerun the URA dependency probe against that DB-backed target
4. execute the full `194`-route generated Postman and Hoppscotch packs in domain waves
5. add staged base URL and repeat the same sequence for shared dev
6. add URA sandbox credentials and run backend-routed provider-backed verification

## Bottom Line

The project is past the vague-analysis stage.

There is now a current executable stack:

- full inventory
- full parity
- openapi skeleton
- passing local health smoke
- repeatable dependency probe
- repeatable onboarding safe-path runner

The honest remaining gap is operational execution with real DB and sandbox inputs, not lack of architecture, lack of collection coverage, or lack of route implementation.
