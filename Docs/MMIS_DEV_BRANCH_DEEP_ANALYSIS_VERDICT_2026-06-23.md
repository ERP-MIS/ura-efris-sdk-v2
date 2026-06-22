# MMIS Dev Branch Deep Analysis And Verdict

Date: 2026-06-23

Scope:

- Backend dev branch: `MarketMasterApi` `origin/dev` at `f8a355d9d10d261a9bf1067b5e683de91659d955`
- Frontend dev branch: `MarketMasterERP` `origin/dev` at `008d31559644278593e3e15203a7ad5b2962bad1`
- Local API collection artifacts under `Docs/`
- Local URA, EFRIS, InstantTIN, smoke/UAT docs already present in this workspace

Source-of-truth rule used for this audit:

1. current backend and frontend dev worktrees under `.branch-work/.../_audit/...`
2. current backend/frontend route and service code
3. local collection artifacts and local runbooks
4. older narrative docs only when they still match current code

## 1. Executive Verdict

Verdict: `PARTIALLY READY / BLOCKED FOR CLEAN UAT HANDOFF`

Why:

- The backend dev branch is materially ahead of the collection/docs baseline and validates cleanly from this workspace for its URA unit tests and production bundle build.
- The frontend dev branch also builds successfully, but key onboarding and URA/EFRIS refinements are uneven: some flows are production-shaped, while other user-facing onboarding/KYC surfaces are still mock-driven or not wired to the backend contract that now exists.
- The main local backend Postman collection is a subset of the live backend route surface. It is not stale in the dangerous sense of documenting removed endpoints, but it is incomplete for current `dev`.
- URA Partner Service integration is real and code-backed. InstantTIN is still only a local evidence/SDK surface and is not integrated into runtime MMIS code.

Practical conclusion:

- `MarketMasterApi` `dev` is strong enough for continued backend URA/EFRIS delivery work.
- `MarketMasterERP` `dev` is not yet a fully coherent operational surface for all onboarding and URA/EFRIS user journeys.
- The current collections are usable for smoke and admin URA coverage, but not as a complete contract set for current backend `dev`.

## 2. Validation Performed

Backend `MarketMasterApi` `origin/dev`:

- `npm test` -> passed: `17 URA unit test(s) passed.`
- `npm run build` -> passed

Frontend `MarketMasterERP` `origin/dev`:

- `npm run build` -> passed after rerunning outside the sandbox
- Initial in-sandbox attempt failed with `Error: spawn EPERM`, which was environment-related rather than a code failure
- Production build completed, but Vite reported a large-chunk warning for `assets/index-CZ7Ve-Zp.js` at about `2.09 MB`

## 3. Backend Route Inventory

Deduped backend route surface found from current `MarketMasterApi` `origin/dev`: `192` endpoints.

Largest route groups by file:

| Route file | Endpoint count | Notes |
| --- | ---: | --- |
| `src/routes/payment.routes.js` | 40 | rent, invoices, claims, refunds, webhook ops |
| `src/routes/ura.routes.js` | 34 | TIN, PRN, catalogs, audit, EFRIS outbox |
| `src/routes/auth.routes.js` | 27 | auth, verification, password, claim flows |
| `src/routes/product.routes.js` | 12 | mounted twice, creating alias paths |
| `src/routes/supplier.routes.js` | 12 | mounted twice, creating admin and superadmin aliases |
| `src/routes/vendor.routes.js` | 12 | mounted twice, creating onboarding and admin aliases |

Important inventory nuance:

- `ura.routes.js` is mounted under both `/api/ura` and `/api/admin/ura`
- `product.routes.js` is mounted under both `/api/vendors` and `/api/products`
- `vendor.routes.js` and `supplier.routes.js` are also mounted under multiple prefixes

This is not automatically wrong, but it increases documentation drift risk and can make role boundaries less obvious in collections and client code.

## 4. Collection Comparison

Collection counts extracted locally:

| Artifact | Endpoint count | Notes |
| --- | ---: | --- |
| backend code inventory | 192 | current `MarketMasterApi` `origin/dev` |
| `Docs/MMIS_Backend_Postman_Collection.json` | 146 | strongest local MMIS backend collection |
| `Docs/MMIS_Hoppscotch_Backend_Smoke_Cleaned.json` | 57 | smoke/UAT subset |
| `Docs/Market Master Api_for_sabit.json` | 47 | mixed/stale sample-state Hoppscotch export |
| `Docs/MMIS_URA_Provider_UAT_Only_Hoppscotch.json` | 3 | direct URA provider-side calls, not MMIS app routes |

