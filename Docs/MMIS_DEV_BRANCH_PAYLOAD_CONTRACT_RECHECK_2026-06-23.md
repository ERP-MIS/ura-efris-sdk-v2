# MMIS Dev Branch Payload Contract Recheck

Date: 2026-06-23

Confidence: `VERY HIGH`

Why:

- derived from current backend `origin/dev` route files, validation schemas, controllers, and FE API-client usage
- not inferred from older screenshots or narrative docs

## 1. Supplier Public Signup

Backend route:

- `POST /api/auth/register-supplier`

Backend contract from `src/validations/auth.validation.js`:

```json
{
  "name": "string",
  "email": "user@example.com",
  "password": "minimum 8 chars",
  "businessName": "string",
  "marketId": "string",
  "taxId": "URA TIN",
  "supplierType": "WHOLESALER|DISTRIBUTOR|MANUFACTURER|IMPORTER|LOCAL_PRODUCER",
  "licenseNumber": "optional",
  "warehouseAddress": "optional",
  "deliveryRadius": 10,
  "minimumOrder": 50000,
  "paymentTerms": "optional"
}
```

Observed FE request from `components/auth/AuthPage.tsx`:

```json
{
  "name": "string",
  "email": "string",
  "password": "string",
  "phoneNumber": "string",
  "marketId": "string",
  "role": "Supplier",
  "supplierType": "string",
  "businessName": "string"
}
```

Alignment verdict:

- `supplierType` and `businessName` are aligned
- `taxId` is missing from FE
- `phoneNumber` and `role` are sent by FE but not part of the backend-required schema

Status:

- `BROKEN CONTRACT`

## 2. Vendor Shop Setup

Backend route:

- `POST /api/vendors/setup-shop`

Request type:

- `multipart/form-data`

Backend required fields:

- `marketId`
- `shopName`
- `monthlyRent`
- `taxIdNumber`
- `tinDocument`

Backend optional fields:

- `stallNumber`

Observed FE request from `services/api.ts` and `components/onboarding/VendorShopSetup.tsx`:

- uses `FormData`
- sends `marketId`
- sends `shopName`
- sends `monthlyRent`
- sends `taxIdNumber`
- sends `tinDocument`
- optionally sends `stallNumber`

Alignment verdict:

- `ALIGNED`

Expected success envelope:

```json
{
  "success": true,
  "message": "Shop and stall setup completed successfully",
  "data": {
    "shop": {},
    "stall": {},
    "user": {
      "id": "string",
      "email": "string",
      "role": "Vendor",
      "vendorId": "uuid",
      "kycStatus": "VERIFIED",
      "taxComplianceStatus": "TIN_VERIFIED"
    },
    "tinVerification": {
      "status": "VERIFIED|OVERRIDDEN"
    }
  }
}
```

Expected failure modes:

- missing `taxIdNumber`
- missing `tinDocument`
- failed URA TIN verification

## 3. Supplier Profile Setup

Backend route:

- `POST /api/suppliers/setup-profile`

Request type:

- `multipart/form-data`

Backend required fields:

- `businessName`
- `taxIdNumber`
- `tinDocument`

Backend optional fields:

- `businessType`

Observed FE API helper:

- `ApiService.vendors.setupSupplier(...)`

Observed FE screen consumer:

- no current confirmed dev UI consumer found in this pass

Alignment verdict:

- helper contract exists
- user-facing flow not clearly wired

Status:

- `PARTIALLY WIRED`

## 4. URA TIN Verification

Backend route:

- `POST /api/ura/tin/verify`

Validated backend body:

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

Observed FE request:

- `ApiService.ura.verifyTin(...)` matches this shape

Alignment verdict:

- `ALIGNED`

Expected success envelope:

```json
{
  "success": true,
  "message": "TIN verification completed",
  "data": {
    "id": "uuid",
    "tin": "string",
    "normalizedTin": "string",
    "status": "PENDING|VERIFIED|MISMATCH|INVALID|FAILED|EXPIRED|OVERRIDDEN",
    "submittedName": "optional string",
    "uraTaxpayerName": "optional string",
    "failureReason": "optional string"
  }
}
```

## 5. URA Generic PRN Generation

Backend route:

- `POST /api/ura/prn/generate`

Validated backend body:

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

Observed FE request:

- `ApiService.ura.generatePrn(...)` matches the current backend shape closely

Alignment verdict:

- `ALIGNED`

## 6. URA Rent-Payment PRN Generation

Backend routes:

- `POST /api/ura/prn/rent-payment`
- `POST /api/ura/rent-payments/:invoicePaymentId/prn`

Validated backend schema in current dev:

```json
{
  "invoicePaymentId": "uuid",
  "force": true
}
```

Important nuance:

- current `rentPaymentPrnSchema` is effectively a flat object with `invoicePaymentId` and `force`
- the route `/rent-payments/:invoicePaymentId/prn` reads from `req.validated.invoicePaymentId`, which works for the flat schema but is not expressed as a segmented `params` object
- this is functional in current tests, but not as structurally explicit as the other URA schemas

Observed FE support:

- no current FE API helper or admin UI action discovered for this specific rent-payment PRN route

Status:

- `BACKEND IMPLEMENTED / FE NOT EXPOSED`

## 7. URA Audit And Override Flows

Routes and alignment:

| Route | Backend | FE client | Status |
| --- | --- | --- | --- |
| `GET /admin/ura/requests` | yes | yes | aligned |
| `GET /admin/ura/requests/:id` | yes | yes | aligned |
| `POST /admin/ura/requests/:id/retry` | yes | yes | aligned |
| `GET /admin/ura/tin-verifications` | yes | yes | aligned |
| `POST /admin/ura/tin-verifications/:id/override` | yes | yes | aligned |

## 8. EFRIS Outbox

Backend routes:

- `GET /api/ura/efris-outbox`
- `POST /api/ura/efris-outbox/enqueue-from-request`
- `POST /api/ura/efris-outbox/:id/retry`
- admin aliases under `/api/admin/ura/...`

Backend validated request bodies:

Enqueue:

```json
{
  "uraServiceRequestId": "uuid",
  "workflowType": "INVOICE_SUBMISSION",
  "payload": {},
  "metadata": {}
}
```

Retry:

```json
{
  "reason": "minimum 5 chars"
}
```

Observed FE support:

- no current FE API methods for EFRIS outbox
- no current admin queue screen in current dev

Status:

- `BACKEND ONLY`

## 9. Validation Error Shape

Current backend validation middleware returns:

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

This is one of the more consistent contracts in current backend dev and should be preserved as the canonical validation error model.

## 10. Contract Verdict

| Flow | Verdict |
| --- | --- |
| vendor shop setup | aligned |
| URA TIN verify | aligned |
| URA generic PRN | aligned |
| URA audit/override | aligned |
| supplier public signup | broken FE/BE contract |
| supplier multipart setup | helper exists, UI parity incomplete |
| rent-payment PRN | backend only in current dev |
| EFRIS outbox | backend only in current dev |
