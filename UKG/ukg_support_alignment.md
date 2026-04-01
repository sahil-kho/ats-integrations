# UKG – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported (dateRangeConfig ✓, GET /api/opportunities)
* APPLICATION → WRITE_REST_API → Supported (6-step chain: lookup → create candidate → create application × 2 → upload resume × 2)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → REST_API → Supported (4-step enrichment chain)

---

## Authentication

* OAUTH_2 → Supported (client_credentials grant)
* Base URL: https://service2.ultipro.com/talent/recruiting/v2/SAL1014SAMN
* Sync frequency: 1 hour

---

## Data Model Validation

### JOB

* atsJobId → Present (RequisitionId)
* refNumber → Present (PositionId)
* title → Present (Title)
* url → Present (ExternalApplyUrl)
* status → Present (mapped from Status field)
* description → Present (Description)
* department → Present (DepartmentName)
* locations → Present (city, state, country, postalCode)
* company → Present (CompanyName)
* createdDate → Present (PostedDate)
* lastUpdatedDate → Present (UpdatedDate)
* dateRangeConfig → UpdatedDate (delta sync supported ✓)
* No paginationConfig — truncation risk for large catalogs

### APPLICATION

* 6-step WRITE chain:
  1. Start-chain (dummy anchor)
  2. Check if candidate already exists (GET by email)
  3. Create candidate if not exists (POST /api/applicants)
  4. Create application path 1 (POST /api/applications — for specific form scenarios)
  5. Create application path 2 (POST /api/applications — for alternate form scenarios)
  6. Upload resume (PATCH /api/applications/{id}/attachments)
* Requires: firstName, lastName, email, atsJobId, availableStartDate (in customFields)
* Hardcoded `applicantSource` UUID: `ee9e2991-ee55-42a3-add6-6dad8110810a`
* candidateId → returned from create steps
* applicationId → returned from create steps

### APPLICATION_STAGE

* 4-step enrichment chain per application:
  1. GET /api/applications — list applications with date filter
  2. GET /api/recruiting-processes — fetch process metadata per application
  3. GET /api/job-refnumber — resolve job reference from applicationId
  4. GET /api/candidate-email — resolve candidate email from candidateId
  5. Final mapping: stageId from process step_id, stageName from process step label
* dateRangeConfig → lastUpdatedDate
* stageId → internal step_id (numeric)
* stageName → step label from process

---

## Issues

1. APPLICATION_STAGE READ: 4-step sequential API chain per application — N+4 calls at scale; significant performance concern
2. APPLICATION WRITE: `applicantSource` UUID `ee9e2991-ee55-42a3-add6-6dad8110810a` is hardcoded — tenant-specific; must be updated per customer
3. APPLICATION WRITE: `availableStartDate` in `customFields` is a required field — submission blocked if not provided by the apply flow
4. WRITE_API_RESPONSE_HANDLER: contains hardcoded tenant-specific URL `https://service2.ultipro.com/talent/recruiting/v2/SAL1014SAMN/api/applications` — not portable across UKG customers
5. JOB: no `paginationConfig` — truncation risk if job catalog exceeds single-response limit
6. APPLICATION WRITE: two parallel application creation paths (`ukg-create-applications-one`, `ukg-create-applications-two`) — selection logic based on form structure; may create duplicate applications if both conditions are met

---

## Final Status

SUPPORTED (with performance concerns on APPLICATION_STAGE and hardcoded tenant values that must be updated per customer)
