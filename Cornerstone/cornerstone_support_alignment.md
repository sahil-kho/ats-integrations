# Cornerstone – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported (PageConfig pagination, lastModifiedSince filter ✓)
* JOB → WEBHOOK → Supported
* APPLICATION → WRITE_REST_API → Supported (dependency chain)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → REST_API → Supported (PageConfig pagination, status-list filter)
* APPLICATION_STAGE → WRITE_REST_API → Supported (PUT status update)
* CANDIDATE → REST_API → Supported (status-list filter)
* JOB_QUESTION → REST_API → Supported
* JOB_QUESTION → REST_API_RESPONSE_HANDLER → Supported

---

## Authentication

* OAUTH_2 → Supported (client_credentials grant)
* Base URL: https://pservjoveo.csod.com
* Sync frequency: 15 minutes

---

## Data Model Validation

### JOB

* atsJobId → Present (requisitionId)
* title → Present
* status → Mapped (isOpen field)
* department → Present
* locations → Present (city, state, country)
* createdDate → Present
* lastUpdatedDate → lastModifiedDate
* dateRangeConfig → lastModifiedSince (delta sync supported)
* paginationConfig → PageConfig

### APPLICATION

* Multi-step WRITE chain:
  1. Start-chain (dummy)
  2. Create application (POST /recruiting/v1/applications)
* Source label hardcoded as "cornerstoneId"
* WRITE_API_RESPONSE_HANDLER: returns null candidateId and applicationId on failure

### APPLICATION_STAGE

* atsApplicationId → Present
* atsCandidateId → Present
* stageId / stageName → applicationStatus
* createdDate / lastUpdatedDate → date parsing via MM/DD/YYYY split with $now() fallback on unexpected format
* paginationConfig → PageConfig
* No dateRangeConfig — filters by hardcoded status list instead of date range

### CANDIDATE

* atsCandidateId → Present
* firstName, lastName, email → Present
* No dateRangeConfig — filters by hardcoded status list

### JOB_QUESTION

* Questions and answer options mapped from job profile
* REST_API_RESPONSE_HANDLER aggregates question structure

---

## Issues

1. APPLICATION_STAGE READ: date parsing uses MM/DD/YYYY `$split` logic — falls back to `$now()` when format is unexpected, causing inaccurate timestamps
2. WRITE_API_RESPONSE_HANDLER: candidateId and applicationId are both `null` if the success body check fails — may result in untracked applications
3. APPLICATION, APPLICATION_STAGE, CANDIDATE READ: no `dateRangeConfig` — relies on hardcoded status list filter; all matching records re-ingested every sync cycle
4. APPLICATION WRITE: source hardcoded as `"cornerstoneId"` — not customer-configurable
5. JOB WEBHOOK: triggers a full job fetch without a delta/date filter — may re-process unchanged records

---

## Final Status

SUPPORTED (minor issues with date fallbacks and missing delta sync on application/stage reads)
