# Ashby – Platform Alignment

## Integration Modes

* JOB → WEBHOOK → Partially Supported (no REST_API poll; depends on webhook delivery)
* JOB → REST_API enrichment → Partial (secondary call to /jobPosting.info; failSilently:true)
* CANDIDATE → REST_API → Supported (cursor-based pagination)
* CANDIDATE → WRITE_REST_API → Supported (check → create → add resume chain)
* APPLICATION → WRITE_REST_API → Supported (2-step: start-chain → create-application)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → Not Configured

---

## Authentication

* BASIC → Supported (Base64-encoded API key via Authorization header)
* Base URL: https://api.ashbyhq.com

---

## Data Model Validation

### JOB

* atsJobId → Present (from webhook jobId)
* title → Present
* url → Present (externalLink)
* description → Enriched via /jobPosting.info (failSilently:true — may be null)
* locations → Enriched via /jobPosting.info (failSilently:true — may be null)
* department → Enriched via /jobPosting.info (failSilently:true — may be null)
* status → Derived from `$.success` in enrichment response
* createdDate → publishedDate (from webhook)
* lastUpdatedDate → updatedAt (from webhook)
* updatedAt → $now()

### CANDIDATE

* atsCandidateId → Present ($.id)
* firstName → Derived: $split(name, " ")[0] — edge case for middle names
* lastName → Derived: $substringAfter(name, " ") — empty string for single-token names
* email → primaryEmailAddress.value with fallback to emailAddresses[0].value
* phoneNumbers → Mapped from phoneNumbers or primaryPhoneNumber
* links → socialLinks + profileUrl
* workExperience → position + company fields
* education → school field
* createdDate → $fromMillis($toMillis(createdAt))
* lastUpdatedDate → $fromMillis($toMillis(updatedAt))
* insertedAt / updatedAt → $now()
* No dateRangeConfig — full catalog each sync

### APPLICATION

* Candidate-first flow: creates/finds candidate, then creates application
* Requires: atsJobId, atsCandidateId (from prior CANDIDATE WRITE chain)
* Resume upload: optional (failSilently:true on ashby-add-resume)
* applicationId → applicationId from create response
* candidateId → candidateId from create response

### APPLICATION_STAGE

* Not configured

---

## Issues

1. JOB read is webhook-only — no REST_API polling; jobs not synced until webhook fires
2. JOB enrichment via /jobPosting.info is failSilently:true — description, locations, department may be missing if enrichment call fails
3. CANDIDATE list: firstName/lastName split on first space — fails for middle names or single-token names
4. CANDIDATE list: no dateRangeConfig — full catalog re-fetched every sync cycle
5. APPLICATION WRITE: depends on candidateId produced by prior CANDIDATE WRITE chain — breaks if candidate write is skipped
6. No APPLICATION_STAGE ingestion configured — funnel tracking not available

---

## Final Status

PARTIAL (job webhook-only, no funnel tracking)
