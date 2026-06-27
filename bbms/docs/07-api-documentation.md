# API Documentation — BBMS REST API

Base URL: `/api/v1`. JSON over HTTPS. Versioned. Contract types shared via
`packages/types` (zod schemas) so frontend and backend never drift.

## Conventions
- **Auth:** `Authorization: Bearer <access_token>`; refresh via httpOnly cookie.
- **Content type:** `application/json` (multipart for uploads).
- **IDs:** UUID v4.
- **Money:** integer paise. **Timestamps:** ISO-8601 UTC.
- **Pagination:** `?page=1&limit=25&sort=-created_at` → response `{ data, meta:{ page,
  limit, total, totalPages } }`.
- **Filtering:** `?status=pending&blood_group=O_POS&q=mehta`.
- **Idempotency:** money/issue POSTs accept `Idempotency-Key` header.
- **Errors:** consistent envelope
  ```json
  { "error": { "code": "VALIDATION_ERROR", "message": "…", "details": [ ... ] } }
  ```
  Codes: `UNAUTHENTICATED` 401, `FORBIDDEN` 403, `NOT_FOUND` 404,
  `VALIDATION_ERROR` 422, `CONFLICT` 409, `RATE_LIMITED` 429, `SERVER_ERROR` 500.
- **RBAC:** every route declares `{ module, action }`; middleware enforces it.

## Standard CRUD shape (applies to most resources)
```
GET    /<resource>            list (paginated, filterable)
POST   /<resource>            create
GET    /<resource>/:id        read
PATCH  /<resource>/:id        update
DELETE /<resource>/:id        soft-delete (undoable)
POST   /<resource>/:id/restore  undo delete
```

## Auth
```
POST /auth/login                { email, password } → { user, accessToken } (+refresh cookie)
POST /auth/otp/verify           { challengeId, code } → tokens
POST /auth/refresh              (cookie) → { accessToken }
POST /auth/logout               → 204
POST /auth/forgot-password      { email } → 202
POST /auth/reset-password       { token, password } → 204
GET  /auth/me                   → current user + permissions
```

## Donors
```
GET  /donors                    ?q&blood_group&status&eligibility&city&page&limit
POST /donors                    create (multipart: photo)
GET  /donors/:id                detail + computed eligibility
PATCH/donors/:id                update
DELETE /donors/:id              soft-delete
GET  /donors/:id/donations      donation history
GET  /donors/:id/card           → donor card PDF (QR)
POST /donors/import             CSV bulk import → { imported, errors[] }
GET  /donors/export             ?format=csv|xlsx
```

## Collection
```
GET  /collections               ?status&donor_id&camp_id&from&to
POST /collections               { donorId, bagNumber, volumeMl, donationType, source, vitals }
GET  /collections/:id
PATCH/collections/:id/status    { status, reason? }   (state-machine guarded)
```

## Lab
```
GET  /lab/queue                 quarantined bags awaiting tests
POST /lab/:collectionId/tests   { hb, hiv, hbsag, hcv, malaria, syphilis, typing, comments }
POST /lab/:collectionId/approve { doctorId } → guarded; reactive ⇒ 409 (auto-rejected)
POST /lab/:collectionId/reject  { reason }
POST /lab/cross-match           { requestId, componentId, patientSampleId } → result
```

## Components
```
POST /collections/:id/separate  { components:[{ type, volumeMl, storageLocation }] }
GET  /components                ?type&blood_group&status&expiringInDays
GET  /components/:id            full traceability (donor↔bag↔component↔issue)
GET  /components/:id/label      → barcode/QR label PDF
```

## Inventory
```
GET  /inventory                 snapshot grid: by group × component (available/reserved/…)
GET  /inventory/expiring        ?days=7
GET  /inventory/ledger          ?component_id  audit trail of stock movements
POST /inventory/transfer        { componentId, toBranchId }   inter-branch (in-transit)
POST /inventory/adjust          { componentId, reason }       manual correction (audited)
```

## Patients
```
GET/POST/GET:id/PATCH:id        standard CRUD
GET  /patients/:id/history      requests + issues
```

## Requests
```
GET  /requests                  ?status&priority&hospital_id
POST /requests                  { patientId, hospitalId, doctorId, bloodGroup,
                                  componentType, units, priority, source }
GET  /requests/:id              + availability (compatible units)
POST /requests/:id/approve      → reserves units (guarded by stock + compatibility)
POST /requests/:id/reject       { reason }
POST /requests/:id/cancel       → releases reservations
```

## Issue
```
POST /issues                    { requestId, componentBarcodes:[…] }  (Idempotency-Key)
                                guards: compatibility, cross-match, expiry, status
                                effects: component→issued, ledger−1, invoice line
GET  /issues                    ?from&to&hospital_id&patient_id
GET  /issues/:id
POST /issues/:id/return         { reason } → restock + reverse billing (policy window)
```

## Hospitals
```
CRUD /hospitals
CRUD /hospitals/:id/doctors
GET  /hospitals/:id/billing     invoices + outstanding
GET  /hospitals/:id/requests    request history
```

## Camps
```
CRUD /camps
GET  /camps/:id/stats           collection statistics
CRUD /camps/:id/expenses
POST /camps/:id/photos          (multipart)
```

## Staff
```
CRUD /staff
POST /staff/:id/attendance      { checkIn|checkOut }
CRUD /staff/:id/leaves
GET  /staff/:id/activity        from audit log
```

## Billing
```
GET  /invoices  POST /invoices  GET /invoices/:id  PATCH /invoices/:id
POST /invoices/:id/payments     { amountPaise, method, refNo }   (Idempotency-Key)
GET  /invoices/:id/pdf
CRUD /expenses
GET  /billing/daily-cash        ?date
```

## Reports
```
GET  /reports/:type             type∈{daily,weekly,monthly,yearly,inventory,donors,
                                collection,issue,lab,finance,government}
                                ?from&to&branch_id&format=json|pdf|xlsx|csv
                                large exports return { jobId } and notify on completion
```

## Notifications
```
GET  /notifications             in-app feed
POST /notifications/:id/read
PATCH/notifications/preferences  per-channel opt-in
```

## Analytics
```
GET  /analytics/collection-trends      ?range
GET  /analytics/demand-trends          ?range
GET  /analytics/inventory-heatmap
GET  /analytics/top-hospitals
GET  /analytics/top-donors
GET  /analytics/revenue                ?range
```

## Settings
```
GET/PATCH /settings/organization
CRUD /settings/branches
CRUD /settings/users
CRUD /settings/roles            + /roles/:id/permissions
PATCH /settings/integrations    email/sms/whatsapp gateway config (secrets write-only)
POST /settings/backup  POST /settings/restore
```

## AI (deferred, feature-flagged) — `/ai/*`
```
GET  /ai/forecast/low-stock      predicted shortages per group/component
GET  /ai/forecast/demand         ?range
GET  /ai/donor-suggestions       ?bloodGroup  recall candidates
POST /ai/search                  { query } → structured filter (NL search)
POST /ai/report-summary          { reportId } → natural-language summary
```

## Security headers & middleware (all routes)
`helmet`, CORS allowlist, per-IP + per-user rate limit, zod validation, RBAC,
audit middleware, request-id, structured logging.

## Webhooks / realtime
Server-Sent Events (or WS) channel `/events` pushes: new emergency request,
low-stock alert, expiry alert, issue completed — for live dashboard updates.
