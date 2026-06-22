# MMIS Dev vs Feature vs Local Branch Diff Table

Date: 2026-06-23
Workspace: `C:\FTS\NovoRepo_FOITS\DOWNLOADS\FTS-OITS\MMIS-URA`

## Scope Note

The branch `fix/PR-PRN-URA-CodebaseRefinements` is **not** a branch inside the real `MarketMasterApi` repository.
It is the outer MMIS workspace repository.

The actual backend code for `MarketMasterApi` lives at:

- `C:\FTS\NovoRepo_FOITS\DOWNLOADS\FTS-OITS\MMIS-URA\.branch-work\MarketMasterApi`

So this comparison is split into two layers:

1. Real backend branch comparison inside `MarketMasterApi`
2. Outer workspace branch state that pins the backend repo to a specific commit and adds local analysis artifacts

## Branch Identity

| Layer | Ref | SHA | Meaning |
| --- | --- | --- | --- |
| MarketMasterApi | `origin/dev` | `f8a355d9d10d261a9bf1067b5e683de91659d955` | Current GitHub `dev` |
| MarketMasterApi | `feature/mmis-420-prn-t109-handoff` | `a84b916029280ea663fe5a8c127c675c1211808d` | Current feature branch head |
| Workspace repo | `fix/PR-PRN-URA-CodebaseRefinements` | `da90461428b1fa7942268cd470eb048ad68b35e8` | Outer workspace branch |
| Workspace gitlink | `.branch-work/MarketMasterApi` | `a84b916029280ea663fe5a8c127c675c1211808d` | Local branch pins backend repo to the same commit as `feature/mmis-420-prn-t109-handoff` |

## Divergence Summary

| Metric | Value |
| --- | --- |
| Merge base between `origin/dev` and `feature/mmis-420-prn-t109-handoff` | `4c24374e23132c00ac0e20d52a55d18456ad8aea` |
| `origin/dev` ahead of feature | `8` commits |
| `feature/mmis-420-prn-t109-handoff` ahead of `origin/dev` | `1` commit |
| Effective backend delta between feature and local `fix/...` branch | `0` code commits |

## What This Means

Current GitHub `dev` already includes the earlier MMIS-420 PRN/T109 handoff slice through merge commit `131ed32`.
The remaining code difference is:

- `feature/mmis-420-prn-t109-handoff` adds one extra backend hardening commit: `a84b916`
- `origin/dev` adds eight later commits, mostly webhook hardening, validation wiring, and the supplier delivery module
- Local branch `fix/PR-PRN-URA-CodebaseRefinements` points its backend gitlink to `a84b916`, so its effective backend code matches the feature branch

## Branch-Exclusive Commits

### Feature-Only Commit

| Side | Commit | Summary |
| --- | --- | --- |
| Feature and local backend pin | `a84b916` | `fix: harden rent PRN config and add fiscal mapping artifacts` |

### Dev-Only Commits

| Side | Commit | Summary |
| --- | --- | --- |
| Dev only | `f8a355d` | `Merge pull request #100 from oits-dhaka/win/mmis-419-add-supplier-delivery-module` |
| Dev only | `ad26c0c` | `Merge pull request #86 from oits-dhaka/win/mmis-121-webhook-idempotency` |
| Dev only | `80552bf` | `Merge pull request #87 from oits-dhaka/win/mmis-232-route-validation-middleware` |
| Dev only | `131ed32` | `Merge pull request #98 from oits-dhaka/feature/mmis-420-prn-t109-handoff` |
| Dev only | `7d76d53` | `Implement delivery module` |
| Dev only | `e14d2a3` | `fix: preserve webhook lifecycle event progression` |
| Dev only | `bedf0c7` | `MMIS-232 add standardized validation middleware and wire auth/payment routes` |
| Dev only | `dda25b0` | `MMIS-121 harden Flutterwave webhook idempotency and signature handling` |

Note:
`131ed32` is the reason most of the original MMIS-420 branch work is already present on current GitHub `dev`.

## Shared MMIS-420 Slice Already Present In Both `origin/dev` And Feature

These are already shared by both current GitHub `dev` and the current feature/local backend pin:

| Area | Shared Outcome |
| --- | --- |
| URA partner service | Reusable URA auth, client, config, mock client, service, audit, utilities |
| Rent PRN | Rent payment PRN attachment service, controller, route wiring, admin exposure |
| Fiscal fixtures | Draft T109 payload fixture and rent tax mapping vectors |
| EFRIS prep | EFRIS outbox skeleton and queue service |
| Validation | URA route validation schemas and middleware wiring |
| Onboarding | TIN-driven vendor and supplier onboarding updates |
| Tests | URA unit-test coverage scaffold |
| Documentation | MMIS-373, MMIS-374/376, MMIS-407, smoke/UAT runbook, EFRIS change log |

## Residual File-Level Diff Since The Current Merge Base

This is the actionable diff between the current GitHub `dev` branch and the current feature/local backend commit.

