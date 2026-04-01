# Dayforce – Platform Alignment

## Integration Modes

* JOB → REST_API → Partially Supported (no pagination, no delta sync, status always OPEN)
* APPLICATION → WRITE_REST_API → Supported (questionnaire lookup → create application)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → Not Configured

---

## Authentication

* OAUTH_2 → Supported (password grant type)
* Base URL: https://www.dayforcehcm.com/api/tlc/V1
* Sync frequency: 10 hours

---

## Data Model Validation

### JOB

* atsJobId → Present (JobFeedId)
* title → Present (Title)
* url → Present (ApplyUri)
* description → Present (Description)
* status → Hardcoded "OPEN" — not derived from actual job state
* locations → Present (city, state, country from location array)
* department → Present (Department)
* company → Present (Company)
* createdDate → Present (PostedDate)
* lastUpdatedDate → $fromMillis($toMillis(LastUpdated)) — format-dependent
* No paginationConfig — full result set in single response
* No dateRangeConfig — full catalog every sync

### APPLICATION

* 2-step chain:
  1. Fetch questionnaire (GET /CandidateSourcing/Forms/{questionSetId}) — requires customFields.questionSetId
  2. POST to /CandidateSourcing with candidate + questionnaire answers
* candidateId → returned Identifier field (same as applicationId)
* applicationId → returned Identifier field
* Resume attachment supported if provided

### APPLICATION_STAGE

* Not configured

---

## Issues

1. JOB: no `paginationConfig` — single response; truncation risk for large catalogs
2. JOB: no `dateRangeConfig` — full catalog re-ingested on every sync
3. JOB: `status` hardcoded `"OPEN"` — closed or expired jobs are never tracked as CLOSED
4. APPLICATION WRITE: `candidateId` and `applicationId` both map to the same `Identifier` field — not distinct IDs
5. APPLICATION WRITE: requires `customFields.questionSetId` to fetch questionnaire — fails silently if not supplied
6. OAuth2 uses `password` grant type — credentials (username + password) included in token request; less secure than client_credentials
7. No APPLICATION_STAGE configured — funnel tracking not available

---

## Final Status

PARTIAL (application write supported; job ingestion limited; no funnel tracking)