Normalized code-vs-collection comparison:

| Comparison | Code endpoints | Collection endpoints | Code-only | Collection-only | Read |
| --- | ---: | ---: | ---: | ---: | --- |
| backend code vs `MMIS_Backend_Postman_Collection.json` | 191 normalized | 146 normalized | 45 | 0 | collection is a subset of code |
| backend code vs `MMIS_Hoppscotch_Backend_Smoke_Cleaned.json` | 191 normalized | 57 normalized | 134 | 0 | smoke pack is intentionally narrow |

High-confidence interpretation:

- The main backend Postman collection does not invent routes that current code removed.
- The drift is one-directional: current backend `dev` has materially more live routes than the collection documents.

Largest missing categories from the main Postman collection:

1. delivery module endpoints under `/api/deliveries`
2. EFRIS outbox admin/runtime endpoints under `/api/ura/efris-outbox` and `/api/admin/ura/efris-outbox`
3. rent-payment PRN endpoints:
   - `POST /api/ura/prn/rent-payment`
   - `POST /api/ura/rent-payments/:invoicePaymentId/prn`
   - admin aliases under `/api/admin/ura/...`
4. several alias surfaces created by duplicate route mounts:
   - `/api/superadmin/suppliers...`
   - `/api/superadmin/vendors...`
   - `/api/products/:vendorId/products`
   - `/api/vendors/:productId`

Machine-readable diff export:

- `Docs/MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv`

## 5. Onboarding / KYC / URA / InstantTIN Status

### 5.1 Vendor onboarding and KYC

Status: `PARTIAL, SPLIT BETWEEN REAL AND MOCKED SURFACES`

What is real:

- Backend `POST /api/vendors/setup-shop` is live and expects a multipart payload with:
  - `marketId`
  - `shopName`
  - `monthlyRent`
  - `taxIdNumber`
  - `tinDocument`
  - optional `stallNumber`
- The backend verifies the TIN through `uraService.verifyTin(...)` before activating the shop flow.
- The frontend `components/onboarding/VendorShopSetup.tsx` is wired to that backend contract through `ApiService.vendors.setupShop(...)`.

What is still weak:

- `components/VendorKYCForm.tsx` is still a purely local document picker with a `setTimeout` mock submit and no backend upload or persistence.
- `components/onboarding/OnboardingWizard.tsx` still uses that mock KYC component for the generic identity step, which makes the overall onboarding story inconsistent.

Meaning:

- The vendor shop/TIN setup path is real.
- The broader FE onboarding/KYC UX is only partially refactored and still contains placeholder behavior.

### 5.2 Supplier onboarding and KYC

Status: `BACKEND READY, FRONTEND PARTIALLY WIRED, PUBLIC SIGNUP BROKEN`

What is real:

- Backend supplier signup schema now requires `taxId`.
- Backend supplier profile setup endpoint requires multipart `businessName`, `businessType`, `taxIdNumber`, and `tinDocument`.
- Backend supplier create/update flows can trigger `uraService.verifyTin(...)`.

What is missing or broken:

- Frontend public signup in `components/auth/AuthPage.tsx` calls `ApiService.auth.registerSupplier(...)` without sending `taxId`.
- Current FE signup form state does not expose a `taxId` field at all.
- FE has an API helper for multipart supplier setup (`ApiService.vendors.setupSupplier`), but no discovered current UI consumer for it in the dev worktree.

Meaning:

- Supplier onboarding refactoring is incomplete.
- The backend contract has moved ahead of the current FE public supplier signup surface.

### 5.3 Approval gating for vendor/supplier roles

Status: `IMPLEMENTED ON BACKEND, PARTIALLY ENFORCED IN FE`

Implemented backend behavior:

- Vendor/supplier approval is blocked unless the latest TIN verification status is `VERIFIED` or `OVERRIDDEN`.
- This gate exists in backend approval logic and in URA helper service checks.

Implemented frontend behavior:

- Pending approvals UI uses `TinVerificationBadge` and disables approval actions when TIN status is not approval-ready.
- Vendor and supplier data grids now expose tax-compliance and latest-verification context.

Meaning:

- The compliance rule is now real, not just visual.
- However, some onboarding entry flows still do not collect the backend-required data early enough.

### 5.4 URA Partner Service

Status: `IMPLEMENTED`

Confirmed live backend surfaces:

