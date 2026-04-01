# iCIMS – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported (4-step enrichment chain: search → job details → location → portal profile)
* APPLICATION → WRITE_REST_API → Supported (create candidate + submit application)
* CANDIDATE → WRITE_REST_API → Supported (create → update → resume upload)
* CANDIDATE → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → Not Configured

---

## Authentication

* OAUTH_2 → Supported for reads (client_credentials grant)
* BASIC → Supported for writes
* Base URL: https://api.icims.com
* Sync frequency: 1 hour

---

## Data Model Validation

### JOB

* atsJobId → Present (id)
* title → Present (jobtitle.value)
* url → Present (jdurl)
* status → Hardcoded "OPEN"
* department → Present (folder.value)
* locations → Enriched via location profile API (failSilently:true — may be null)
* company → Enriched via portal profile API
* createdDate → Present (createdon)
* lastUpdatedDate → Present (updatedon)
* No dateRangeConfig — full catalog each sync

### APPLICATION

* 2-step WRITE chain:
  1. POST to /people (candidate create) — requires pre-populated candidateId in customFields
  2. POST to /applications (application submission)
* applicationId → returned from submission
* candidateId → from customFields.candidateId (must be pre-populated)

### CANDIDATE

* Multi-step WRITE chain:
  1. Create candidate profile
  2. Update with additional fields
  3. Upload resume
* Uses client-specific field IDs in JSONata expressions (field38755, field38808)

### APPLICATION_STAGE

* Not configured

---

## Issues

1. No APPLICATION_STAGE configured — funnel tracking completely absent
2. JOB: no `dateRangeConfig` — full catalog re-ingested on every 1-hour sync
3. JOB: `status` hardcoded `"OPEN"` — no support for CLOSED jobs
4. JOB: location enrichment step uses `failSilently:true` — locations may be absent in job records
5. APPLICATION WRITE: requires `customFields.candidateId` to be pre-populated — tightly couples APPLICATION write to external candidate state
6. CANDIDATE WRITE: JSONata references client-specific field IDs (e.g., `field38755`) — not portable across iCIMS customers
7. Split auth (OAUTH_2 for reads, BASIC for writes) — requires managing two separate credential sets

---

## Final Status

PARTIAL (job and application write supported; funnel tracking absent; auth complexity and client-specific field mappings)
