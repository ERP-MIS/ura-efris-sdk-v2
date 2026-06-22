# MMIS Dev Branch Issue Matrix And Execution Plan

Date: 2026-06-23

Related artifacts:

- [MMIS_DEV_BRANCH_DEEP_ANALYSIS_VERDICT_2026-06-23.md](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_DEEP_ANALYSIS_VERDICT_2026-06-23.md)
- [MMIS_DEV_BRANCH_PAYLOAD_CONTRACT_RECHECK_2026-06-23.md](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_PAYLOAD_CONTRACT_RECHECK_2026-06-23.md)
- [MMIS_DEV_BRANCH_ENDPOINT_AND_COLLECTION_INVENTORY_2026-06-23.md](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_ENDPOINT_AND_COLLECTION_INVENTORY_2026-06-23.md)
- [MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv)
- [MMIS_DEV_BRANCH_API_COVERAGE_MATRIX_2026-06-23.csv](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_API_COVERAGE_MATRIX_2026-06-23.csv)
- [MMIS_DEV_FEATURE_LOCAL_BRANCH_DIFF_TABLE_2026-06-23.md](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_FEATURE_LOCAL_BRANCH_DIFF_TABLE_2026-06-23.md)

## 1. Release Readiness Summary

Overall verdict: `PARTIALLY READY / BLOCKED FOR CLEAN UAT HANDOFF`

Current readiness by surface:

| Surface | Status | Read |
| --- | --- | --- |
| Backend URA partner-service flows | ready for continued delivery | route, validation, controller, and test-backed |
| Backend EFRIS outbox slice | backend-ready, frontend-absent | queue and retry exist in API only |
| Vendor onboarding | partially ready | real shop + TIN setup exists, generic KYC UX is still mock-shaped |
| Supplier onboarding | not coherent end to end | BE requires tax data, FE public signup does not collect it |
| FE admin URA tooling | partially ready | TIN, PRN, catalog, audit tabs exist; outbox does not |
| API collections | incomplete | main collection lags current backend dev |
| InstantTIN integration | not implemented | docs and SDK present, runtime adapter absent |

## 2. Prioritized Issue Matrix

| Priority | Area | Current state | Evidence | Risk | Fix direction | Validation gate |
| --- | --- | --- | --- | --- | --- | --- |
| P1 | Supplier public signup | FE request omits required `taxId` while BE schema requires it | `AuthPage.tsx` request shape vs backend `auth.validation.js`; documented in payload recheck | supplier self-onboarding fails or cannot satisfy backend contract | add `taxId` field to FE state, form UI, client request, and any FE-side validation | successful `POST /api/auth/register-supplier` with current backend schema and no ad hoc payload patching |
| P1 | Supplier setup flow | multipart setup helper exists but no confirmed current UI consumer was found | FE helper exists in `services/api.ts`; no discovered screen driver | supplier cannot complete tax/TIN setup from the intended UX | create explicit post-login supplier setup screen or collapse required data into signup | user can complete signup and then reach a real `setup-profile` screen that persists TIN document and backend status |
| P1 | Vendor KYC consistency | generic KYC step still uses local-only mock submit behavior | `VendorKYCForm.tsx` uses local state and `setTimeout`; verdict doc records this | misleading onboarding state, no persisted compliance evidence | replace with real upload/persist flow or remove from primary onboarding until backend contract exists | document upload reaches backend and returned compliance state is reflected in the UI |
| P1 | FE EFRIS parity | backend EFRIS outbox exists, FE has no client methods or queue screen | backend routes in `ura.routes.js`; FE changelog and audit show no outbox support | operators cannot inspect or retry fiscalization queue items | add FE URA client methods for outbox list/enqueue/retry and an admin queue surface | queue items visible in FE and retry action reaches backend |
| P1 | Canonical API collection drift | main Postman collection covers 146 normalized routes vs 191 in current code | collection diff CSV and endpoint inventory report | QA/UAT operates against incomplete contract set | regenerate canonical collection from live route inventory and choose canonical aliases | exported collection includes delivery, rent-payment PRN, EFRIS outbox, and chosen admin aliases |
| P2 | Endpoint alias sprawl | same functionality is exposed under multiple mounted families | `/api/ura` + `/api/admin/ura`, `/api/vendors` + `/api/products`, superadmin aliases | client confusion, doc drift, auth-boundary ambiguity | declare canonical public/admin paths and deprecate or explicitly document the rest | route catalog annotated with canonical vs compatibility aliases |
| P2 | FE bundle size | production build completes with large main bundle warning | `npm run build` warning on ~2.09 MB main chunk | slower operator UI, harder future FE scaling | split heavy admin/dashboard surfaces using dynamic imports and review shared module boundaries | FE build still passes and main chunk size drops materially |
| P2 | Legacy sample collection usage | `Market Master Api_for_sabit.json` is not a clean source-of-truth contract artifact | inventory report classifies it as mixed/legacy sample export | stale IDs and sample assumptions leak into testing practice | archive it as legacy-only and stop using it as canonical contract evidence | README or docs explicitly mark it sample-only |
| P2 | Rent-payment PRN FE parity | backend rent-payment PRN endpoints exist, FE does not expose them | payload recheck shows backend-only status | rent PRN generation remains admin/backend-only despite backend support | add FE helper and operator flow only if product intent requires it | operator can generate rent-payment PRN from intended UI surface |
| P3 | InstantTIN scope ambiguity | local docs and SDK exist but runtime integration is absent | audit verdict, local InstantTIN docs, no runtime imports | project status can be overstated in demos or docs | keep status explicitly marked research-only until adapter, config, and tests exist | runtime code imports adapter and smoke/UAT evidence exists before status changes |

