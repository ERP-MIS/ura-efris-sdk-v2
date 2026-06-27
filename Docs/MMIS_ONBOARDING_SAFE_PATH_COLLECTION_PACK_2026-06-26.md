# MMIS Onboarding Safe Path Collection Pack

Date: 2026-06-26

## Purpose

This pack is the current narrow UAT subset for the verified onboarding and URA admin safe path in the live workspace.

It is intentionally smaller than the older broad June 24-25 collections. The goal is to exercise only the routes that are confirmed to exist and align with the current frontend and backend implementation state.

## Included Files

- `MMIS_ONBOARDING_SAFE_PATH_UAT_POSTMAN_COLLECTION_2026-06-26.json`
- `MMIS_ONBOARDING_SAFE_PATH_UAT_POSTMAN_ENV_TEMPLATE_2026-06-26.json`
- `MMIS_ONBOARDING_SAFE_PATH_UAT_HOPPSCOTCH_COLLECTION_2026-06-26.json`
- `MMIS_ONBOARDING_SAFE_PATH_UAT_HOPPSCOTCH_ENV_TEMPLATE_2026-06-26.json`

## Verified Route Coverage

The generated safe-path collections include:

- `POST /api/vendors/setup-shop`
- `POST /api/suppliers/setup-profile`
- `GET /api/admin/ura/tin-verifications?limit=20`
- `GET /api/admin/ura/efris-outbox?limit=20`

The environment templates target:

- `http://127.0.0.1:5053`

## Why This Pack Exists

Current code truth is narrower than some earlier analysis artifacts implied:

- Vendor onboarding is backend-backed through `setup-shop`.
- Supplier onboarding is backend-backed through `setup-profile`.
- Admin URA verification audit is real.
- Admin EFRIS outbox listing is real and frontend parity now exists in the current workspace.
- Generic `/api/applications/*` onboarding flow is not part of this safe-path pack because it is not mounted in the current backend route surface.

## Use Order

1. Fill the environment template with valid local credentials and TIN evidence path.
2. Run vendor or supplier setup against a clean local MMIS backend on `5053`.
3. Verify resulting TIN audit records through `/api/admin/ura/tin-verifications`.
4. Verify fiscal queue visibility through `/api/admin/ura/efris-outbox`.

## Remaining Blockers

- Real URA sandbox/UAT still depends on valid gateway credentials, any required device or certificate material, and a working backend database/runtime.
- These collections prove the current backend-routed MMIS surface, not direct provider-side URA credential success.
- `prisma validate` remained environment-blocked in the last local pass due the known engine-access approval limitation, not a confirmed schema error.

## Relationship To Older Artifacts

Use this pack for the current safe-path run.

Use older broad June 24-25 reports and collections as supporting audit evidence only:

- they cover more historical analysis
- they include some now-stale FE gap statements
- they are not the cleanest current UAT starting point
