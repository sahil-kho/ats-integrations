# Bullhorn – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported
* APPLICATION → REST_API → Supported
* APPLICATION → WRITE_REST_API → Supported
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → REST_API → Supported
* APPLICATION_STAGE → WRITE_REST_API → Supported
* APPLICATION_STAGE → WRITE_API_RESPONSE_HANDLER → Supported
* CANDIDATE → REST_API → Supported

---

## Authentication

* BULLHORN_OAUTH2 → Supported (custom refresh_token OAuth2 with BhRestToken pattern)

---

## Data Model Validation

### JOB

* atsJobId → Present (id)
* title → Present
* description → Present (publicDescription)
* url → Present (jobPostingURL) — **ISSUE**: fallback is literal "string" not null
* status → Present (isOpen mapped to OPEN/CLOSED)
* locations → Present (address with city/state/countryName/countryCode/zip)
* company → Present (clientCorporation.name)
* remoteType → Present (isWorkFromHome → REMOTE/ONSITE)
* startDate → Present ($fromMillis(startDate))
* createdDate → Present ($fromMillis(dateAdded))
* lastUpdatedDate → Present ($fromMillis(dateLastModified))
* updatedAt → **ISSUE**: mapped to $now() instead of dateLastModified
* Pagination → Present (ExpressionConfig, 200/page, start offset)
* DateRange → Present (Lucene dateLastModified range query)

### APPLICATION (READ)

* atsApplicationId → Present (id)
* atsJobId → Present (jobOrder.id)
* atsCandidateId → Present (candidate.id)
* status → Present
* source → Present (falls back to "NOT_PROVIDED")
* createdDate → Present ($fromMillis(dateAdded))
* lastUpdatedDate → Present ($fromMillis(dateLastModified))
* Pagination → Present (ExpressionConfig, 200/page)
* DateRange → Present (Lucene dateLastModified range)

### APPLICATION (WRITE)

* Check existing candidate → Present (GET /search/Candidate by email)
* Create candidate → Present (PUT /entity/Candidate)
* Upload resume → Present (PUT /file/Candidate/{id})
* Check existing application → Present (GET /search/JobSubmission by candidate+job)
* Create application (path 1 — new candidate) → Present (PUT /entity/JobSubmission via bullhorn-create-application-one)
* Create application (path 2 — existing candidate) → Present (PUT /entity/JobSubmission via bullhorn-create-application-two)
* Required inputs: email, firstName, lastName, atsJobId

### APPLICATION_STAGE (READ)

* atsApplicationId → Present (id from JobSubmission)
* atsJobId → Present (jobOrder.id)
* atsCandidateId → Present (candidate.id)
* stageId → Present (status string) — **NOTE**: not a numeric ID
* stageName → Present (status string)
* appliedAt → Present ($fromMillis(dateAdded))
* createdDate, lastUpdatedDate → Present ($fromMillis)
* Pagination → Present (ExpressionConfig, 200/page)
* DateRange → Present (Lucene dateLastModified range)

### APPLICATION_STAGE (WRITE)

* Stage update → Present (POST /entity/JobSubmission/{applicationId} with {status: newStatus})
* Required inputs: applicationId, newStatus (or applicationStage)

### CANDIDATE (READ)

* atsCandidateId → Present (id)
* firstName, lastName → Present
* email → Present
* phoneNumbers → Present (mobile, phone, phone2, phone3, workPhone)
* address → Present (city, state, countryName, zip)
* Pagination → Present (ExpressionConfig, 200/page)
* DateRange → Present (Lucene dateLastModified range)

---

## Security Issues (Action Required)

1. **CRITICAL**: BhRestToken `28645_8204920_aeff4e2f-4d0b-4c2f-b321-83a61abe7cb9` hardcoded in response handlers — rotate immediately and store as customer static value
2. **CRITICAL**: Hardcoded restUrl `https://rest44.bullhornstaffing.com/rest-services/dkixnd/` in response handlers — remove and use dynamic resolution
3. **CRITICAL**: `bullhorn_customer_integration_config.json` contains plaintext `password` and `client_secret` in `additionalParams` — rotate credentials and use secure secret management

---

## Data Quality Issues

1. JOB url falls back to literal string `"string"` when jobPostingURL is null or empty
2. JOB updatedAt uses $now() — downstream systems cannot detect unchanged jobs
3. APPLICATION_STAGE stageId equals stageName (status string) — may cause incorrect stage deduplication

---

## Final Status

SUPPORTED (CRITICAL: hardcoded credentials must be rotated before production use)
