# Eploy – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported
* APPLICATION → REST_API → Supported
* APPLICATION → WRITE_REST_API → Supported
* APPLICATION_STAGE → REST_API → Supported
* APPLICATION_STAGE → WRITE_REST_API → Supported
* APPLICATION_STAGE → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* CANDIDATE → REST_API → Supported

---

## Authentication

* OAUTH_2 → Supported

---

## Data Model Validation

### JOB

* atsJobId → Present (VacancyId)
* title → Present (Title)
* url → Present (WebsiteURLs[0]) — may be null for some vacancies
* status → Present (VacancyStatus.Description mapped to OPEN/CLOSED)
* locations → Present (Address object with city/state/country/postalCode)
* company → Present (falls back to "Eploy" literal if company null)
* department → Present (Position.Description)
* remoteType → Present (VacancyType.Description)
* createdDate → Present (CreationDate)
* lastUpdatedDate → Present (ModificationDate)
* updatedAt → **ISSUE**: mapped to $now() not ModificationDate
* isDeleted → Present (derived from VacancyStatus)
* Pagination → **ISSUE**: RecordsPerPage hardcoded as 500 in request body; no ExpressionConfig paginationConfig
* Delta sync → **ISSUE**: No dateRangeConfig on job fetch — full catalog on every run

### APPLICATION (READ)

* atsApplicationId → Present (ApplicationId)
* atsJobId → Present (Vacancy.Id)
* atsCandidateId → Present (Candidate.Id)
* status → Present (Workflow.Stage.Description + Status.Description)
* source → Present
* createdDate → Present (ApplicationDate)
* lastUpdatedDate → Present (ModificationDate)
* Pagination → Present (ExpressionConfig, 200/page)
* DateRange → Present (ModificationDate filter)

### APPLICATION (WRITE)

* Candidate check → Present (search by email)
* Candidate create → Present (POST /api/candidates)
* Candidate update → Present (PATCH /api/candidates/{id} for existing)
* Resume upload → Present (POST /api/candidates/{id}/upload/cv, Base64ToFile)
* Application create → Present (POST /api/applications, two paths: new/existing candidate)
* Required inputs: email, firstName, lastName, atsJobId

### APPLICATION_STAGE (READ)

* atsJobId → Present (Vacancy.Id via fanout)
* atsCandidateId → Present (Candidate.Id via fanout)
* atsApplicationId → Present (ApplicationId via fanout)
* stageId → Present (Workflow.Stage.Id + optional Status.Id)
* stageName → Present (Workflow.Stage.Description + optional Status.Description)
* appliedAt → Present (ApplicationDate)
* createdDate → Present (ModificationDate)
* Pagination → Present (ExpressionConfig, 10/page)

### APPLICATION_STAGE (WRITE)

* Stage lookup → Present (options → stage types → vacancy statuses)
* Stage update → Present (PATCH /api/applications/{id} with Workflow body)
* Supports stage+status combination in name format "Stage - Status"

### CANDIDATE (READ)

* atsCandidateId → Present (CandidateId)
* firstName, lastName, email → Present
* phoneNumbers → Present (Mobile, Telephone, WorkTelephone)
* address → Present (Address object)
* Pagination → Present (ExpressionConfig, 200/page)
* DateRange → Present (ModificationDate filter)

---

## Issues

1. JOB pagination relies on fixed RecordsPerPage:500 body value — catalogs exceeding 500 records silently truncate
2. JOB lacks dateRangeConfig — full catalog is fetched on every sync (no incremental sync)
3. JOB updatedAt uses $now() instead of ModificationDate — breaks downstream delta logic
4. APPLICATION_STAGE read performs one GET per candidate (N+1) — performance degrades at scale
5. APPLICATION_STAGE write uses a 4-step lookup chain — added latency and more failure surfaces

---

## Final Status

SUPPORTED
