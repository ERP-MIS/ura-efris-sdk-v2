# MMIS June 26 Current Truth Addendum

Date: 2026-06-26
Workspace: `C:\FTS\NovoRepo_FOITS\MMIS-URA-FTS-DEV`

## Purpose

This addendum narrows the current authoritative truth where some broader June 24-25 dossiers still describe older frontend onboarding and EFRIS-admin assumptions.

Use this addendum when there is any conflict with:

- `MMIS_CURRENT_UAT_AND_UI_API_EVAL_REFRESH_2026-06-24.md`
- `MMIS_COMPREHENSIVE_API_UAT_AND_SELF_EVAL_DOSSIER_2026-06-25.md`
- `MMIS_CONSOLIDATED_API_UAT_AND_FLOW_MASTER_DOC_2026-06-25.md`

## Current Truth Corrections

### 1. Active KYC route is no longer mock-only

Current live frontend truth:

- dashboard `KYC` route now uses the live role-specific onboarding path
- vendor users go to `POST /api/vendors/setup-shop`
- supplier users go to `POST /api/suppliers/setup-profile`

This means the old statement "`generic KYC flow is still mock-only`" is no longer accurate for the active dashboard KYC route.

More precise statement:

`THE STANDALONE LEGACY VendorKYCForm STILL CONTAINS A MOCK SUBMIT PATH IN ISOLATION, BUT THE ACTIVE DASHBOARD KYC ROUTE HAS BEEN REROUTED TO LIVE BACKEND-BACKED VENDOR AND SUPPLIER ONBOARDING.`

Authoritative companion artifacts:

- [MMIS_ONBOARDING_PERSISTENCE_AND_KYC_ROUTE_VERDICT_2026-06-25.md](./MMIS_ONBOARDING_PERSISTENCE_AND_KYC_ROUTE_VERDICT_2026-06-25.md)
- [MMIS_CURRENT_ONBOARDING_SAFE_PATH_UAT_GUIDE_2026-06-26.md](./MMIS_CURRENT_ONBOARDING_SAFE_PATH_UAT_GUIDE_2026-06-26.md)

### 2. Generic Applications flow is still not live

Current frontend still exposes:

- `POST /api/applications/role-access`
- `GET /api/applications/me`

Current backend router does not mount `/api/applications/*`.

So the correct execution truth is:

`UserApplications` is not a safe current UAT surface and must not be treated as implemented end to end.

Authoritative companion artifacts:

- [MMIS_COLLECTION_AND_ONBOARDING_RECONCILIATION_ADDENDUM_2026-06-25.md](./MMIS_COLLECTION_AND_ONBOARDING_RECONCILIATION_ADDENDUM_2026-06-25.md)
- [MMIS_FRONTEND_BACKEND_ONBOARDING_API_MISMATCH_MATRIX_2026-06-25.csv](./MMIS_FRONTEND_BACKEND_ONBOARDING_API_MISMATCH_MATRIX_2026-06-25.csv)

### 3. FE EFRIS operator parity gap is no longer current truth

The older statement "`backend ready / FE EFRIS operator gap remains`" is no longer current for this workspace.

Current live frontend admin URA screen already exposes:

- `GET /api/admin/ura/efris-outbox`
- `POST /api/admin/ura/efris-outbox/enqueue-from-request`
- `POST /api/admin/ura/efris-outbox/:id/retry`

So the EFRIS admin/operator gap should not remain in June 26 execution guidance as an open frontend gap.

Authoritative companion artifact:

- [MMIS_FE_ACCESS_GATE_AND_EFRIS_OUTBOX_IMPLEMENTATION_UPDATE_2026-06-25.md](./MMIS_FE_ACCESS_GATE_AND_EFRIS_OUTBOX_IMPLEMENTATION_UPDATE_2026-06-25.md)

## Current Safe UAT Path

For current dev-branch execution, trust this path:

1. supplier public registration contract checks
2. seeded vendor login
3. seeded supplier login
4. `GET /api/auth/me`
5. vendor `KYC -> setup-shop`
6. supplier `KYC -> setup-profile`
7. admin `GET /api/admin/ura/tin-verifications`
8. admin `GET /api/admin/ura/efris-outbox`

Do not use the `Applications` dashboard flow as part of the current implemented-safe onboarding proof.

## What Remains Blocked

The remaining blockers are still runtime and environment truth, not route discovery:

- real `DATABASE_URL`
- real backend env
- real frontend env
- seeded vendor, supplier, and admin credentials
- valid market UUIDs
- accessible `tinDocument` path
- staged/shared-dev base URL
- URA sandbox credentials

## Practical Current Verdict

`COLLECTION PARITY IS REAL, ACTIVE KYC UAT PATH IS NOW ROLE-SPECIFIC AND BACKEND-BACKED, LEGACY APPLICATIONS FLOW REMAINS PHANTOM, AND FULL DB-BACKED OR STAGED UAT IS STILL BLOCKED BY RUNTIME INPUTS.`
