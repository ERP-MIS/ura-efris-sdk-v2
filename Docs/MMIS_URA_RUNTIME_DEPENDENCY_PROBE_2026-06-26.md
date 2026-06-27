# MMIS URA Runtime Dependency Probe

Date: 2026-06-26

## Purpose

This addendum captures a small but real authenticated route probe against the local mock-backed MMIS backend after the URA validator fix.

The goal is to separate:

- routes that are runnable with only local process + local JWT
- routes that still hard-stop on database connectivity

## Runtime Used

- backend process: local `MarketMasterApi`
- base URL: `http://127.0.0.1:5052`
- auth: workspace-local generated `SuperAdmin` JWT
- probe method: raw `curl` against live localhost routes

## Probe Results

| Route | Result | Meaning |
| --- | --- | --- |
| `GET /api/ura/health` | `200 OK` | local adapter health is runnable without DB |
| `GET /api/ura/catalog/mda-listing` | `503 Database connection failed` | not runnable without DB |
| `GET /api/admin/ura/requests?limit=1` | `503 Database connection failed` | audit surface is DB-backed |
| `GET /api/admin/ura/tin-verifications?limit=1` | `503 Database connection failed` | TIN history surface is DB-backed |
| `GET /api/admin/ura/efris-outbox?limit=1` | `503 Database connection failed` | EFRIS outbox admin surface is DB-backed |

## Key Finding

The local mock mode is real, but it is not a database-free URA sandbox replacement.

Only the adapter health route is currently proven runnable without database access. The next obvious URA admin and catalog surfaces still fail at the persistence boundary with:

- `503`
- `Database connection failed`

## Why This Matters

This clarifies the current testing ladder:

1. `health` smoke can prove backend process, auth, route wiring, and adapter config.
2. Most useful URA admin/operator routes still require a real database-backed MMIS runtime.
3. Provider-backed sandbox testing is an additional later layer, not the immediate blocker for these routes.

## Updated UAT Interpretation

What is now proven:

- authenticated local URA health works through Postman and Hoppscotch
- the earlier Express 5 validation middleware defect is fixed
- the backend process and local JWT harness are valid

What is still blocked:

- catalog retrieval in real request flow
- audit history retrieval
- TIN verification history retrieval
- EFRIS outbox visibility
- onboarding write-path execution

All of those still depend on a real database-backed runtime, not just a running mock-mode server.

## Recommended Next Step

Do not spend more time trying to prove broader URA execution on pure mock mode alone.

The next high-value move is:

1. provide valid local backend/frontend env files and `DATABASE_URL`
2. start a real DB-backed local backend target
3. rerun the June 26 onboarding safe-path harness
4. rerun focused authenticated URA admin probes against that DB-backed target
