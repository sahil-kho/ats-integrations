# Pinpoint – Platform Alignment

## Integration Modes

* JOB → REST_API → NOT SUPPORTED (dummy integration — executionEligibilityRules: false)
* APPLICATION → WRITE_REST_API → Supported (POST /api/v1/applications)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → Not Configured

---

## Authentication

* API_KEY → Supported (Authorization header)
* Base URL: https://joveo-sandbox.pinpointhq.com (sandbox — may need production URL)
* Sync frequency: not specified

---

## Data Model Validation

### JOB

* Not supported — `executionEligibilityRules` set to `false` on all JOB integrations
* Job dummy integration exists only as a dependency anchor, never executed

### APPLICATION

* POST /api/v1/applications with candidate + job details
* Required fields: firstName, lastName, email, atsJobId, attachmentContent (resume)
* candidateId → null (not returned from Pinpoint create endpoint)
* applicationId → returned from create response
* Requires resume in base64 format — submission blocked if absent

### APPLICATION_STAGE

* Not configured

---

## Issues

1. JOB ingestion not supported — `executionEligibilityRules: false` on all JOB entries; never executed
2. No APPLICATION_STAGE configured — funnel tracking not available
3. APPLICATION WRITE: `candidateId` is always `null` — not returned by Pinpoint create endpoint
4. APPLICATION WRITE: resume (`attachmentContent`) is required — submission fails without it
5. Base URL points to sandbox environment (`joveo-sandbox.pinpointhq.com`) — must be updated to production URL before go-live
6. No deduplication or conflict handling — no error response for duplicate application submissions

---

## Final Status

NOT_SUPPORTED (application write only; job ingestion disabled; funnel tracking absent; sandbox URL)