- health
- TIN verification
- PRN detail lookup
- generic PRN generation
- rent-payment PRN generation
- MDA catalog lookup
- MDA tax-code lookup
- tax-head detail
- request audit list/detail/retry
- TIN verification history and override
- EFRIS outbox list, enqueue, retry

Frontend status:

- Admin URA console exists and covers status, TIN, PRN, catalog, and request audit tabs.
- FE client includes URA health, verify, generic PRN, catalog, audit request, and TIN override methods.
- FE client does not currently expose the backend EFRIS outbox endpoints.
- FE docs explicitly note that no frontend EFRIS queue screen exists yet.

### 5.5 URA route validation

Status: `IMPLEMENTED`

Confirmed:

- shared validation middleware supports segmented `body`, `params`, and `query`
- `ura.validation.js` defines segmented schemas for most URA operations
- URA route group uses that middleware consistently for verification, PRN, audit, and outbox endpoints

This is one of the stronger refinements in current backend `dev`.

### 5.6 InstantTIN API / SDK

Status: `NOT INTEGRATED INTO MMIS RUNTIME`

Confirmed:

- local InstantTIN SDK and docs exist in the workspace
- local InstantTIN endpoint and auth evidence reports are present
- no current backend runtime code path in `MarketMasterApi` `origin/dev` imports or wraps the InstantTIN SDK
- current MMIS TIN verification path is still based on URA Partner Service `get-tin-details`

Meaning:

- InstantTIN is a researched candidate integration family, not a shipped MMIS integration in current `dev`

## 6. Request / Response Model Recheck

Confidence: `VERY HIGH` for the following contracts because they are derived directly from current route, validation, controller, and FE client code.

### 6.1 `POST /api/auth/register-supplier`

Expected backend body:

```json
{
  "name": "string",
  "email": "user@example.com",
  "password": "min-8-char-password",
  "businessName": "string",
  "marketId": "uuid-or-market-id",
  "taxId": "URA TIN",
  "supplierType": "WHOLESALER|DISTRIBUTOR|MANUFACTURER|IMPORTER|LOCAL_PRODUCER",
  "licenseNumber": "optional",
  "warehouseAddress": "optional",
  "deliveryRadius": 10,
  "minimumOrder": 50000,
  "paymentTerms": "optional"
}
```

Current FE request body is incomplete because it does not send `taxId`.

### 6.2 `POST /api/vendors/setup-shop`

Expected backend request type: `multipart/form-data`

Required fields:

- `marketId`
- `shopName`
- `monthlyRent`
- `taxIdNumber`
- `tinDocument`

Optional:

- `stallNumber`

Current FE vendor shop setup appears aligned to this contract.

### 6.3 `POST /api/suppliers/setup-profile`

Expected backend request type: `multipart/form-data`

Required fields:

- `businessName`
- `taxIdNumber`
- `tinDocument`

Optional/currently accepted:

- `businessType`

Current FE has an API helper for this payload, but no discovered current user-facing screen that drives it.

### 6.4 `POST /api/ura/tin/verify`

Expected JSON body:

```json
{
  "tin": "string",
  "submittedName": "optional string",
  "entityType": "VENDOR|SUPPLIER",
  "vendorId": "optional uuid",
  "supplierId": "optional uuid",
  "stakeholderId": "optional uuid"
}
```

Expected success envelope:

```json
{
  "success": true,
  "message": "TIN verification completed",
  "data": {
    "id": "uuid",
    "status": "VERIFIED|MISMATCH|INVALID|FAILED|OVERRIDDEN|PENDING",
    "normalizedTin": "string"
  }
}
```

### 6.5 `POST /api/ura/prn/generate`

Expected JSON body:

```json
{
  "amount": 150000,
  "tin": "string",
  "taxHead": "string",
  "taxPayerName": "optional string",
  "assessmentDate": "optional string",
  "paymentType": "optional string",
  "referenceNo": "optional string",
  "paymentPeriod": "optional string",
  "expiryDays": 30,
  "vendorId": "optional uuid",
  "supplierId": "optional uuid",
  "stakeholderId": "optional uuid",
  "marketId": "optional uuid",
  "invoiceId": "optional uuid",
  "taxPaymentId": "optional uuid"
}
```

### 6.6 Error model

Segmented validation failures are consistently shaped as:

```json
{
  "success": false,
  "message": "Validation error",
  "errors": [
    {
      "path": "body.tin",
      "field": "tin",
      "message": "TIN is required",
      "code": "too_small"
    }
  ]
}
```

