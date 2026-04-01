# Eyrecruit – Platform Alignment

## Integration Modes

* JOB → REST_API → Partially Supported (no pagination, no delta sync, limited field mapping)
* APPLICATION → Not Configured
* APPLICATION_STAGE → Not Configured

---

## Authentication

* API_KEY → Supported (Authorization header)
* Base URL: http://recruitment.eylog.co.uk (HTTP — insecure)
* Sync frequency: 6 hours

---

## Data Model Validation

### JOB

* atsJobId → Present (job_id)
* refNumber → Present (job_id — same value as atsJobId)
* title → Present (job_title)
* url → Present (url)
* description → Present (description)
* status → Hardcoded "OPEN" — not derived from actual job state
* locations → Present (city, state, country from location_city / location_county)
* createdDate → Present (published_at)
* lastUpdatedDate → Not mapped — no last-modified timestamp available
* insertedAt / updatedAt → $now()
* No paginationConfig
* No dateRangeConfig

### APPLICATION

* Not configured

### APPLICATION_STAGE

* Not configured

---

## Issues

1. JOB only — no application write or funnel tracking of any kind
2. JOB: no `paginationConfig` — truncation risk if catalog exceeds single-page response
3. JOB: no `dateRangeConfig` — full catalog re-ingested on every 6-hour sync
4. JOB: `lastUpdatedDate` not mapped — only `createdDate` available; delta tracking not possible
5. JOB: `status` hardcoded `"OPEN"` — no mechanism to mark jobs as CLOSED
6. Base URL uses HTTP not HTTPS — insecure transport; credentials and data transmitted in plaintext
7. JOB: `refNumber` mapped from `job_id` (same as `atsJobId`) — no meaningful ref distinction

---

## Final Status

NOT_SUPPORTED (job ingestion only; application write and funnel tracking absent; insecure base URL)
