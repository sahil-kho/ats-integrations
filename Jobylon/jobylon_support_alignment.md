# Jobylon – Platform Alignment

## Integration Modes

* JOB → REST_API → Partially Supported (no pagination, no dateRangeConfig, failSilently:true)
* APPLICATION → WRITE_REST_API → Supported (POST to /p1/applications/)
* APPLICATION → WEBHOOK → Supported
* APPLICATION_STAGE → WEBHOOK → Partially Supported (timestamps hardcoded to $now())
* JOB_QUESTION → REST_API → Supported (per-job fetch chain)
* JOB_QUESTION → REST_API_RESPONSE_HANDLER → Supported

---

## Authentication

* NO_AUTH on customer config
* Per-integration static headers: X-App-Key, X-App-Id
* Base URL: https://feed.jobylon.com
* Sync frequency: not specified (continuous webhook + scheduled REST)

---

## Data Model Validation

### JOB

* atsJobId → Present (id)
* title → Present (title)
* url → Present (urls.ad_url)
* status → Present (status)
* description → Present (description)
* locations → Present (city, country)
* department → Present (function)
* company → Present (company.name)
* createdDate → Present (created)
* lastUpdatedDate → Present (modified)
* insertedAt / updatedAt → $now()
* failSilently:true — job fetch errors silently ignored
* No pagination, no dateRangeConfig

### APPLICATION

* POST /p1/applications/ (firstName, lastName, email, jobId, attachmentLocalPath)
* applicationId → returned from create response
* candidateId → returned from create response
* Resume provided as URL (attachmentLocalPath) — must be externally hosted

### APPLICATION (WEBHOOK)

* Full application payload mapping from webhook
* atsJobId, atsCandidateId, atsApplicationId → Present
* createdDate, lastUpdatedDate → Present from webhook payload

### APPLICATION_STAGE

* atsJobId → Present (from webhook)
* atsCandidateId → Present
* atsApplicationId → Present
* stageId / stageName → Present (status from webhook)
* appliedAt → $now() (real timestamp not captured)
* createdDate → $now() (real timestamp not captured)
* lastUpdatedDate → $now() (real timestamp not captured)

### JOB_QUESTION

* Per-job fetch: /jobs/{jobId}/questions → /jobs/{jobId}/questions/{formId}/fields
* Question types, options, and required flags mapped
* N+1 API call per job — performance concern at scale

---

## Issues

1. APPLICATION_STAGE WEBHOOK: `appliedAt`, `createdDate`, and `lastUpdatedDate` all hardcoded to `$now()` — actual event timestamps from webhook payload are discarded
2. JOB: `failSilently:true` on main job fetch — integration failures are silently swallowed with no alerting
3. JOB: no `paginationConfig` and no `dateRangeConfig` — full catalog re-ingested each sync
4. APPLICATION WRITE: resume provided as `attachmentLocalPath` (a URL) — requires file to already be hosted externally; base64 not supported
5. Auth: NO_AUTH at customer level; per-integration `X-App-Key`/`X-App-Id` static headers provide minimal security — no centralized auth management
6. JOB_QUESTION: form field lookup is per-job (N+1 pattern) — performance concern for large job catalogs

---

## Final Status

PARTIAL (application write and job ingestion functional; funnel timestamps inaccurate; no centralized auth)
