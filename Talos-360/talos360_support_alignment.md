# Talos-360 – Platform Alignment

## Integration Modes

* JOB → REST_API → Partially Supported (POST search, no pagination, no dateRangeConfig, status always OPEN)
* APPLICATION → WRITE_REST_API → Supported (POST /api/candidate/external/create)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → Not Configured

---

## Authentication

* BASIC → Supported (customer auth)
* HMAC signature in Authorization header per integration request — non-standard
* Base URL: https://api-careers-sites.talos360.com
* Sync frequency: not specified

---

## Data Model Validation

### JOB

* atsJobId → Present (vacancyId)
* title → Present (title)
* url → Present (url)
* status → Hardcoded "OPEN" — not derived from actual job state
* description → Present (description)
* locations → Present (city, country)
* department → Present (department)
* company → Present (company)
* createdDate → Present (datePosted)
* lastUpdatedDate → Present (datePosted — same as createdDate)
* insertedAt / updatedAt → $now()
* No paginationConfig
* No dateRangeConfig

### APPLICATION

* POST /api/candidate/external/create (single-step)
* Required fields: firstName, lastName, email, atsJobId
* source → hardcoded "Indeed" (not Joveo)
* candidateId → candidate email (not a system ID)
* applicationId → candidate email (not a system ID)
* Resume attachment supported if provided

### APPLICATION_STAGE

* Not configured

---

## Issues

1. APPLICATION WRITE: `source` hardcoded as `"Indeed"` — misattributes applications; must be changed to Joveo
2. APPLICATION WRITE: both `candidateId` and `applicationId` are set to the candidate's email address — not actual system IDs; may cause deduplication issues
3. JOB: no `paginationConfig` — single-response risk for large job catalogs
4. JOB: no `dateRangeConfig` — full catalog re-ingested every sync cycle
5. JOB: `status` hardcoded `"OPEN"` — closed or expired vacancies never updated
6. No APPLICATION_STAGE configured — funnel tracking not available
7. Auth uses HMAC signature construction in `Authorization` header — non-standard; must be correctly computed on every request

---

## Final Status

PARTIAL (application write supported; job ingestion limited; funnel tracking absent; source attribution incorrect)
