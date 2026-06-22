# MMIS Dev Branch Endpoint And Collection Inventory

Date: 2026-06-23

Code source:

- `MarketMasterApi` `origin/dev` at `f8a355d9d10d261a9bf1067b5e683de91659d955`

Collection source files:

- `Docs/MMIS_Backend_Postman_Collection.json`
- `Docs/MMIS_Hoppscotch_Backend_Smoke_Cleaned.json`
- `Docs/Market Master Api_for_sabit.json`
- `Docs/MMIS_URA_Provider_UAT_Only_Hoppscotch.json`

Machine-readable supporting files:

- [backend-dev-endpoints.json](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/_analysis/backend-dev-endpoints.json)
- [backend-collection-comparison.json](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/_analysis/backend-collection-comparison.json)
- [MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv](C:/FTS/NovoRepo_FOITS/DOWNLOADS/FTS-OITS/MMIS-URA/Docs/MMIS_DEV_BRANCH_API_COLLECTION_DIFF_2026-06-23.csv)

## 1. Inventory Summary

| Surface | Endpoint count | Meaning |
| --- | ---: | --- |
| backend code inventory | 192 | deduped routes discovered from current backend dev route mounts |
| main Postman collection | 146 | strongest local MMIS app-route collection |
| cleaned Hoppscotch smoke | 57 | smoke subset only |
| `Market Master Api_for_sabit.json` | 47 | mixed/legacy hoppscotch export |
| URA provider-only Hoppscotch | 3 | direct URA provider calls, not MMIS app routes |

## 2. Backend Route Groups

Grouped by mounted path family:

| Route group | Count | Notes |
| --- | ---: | --- |
| `/api/vendors` | 30 | includes vendor CRUD, onboarding, and product aliases |
| `/api/auth` | 27 | register, verify, claim, password, login flows |
| `/api/admin/ura` | 17 | admin alias of URA services |
| `/api/ura` | 17 | primary URA app surface |
| `/api/superadmin` | 12 | vendor/supplier superadmin aliases |
| `/api/market` | 10 | staff, tokens, and market-related routes |
| `/api/admin/payments` | 9 | payment operations and monitoring |
| `/api/suppliers` | 7 | supplier CRUD and setup-profile |
| `/api/admin/invoices` | 6 | invoice generation and retrieval |
| `/api/deliveries` | 6 | current delivery module |
| `/api/market-gates` | 6 | gate CRUD |
| `/api/marketLevels` | 6 | market level CRUD |
| `/api/markets` | 6 | market CRUD and lookup |
| `/api/products` | 6 | product alias routes |
| `/api/marketSections` | 5 | market section CRUD |
| `/api/shops` | 5 | shop CRUD |

## 3. Collection Coverage

### 3.1 Main Postman collection vs backend code

Normalized comparison result:

- backend code: `191`
- Postman collection: `146`
- code-only routes: `45`
- collection-only routes: `0`

Interpretation:

- the collection is incomplete rather than misleading
- current backend dev added route surface that was never reflected into the collection

Largest missing categories:

1. EFRIS outbox routes
2. delivery routes
3. rent-payment PRN routes
4. admin URA aliases
5. superadmin alias routes

### 3.2 Smoke Hoppscotch vs backend code

Normalized comparison result:

- backend code: `191`
- smoke collection: `57`
- code-only routes: `134`
- collection-only routes: `0`

Interpretation:

- this file is behaving like a deliberate smoke subset, not a full backend contract

### 3.3 `Market Master Api_for_sabit.json`

Read:

- mixed utility/testing collection
- contains real-looking IDs and placeholder state
- not suitable as the canonical current API contract

### 3.4 URA provider-only Hoppscotch

Contains only:

- `POST /oauth2/token`
- `POST /partner-service/2.0.0/mda-services/get-tin-details`
- `POST /partner-service/2.0.0/prn-services/generate-prn`

Interpretation:

- useful for direct URA gateway testing
- not evidence of MMIS backend route parity

## 4. High-Value Missing Routes From Main Postman Collection

These are the highest-signal code routes missing from `Docs/MMIS_Backend_Postman_Collection.json`:

| Method | Route | Backend file |
| --- | --- | --- |
| `GET` | `/api/ura/efris-outbox` | `src/routes/ura.routes.js` |
| `POST` | `/api/ura/efris-outbox/enqueue-from-request` | `src/routes/ura.routes.js` |
| `POST` | `/api/ura/efris-outbox/:id/retry` | `src/routes/ura.routes.js` |
| `POST` | `/api/ura/prn/rent-payment` | `src/routes/ura.routes.js` |
| `POST` | `/api/ura/rent-payments/:invoicePaymentId/prn` | `src/routes/ura.routes.js` |
| `GET` | `/api/admin/ura/efris-outbox` | `src/routes/ura.routes.js` |
| `GET` | `/api/admin/ura/health` | `src/routes/ura.routes.js` |
| `POST` | `/api/admin/ura/prn/rent-payment` | `src/routes/ura.routes.js` |
| `GET` | `/api/deliveries` | `src/routes/delivery.routes.js` |
| `PATCH` | `/api/deliveries/:id/dispatch` | `src/routes/delivery.routes.js` |
| `PATCH` | `/api/deliveries/:id/receive` | `src/routes/delivery.routes.js` |
| `PATCH` | `/api/deliveries/:id/verify` | `src/routes/delivery.routes.js` |

## 5. Inventory Verdict

Current backend dev has outgrown the main API collection. The collection is still useful, but it is no longer complete enough to serve as the only contract artifact for `dev`. The next canonical collection should be regenerated from current backend route inventory and explicitly include:

- URA admin aliases
- rent-payment PRN endpoints
- EFRIS outbox endpoints
- delivery module endpoints
- a decision on whether duplicate mount aliases remain supported or should be deprecated