| Path | `origin/dev` | `feature/mmis-420-prn-t109-handoff` | Local `fix/...` branch | Side | Delta | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| `Docs/ura-efris/MMIS-373-rent-tax-mapping-table.md` | unchanged from merge base | modified | same as feature via gitlink | Feature only | `+41/-0` | Adds rent tax mapping table |
| `Docs/ura-efris/MMIS-374-376-t109-rent-payment-contract-slice.md` | earlier version | modified | same as feature via gitlink | Feature only | `+57/-6` | Extends T109 contract slice details |
| `src/fixtures/ura/mmis-373-rent-tax-mapping.vectors.json` | unchanged from merge base | added | same as feature via gitlink | Feature only | `+70/-0` | Adds executable tax-mapping vectors |
| `src/fixtures/ura/t109-rent-payment.fixture.json` | earlier version | modified | same as feature via gitlink | Feature only | `+200/-84` | Hardens canonical T109 fixture payload |
| `src/services/ura/ura-rent-prn.service.js` | earlier version | modified | same as feature via gitlink | Feature only | `+24/-3` | Tightens rent PRN config and mapping flow |
| `test/ura.unit.test.js` | earlier version | modified | same as feature via gitlink | Feature only | `+67/-0` | Adds regression coverage for fixture/mapping changes |
| `prisma/migrations/20260622123000_add_supplier_request_deliveries/migration.sql` | added | absent | absent | Dev only | `+159/-0` | Delivery-handoff schema change present only on dev |
| `prisma/schema.prisma` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+70/-28` | Dev schema evolved for delivery support |
| `src/controllers/delivery.controller.js` | added | absent | absent | Dev only | `+81/-0` | New delivery controller on dev |
| `src/controllers/payment-ops.controller.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+153/-46` | Dev-side payment ops changes |
| `src/routes/auth.routes.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+15/-6` | Additional auth validation wiring on dev |
| `src/routes/delivery.routes.js` | added | absent | absent | Dev only | `+53/-0` | New delivery routes on dev |
| `src/routes/index.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+2/-0` | Dev mounts delivery routes |
| `src/routes/payment.routes.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+47/-8` | Dev payment route evolution |
| `src/services/delivery.service.js` | added | absent | absent | Dev only | `+555/-0` | Main supplier-delivery service added on dev |
| `src/services/supply-request.service.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+34/-21` | Delivery integration into supply-request flow |
| `src/validations/auth.validation.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+22/-0` | Extra auth validation added on dev |
| `src/validations/delivery.validation.js` | added | absent | absent | Dev only | `+91/-0` | Delivery validation schema |
| `src/validations/payment.validation.js` | added | absent | absent | Dev only | `+42/-0` | Payment validation changes on dev |
| `src/validations/supply-request.validation.js` | modified | unchanged from merge base | unchanged from merge base | Dev only | `+2/-1` | Supply request validation adjustments |

## Historical MMIS-420 Backend Slice Relative To The Old Local `dev` Pointer

If you compare the feature branch to the stale local backend `dev` pointer (`dd28207`) instead of current GitHub `origin/dev`, you see the full historical MMIS-420 plus URA/EFRIS build-out:

| Metric | Value |
| --- | --- |
| Files changed | `39` |
| Insertions | `5174` |
| Deletions | `37` |

That larger view is useful for reconstructing the whole rent-PRN/T109/URA slice, but it is **not** the correct picture of the current GitHub `dev` branch because PR `#98` has already been merged there.

## Outer Workspace Branch State

The outer branch `fix/PR-PRN-URA-CodebaseRefinements` currently contributes backend scope in two ways:

| Item | State on local branch | Relevance |
| --- | --- | --- |
| `.branch-work/MarketMasterApi` gitlink | points to `a84b916` | Makes local backend code effectively identical to the feature branch |
| `.branch-work/MarketMasterERP` gitlink | points to `7e04e74246d3855845b6714d99f072eb762cec92` | Frontend repo pin only |
| `Docs/MMIS_DEV_BRANCH_DEEP_ANALYSIS_VERDICT_2026-06-23.md` | untracked | Local audit artifact |
| `Docs/MMIS_DEV_BRANCH_ENDPOINT_AND_COLLECTION_INVENTORY_2026-06-23.md` | untracked | Local endpoint inventory |
| `Docs/MMIS_DEV_BRANCH_PAYLOAD_CONTRACT_RECHECK_2026-06-23.md` | untracked | Local payload-contract audit |
| `Docs/MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv` | untracked | Local collection diff export |
| `Docs/MMIS_DEV_BRANCH_API_COVERAGE_MATRIX_2026-06-23.csv` | untracked | Local coverage matrix |
| `_analysis/*` | untracked | Local analysis intermediates |

The same local branch also contains many unrelated workspace-level tracked additions such as `.specify`, `.agents/skills`, docs archives, and `node_modules`.
Those were excluded from the backend diff table because they do not represent additional `MarketMasterApi` code beyond the pinned gitlink commit.

## Bottom Line

| Comparison | Result |
| --- | --- |
| `origin/dev` vs `feature/mmis-420-prn-t109-handoff` | diverged: dev is `8` commits ahead, feature is `1` commit ahead |
| `feature/mmis-420-prn-t109-handoff` vs local backend on `fix/...` | effectively identical backend code at `a84b916` |
| `origin/dev` vs local backend on `fix/...` | same residual diff as `origin/dev` vs feature |
| Most important interpretation | Current GitHub `dev` already contains PR `#98`; the live unresolved backend difference is one feature hardening commit versus later dev delivery/webhook/validation work |

