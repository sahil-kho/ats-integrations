# Greenhouse – Platform Alignment

## Integration Modes

* JOB → WEBHOOK → Supported
* JOB → REST_API → Supported
* APPLICATION → WEBHOOK → Supported
* APPLICATION → REST_API → Supported
* APPLICATION → WRITE_REST_API → Supported
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → WEBHOOK → Supported
* APPLICATION_STAGE → REST_API → Supported
* APPLICATION_STAGE → WRITE_REST_API → Supported
* APPLICATION_STAGE → WRITE_API_RESPONSE_HANDLER → Supported (with bug — see Issues)
* CANDIDATE → WEBHOOK → Supported
* CANDIDATE → REST_API → Supported
* CANDIDATE → WRITE_REST_API → Supported
* CANDIDATE → WRITE_API_RESPONSE_HANDLER → Supported
* ATTACHMENT → WEBHOOK → Supported

---

## Authentication

* BASIC → Supported (Harvest API)
* NO_AUTH with custom Authorization header → Supported (write flows, job board API)
* WEBHOOK → Supported

---

## Data Model Validation

### JOB

* atsJobId → Present (job_post_id via WEBHOOK; id via REST_API)
* title → Present
* description → Present (content)
* url → Present (absolute_url via job board lookup; may be absent if failSilently fires)
* status → Present (open/closed → OPEN/CLOSED)
* department → Present (departments[0].name)
* locations → Present (parsed from location.name with city/state/country split logic)
* remoteType → Present (custom_fields.employment_type)
* createdDate → Present (created_at)
* lastUpdatedDate → Present (updated_at)
* Pagination (REST) → Present (PageConfig, 100/page)
* DateRange → Present (REST_API uses last_activity query params)

### APPLICATION (READ)

* atsApplicationId → Present (id)
* atsJobId → Present (job_post_id; fallback "tempJobId" in WEBHOOK)
* atsCandidateId → Present (candidate.id / candidate_id)
* status → Present
* source → Present
* answers → Present (mapped from answers array)
* createdDate → Present (applied_at)
* lastUpdatedDate → Present (last_activity_at)
* Pagination (REST) → Present (PageConfig, 100/page)

### APPLICATION (WRITE)

* Candidate create → Present (POST /v1/prospects via job board token)
* Candidate update → Present (PATCH /v1/candidates/{id} via harvest token)
* Resume upload → Present (POST /v1/candidates/{id}/attachments)
* Job apply → Present (POST boards-api /jobs/{atsJobId} with full question mapping)
* Question mapping → Present (SELECT/TEXT/RESUME types, multi-value support)
* Required inputs: firstName, lastName, email, atsJobId
* Return values: **ISSUE** — candidateId and applicationId both returned as null

### APPLICATION_STAGE (READ)

* atsApplicationId → Present (id)
* atsJobId → Present (job_post_id)
* atsCandidateId → Present (candidate_id)
* stageId → Present (current_stage.id or derived from status)
* stageName → Present (current_stage.name or status)
* appliedAt → Present (applied_at)
* createdDate, lastUpdatedDate → Present (last_activity_at)
* Pagination → Present (PageConfig, 100/page)
* DateRange → Present (last_activity_after/before)

### APPLICATION_STAGE (WRITE)

* Fetch application → Present (GET /v1/applications/{id})
* Get job stages → Present (GET /v1/jobs/{jobId}/stages)
* Move application → Present (POST /v1/applications/{id}/move with from/to stage IDs)

### CANDIDATE (READ)

* atsCandidateId → Present (id)
* firstName, lastName → Present
* email → Present (email_addresses[0].value)
* phoneNumbers → Present (phone_numbers)
* address → Present (addresses[0])
* workExperience → Present (employments)
* education → Present (educations)
* Pagination → Present (PageConfig, 100/page)

### ATTACHMENT (WEBHOOK)

* atsAttachmentId → Present (id)
* atsCandidateId → Present (candidateId)
* atsApplicationId → Present (applicationId)
* fileName → Present (filename)
* link → Present (url)

---

## Issues

1. APPLICATION WRITE response handler references `pinpoint_update_application_stage` (wrong variable — copy-paste error)
2. APPLICATION WRITE returns null for both candidateId and applicationId — downstream tracking broken
3. Hardcoded `On-Behalf-Of: 4052730008` user ID in all write calls — must be updated per customer
4. APPLICATION WEBHOOK: atsJobId falls back to literal "tempJobId" if jobs array is empty
5. JOB WEBHOOK: url enrichment step uses failSilently:true — url may be silently missing
6. Write flows use NO_AUTH type with manually constructed Authorization header — inconsistent auth pattern

---

## Final Status

SUPPORTED (PARTIAL for application write — candidateId/applicationId not returned)
