# JobVite – Platform Alignment

## Integration Modes

* JOB → WEBHOOK → Supported
* JOB → REST_API → Supported (date-filtered GET)
* APPLICATION → WRITE_REST_API → Supported (2-step: start-chain → create)
* APPLICATION_STAGE → WEBHOOK → Supported
* APPLICATION_STAGE → REST_API enrichment → Supported (candidate lookup for atsJobId)
* CANDIDATE → REST_API → Supported

---

## Authentication

* BASIC → Supported (customer auth)
* API_KEY → Hardcoded in integration headers (SECURITY CONCERN — see Issues)
* Base URL: https://api.jobvite.com/api/v2
* Sync frequency: 1 hour

---

## Data Model Validation

### JOB

* atsJobId → Present (id)
* title → Present
* url → Present (applyUrl)
* status → Present (jobState)
* description → Present
* department → Present (department)
* locations → Present (city, state, country)
* createdDate → Present (date)
* lastUpdatedDate → Present (date)
* insertedAt → mapped to lastUpdatedDate (not $now())
* updatedAt → $now()
* dateRangeConfig → date filter on REST_API

### APPLICATION

* 2-step WRITE chain:
  1. Start-chain (dummy dependency anchor)
  2. POST /application/create (firstName, lastName, email, jobId required)
* applicationId → returned from create response
* candidateId → returned from create response
* Resume attachment supported if provided

### APPLICATION_STAGE

* atsJobId → Enriched via GET candidate (failSilently:true — may be null)
* atsCandidateId → Present (candidate.eId)
* atsApplicationId → Present (id)
* stageId → newValue (raw status value from webhook)
* stageName → newValue (same as stageId — no label mapping)
* appliedAt / createdDate → Present
* lastUpdatedDate → Present (date)
* insertedAt / updatedAt → $now()

### CANDIDATE

* atsCandidateId → Present (eId)
* firstName, lastName → Present
* email → Present
* phone → Present
* workExperience → Present
* education → Present
* No dateRangeConfig — full catalog each sync

---

## Issues

1. **SECURITY**: Hardcoded API keys in integration headers (`x-jvi-sc: b9f9df17efb1b1558ce80ea9b590d4aa`, `x-jvi-api: joveo_partner_api_key`) — must be rotated and moved to customerStaticValues
2. APPLICATION_STAGE: `stageName` equals raw `stageId` (newValue from webhook) — no human-readable label transformation
3. APPLICATION_STAGE: `atsJobId` enriched via GET candidate API with `failSilently:true` — may be `null` in stage records
4. JOB REST_API: `insertedAt` set to `lastUpdatedDate` value (not `$now()`) — non-standard pattern
5. CANDIDATE REST_API: no `dateRangeConfig` — full candidate catalog re-ingested every sync

---

## Final Status

SUPPORTED (with critical security concern: hardcoded API keys must be addressed)