## 7. Issues And Fix Suggestions

| Priority | Area | Issue | Evidence | Suggested fix |
| --- | --- | --- | --- | --- |
| P1 | FE signup | Supplier signup UI does not send required `taxId` | FE `AuthPage.tsx` omits it; BE `auth.validation.js` requires it | add `taxId` to the supplier signup schema, form state, UI, and request payload |
| P1 | FE onboarding | Generic KYC component is still mock-only | `VendorKYCForm.tsx` uses local file state and `setTimeout` | replace with real multipart upload/persist flow or remove it from primary onboarding until implemented |
| P1 | FE parity | No discovered UI consumer for multipart supplier setup | FE API helper exists; no current screen calls it | add a supplier setup screen after email verification/login or collapse that contract into signup if intended |
| P1 | FE URA/EFRIS parity | FE URA client lacks EFRIS outbox methods and screen coverage | backend has outbox routes; FE docs explicitly say queue screen is absent | add FE API methods and an admin outbox queue view before claiming EFRIS operator readiness |
| P1 | Collection drift | Main backend Postman collection is missing 45 normalized current-code routes | code inventory 191 normalized vs collection 146 | regenerate collection from current routes and tag missing route groups by module |
| P2 | Route alias sprawl | Duplicate mounts create multiple equivalent endpoint families | `/api/ura` and `/api/admin/ura`; `/api/vendors` and `/api/products` | decide which aliases are canonical and document/deprecate the rest |
| P2 | FE bundle size | Frontend production bundle emits a large-chunk warning | Vite build warning on `index` bundle ~2.09 MB | split dashboard/admin surfaces with dynamic imports and review heavy module boundaries |
| P2 | UAT surface clarity | `Market Master Api_for_sabit.json` contains hardcoded sample state and is not a clean contract artifact | real IDs/tokens and placeholder endpoint gaps | archive it as legacy/sample-only and stop treating it as a source-of-truth collection |
| P3 | InstantTIN delivery | InstantTIN evidence is present but runtime integration is absent | local SDK/docs only, no runtime imports | keep InstantTIN status explicitly marked “research/integration candidate” until a real adapter exists |

## 8. Hoppscotch / Postman Testing Status

Current local testing assets:

- `Docs/MMIS_Backend_Postman_Collection.json`
- `Docs/MMIS_Hoppscotch_Backend_Smoke_Cleaned.json`
- `Docs/MMIS_URA_Provider_UAT_Only_Hoppscotch.json`
- `Docs/LOCAL_SMOKE_UAT_RUNBOOK.md`

High-confidence procedure from current workspace:

1. Start backend in `mock` mode for local smoke.
2. Use the MMIS backend Postman collection for broader app API coverage.
3. Use the cleaned Hoppscotch smoke set for a narrower happy-path pass.
4. Use the URA-provider Hoppscotch set only for direct URA gateway verification, not as proof of MMIS app-route readiness.
5. For sandbox/UAT, follow the existing runbook and keep frontend config proxy-only; do not expose URA secrets in FE config.

Important limitation:

- Hoppscotch assets do not yet cover the full current backend route surface, especially delivery and EFRIS-outbox additions.

## 9. Overall Plan Of Execution

1. Fix FE supplier signup contract drift first.
   - add `taxId` UI/state/validation
   - verify successful signup against current BE schema

2. Unify onboarding/KYC surfaces.
   - replace mock-only KYC submission behavior
   - explicitly choose between “signup-time data capture” and “post-login setup flow”

3. Expose backend EFRIS outbox parity in FE admin tools.
   - add FE API methods
   - add queue list/retry surface

4. Regenerate the canonical backend API collection from current `dev`.
   - include delivery, outbox, rent-payment PRN, and admin alias decisions

5. Decide and document canonical endpoints.
   - especially for URA/admin URA and product/vendor duplicate mounts

6. Keep InstantTIN out of “implemented” status until a real MMIS adapter exists.
   - if it is a roadmap item, define explicit ticketed scope instead of leaving it implied by docs

## 10. Bottom Line

Backend `dev` is ahead of frontend and collections in a useful way: the URA Partner Service, approval gates, route validation, and EFRIS outbox skeleton are real. The frontend `dev` branch has meaningful URA progress, but onboarding and operator parity are still inconsistent. The strongest immediate corrective action is not new backend work; it is closing the FE contract gaps and regenerating the canonical API collection from the current backend route inventory.