## 3. Strongest Code-Review Findings

These are the highest-signal findings from the current backend and frontend dev worktrees.

| Severity | Finding | Why it matters |
| --- | --- | --- |
| P1 | FE supplier signup is contract-broken against the current backend | current user-facing flow cannot satisfy required compliance inputs |
| P1 | Onboarding UX mixes real URA-backed TIN verification with mock-only KYC behavior | operator confidence and auditability are weaker than the UI implies |
| P1 | Backend EFRIS surface is ahead of frontend/admin operational tooling | the system is harder to operate and verify end to end |
| P1 | Canonical collection artifacts no longer represent current backend dev | test coverage and stakeholder expectations drift away from live code |
| P2 | Route aliases multiply equivalent endpoints | parity work, documentation, and client ownership become harder over time |
| P2 | FE delivery of new backend capabilities is inconsistent | backend progress is not automatically meaningful to operators or onboarding users |

## 4. Execution Plan

### Phase 1: Close Blocking FE/BE Contract Gaps

Goal:
Make supplier and vendor onboarding contracts coherent enough for clean smoke/UAT.

Work:

1. add `taxId` to supplier public signup FE flow
2. decide whether supplier tax/TIN capture is signup-time or post-login setup-time
3. replace or temporarily remove mock-only KYC submit behavior from the primary onboarding wizard
4. verify that returned backend TIN/compliance state is actually rendered after submission

Exit criteria:

- supplier signup no longer sends a payload the backend rejects by schema
- no primary onboarding screen implies persistence when only local mock state exists

### Phase 2: Expose Backend-Ready Operator Surfaces

Goal:
Bring frontend admin capability closer to the actual backend URA/EFRIS feature set.

Work:

1. add FE client methods for EFRIS outbox list, enqueue, and retry
2. add an admin queue screen or tab for EFRIS outbox operations
3. decide whether rent-payment PRN generation must be exposed in FE now or remain backend/admin-only

Exit criteria:

- operator can inspect backend queue state from FE
- retry path can be exercised without direct API tooling

### Phase 3: Regenerate Canonical Contract Artifacts

Goal:
Make collections and route documentation match current backend dev.

Work:

1. use current route inventory as the canonical source
2. regenerate the main Postman collection for live backend `dev`
3. annotate or drop legacy/sample collections that are no longer contract-grade
4. explicitly mark canonical vs compatibility aliases

Exit criteria:

- main collection no longer misses delivery, EFRIS outbox, and rent-payment PRN routes
- testing packs are clearly labeled as full-contract vs smoke-only

### Phase 4: Quality And Maintainability Pass

Goal:
Reduce drift and improve confidence before wider rollout.

Work:

1. review route alias policy and reduce ambiguity where feasible
2. trim FE bundle weight on admin-heavy surfaces
3. keep InstantTIN status explicitly non-runtime until a real adapter, config path, and tests exist

Exit criteria:

- route ownership and canonical paths are documented
- FE build still passes after chunk-splitting work
- docs no longer imply shipped InstantTIN support

## 5. Minimal Verification Matrix

These are the smallest credible checks that should prove the fixes rather than just suggest them.

| Area | Minimum credible check |
| --- | --- |
| Backend build health | `npm run build` in backend dev worktree |
| Backend URA regression | `npm test` in backend dev worktree with URA tests passing |
| FE release health | `npm run build` in frontend dev worktree |
| Supplier signup contract | exercise FE request shape against current backend schema and confirm `taxId` presence |
| Supplier setup flow | confirm a real FE screen drives `multipart/form-data` to `POST /api/suppliers/setup-profile` |
| Vendor KYC | confirm any remaining KYC step persists to backend or is clearly marked as not yet implemented |
| EFRIS outbox FE parity | verify queue list and retry actions call backend endpoints successfully |
| Collection parity | compare regenerated collection against current backend inventory and show zero collection-only stale routes |

## 6. Recommended Order Of Work

1. fix supplier signup contract drift
2. decide and wire supplier setup ownership
3. remove or replace mock-only onboarding KYC behavior
4. expose EFRIS outbox in FE admin tooling
5. regenerate canonical collections from live backend routes
6. document canonical endpoint families
7. do bundle-size cleanup and non-blocking maintainability work

## 7. Final Recommendation

Do not present the current dev branches as fully UAT-ready for onboarding and fiscal operations.

A defensible status line is:

`backend URA/EFRIS foundation is real and test-backed; frontend onboarding parity and operator tooling still need targeted completion before a clean end-to-end UAT handoff`
