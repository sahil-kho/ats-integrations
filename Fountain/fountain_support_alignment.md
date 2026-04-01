# Fountain – Platform Alignment

## Integration Modes

* JOB → REST_API → Partially Supported (PageConfig pagination, no dateRangeConfig)
* APPLICATION → WRITE_REST_API → Supported (multi-step: applicant create → S3 presigned URL → S3 upload)
* APPLICATION → WRITE_API_RESPONSE_HANDLER → Supported
* APPLICATION_STAGE → WEBHOOK → Supported (full field mapping)

---

## Authentication

* WebhookCustomerIntegration (no central customerBaseUrl or customerAuthInfo)
* Per-integration API key passed via request headers
* customerBaseUrl: null

---

## Data Model Validation

### JOB

* atsJobId → Present (id)
* title → Present (title)
* url → Present (apply_url)
* status → Present (state mapped to OPEN/CLOSED)
* locations → Present (city, state, country)
* department → Present (brand_name)
* createdDate → created_at
* lastUpdatedDate → created_at (not actual last-modified)
* insertedAt / updatedAt → $now()
* paginationConfig → PageConfig (1000 records per page)
* No dateRangeConfig — full catalog each sync

### APPLICATION

* 4-step WRITE chain:
  1. POST applicant to /v2/applicants (email, phone, name, job_id required)
  2. Fetch presigned S3 URL for resume upload
  3. PUT resume file to S3
  4. Confirm attachment
* candidateId → null (not returned from create endpoint)
* applicationId → returned applicant id
* Phone number required — submission blocked if unavailable

### APPLICATION_STAGE

* atsJobId → Present (job_id)
* atsCandidateId → Present (applicant_id)
* atsApplicationId → Present (id)
* stageId → Present (stage)
* stageName → Present (stage)
* appliedAt → Present (applied_at)
* createdDate → Present (created_at)
* lastUpdatedDate → Present (updated_at)
* insertedAt / updatedAt → $now()

---

## Issues

1. APPLICATION WRITE: `candidateId` not returned from creation endpoint — always `null` in response handler
2. JOB: no `dateRangeConfig` — full catalog re-fetched every sync cycle
3. JOB: `lastUpdatedDate` mapped from `created_at` — not an actual last-modified timestamp
4. APPLICATION WRITE: phone number is a required field for submission — flow breaks if candidate has no phone
5. APPLICATION WRITE: 4-step S3 upload chain — mid-chain failure (e.g., S3 presign timeout) may leave applicant created but without resume
6. Auth is distributed — each integration carries its own API key header; no central auth management

---

## Final Status

PARTIAL (application write and funnel tracking supported; job ingestion lacks delta sync; candidateId not available)
