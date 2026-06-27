# MMIS Onboarding Public Contract Execution Addendum

Date: 2026-06-26

## Scope

This addendum records the first real execution of the `contract` subset from the generated onboarding safe-path UAT pack against the local backend runtime.

Executed subset:

- `GET /api/auth/markets`
- `POST /api/auth/register-supplier` missing `taxId`
- `POST /api/auth/register` vendor
- `POST /api/auth/register-supplier` with `taxId`

Primary evidence:

- `.runner-tools/reports/mmis-onboarding-safe-path.summary.json`
- `.runner-tools/reports/mmis-onboarding-safe-path-contract-5053.summary.json`
- `.runner-tools/reports/mmis-onboarding-safe-path-contract-5053.newman.junit.xml`
- `MarketMasterApi/api-5052.err.log`
- `MarketMasterApi/api-5052.out.log`

## Execution Context

- Base URL used: `http://127.0.0.1:5052`
- Runner mode: `contract`
- Postman status: `1`
- Hoppscotch status: `1`
- Readiness gate: `passed`

Important runner nuance:

- The scoped summary filename still contains `5053` because the script defaults the `port` field separately from the overridden `baseUrl`.
- Actual execution target for this run was `5052`, confirmed by `.runner-tools/reports/mmis-onboarding-safe-path.summary.json`.

## Result Table

| Request | Expected | Actual | Verdict | Evidence |
| --- | --- | --- | --- | --- |
| `GET /api/auth/markets` | `200` | `500` | Runtime blocked, not contract-proven | Newman JUnit plus `getMarkets error` in `api-5052.err.log` |
| `POST /api/auth/register-supplier` missing `taxId` | `400` with validation rejection | `400` rejection happened | Contract mostly confirmed | Newman JUnit |
| `POST /api/auth/register-supplier` missing `taxId` message contains `URA TIN` | assertion true | assertion failed | Collection assertion stale vs current message text | Newman JUnit |
| `POST /api/auth/register` vendor | `200/201` | `503` | DB/runtime blocked | Newman JUnit plus Prisma init error in backend logs |
| `POST /api/auth/register-supplier` with `taxId` | `200/201` | `503` | DB/runtime blocked | Newman JUnit plus Prisma init error in backend logs |

## What The Run Proved

1. The generated contract subset is executable end to end through the current runner.
2. Public onboarding requests now pass the placeholder/readiness gate with disposable runtime values.
3. The supplier signup validation drift remains real:
   - backend still rejects missing `taxId`
   - the current response text no longer matches the collection assertion requiring the literal phrase `URA TIN`
4. The public onboarding happy-path endpoints are currently blocked by runtime database initialization, not by collection generation or route absence.

## Root Causes Observed

### 1. Missing database configuration in the local backend runtime

Backend logs show Prisma initialization failure:

- `Environment variable not found: DATABASE_URL.`

That failure surfaced in:

- `getMarkets`
- `registerUser`

This explains:

- `GET /api/auth/markets` returning `500`
- vendor registration returning `503`
- supplier registration with `taxId` returning `503`

### 2. Validation message assertion drift

The collection expected the rejection body to contain `URA TIN`, but the current response body only proved a validation failure and did not include that exact phrase.

Implication:

- the route behavior is still aligned with the intended mandatory-tax-id contract
- the assertion text in the generated contract pack now needs to be loosened or updated to the backend's current validation message

## Current Verdict

`PARTIALLY VERIFIED / RUNTIME-BLOCKED`

The public onboarding contract pack is no longer just documentation. It now has real execution evidence. However, it is not yet a clean green UAT slice because:

- the local backend runtime lacked `DATABASE_URL`
- the collection still contains one stale message-level assertion

## Immediate Next Steps

1. Start the backend with a valid dev or staged `DATABASE_URL` and rerun the same `contract` subset unchanged.
2. Update the supplier-missing-tax-id assertion to check for stable validation semantics instead of the exact phrase `URA TIN`, unless that wording is an explicit product requirement.
3. Once `contract` mode is green, rerun `auth` mode with seeded credentials.
4. After `auth` mode is green, rerun `full` mode with admin and seeded vendor/supplier tokens plus document path.

## Execution Meaning For The Larger UAT Goal

This run narrows the remaining gap:

- collection generation: working
- runner orchestration: working
- public contract subset: executed
- backend public onboarding runtime: blocked by database config
- seeded onboarding/admin visibility: still pending credentials and DB-backed runtime
