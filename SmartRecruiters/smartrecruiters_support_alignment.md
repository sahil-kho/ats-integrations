# SmartRecruiters – Platform Alignment

## Integration Modes

* APPLICATION → WRITE_REST_API → Supported (get posting → create application)
* APPLICATION_STAGE → WEBHOOK → Supported
* APPLICATION_STAGE → REST_API enrichment → Supported (3-step: get job → get application → get status history)
* JOB_QUESTION → REST_API_RESPONSE_HANDLER → Supported (full question/consent/UCP mapping)
* JOB → REST_API → Partially Supported (not fully confirmed from available config)

---

## Authentication

* OAUTH_2 → Supported (client_credentials grant)
* API_KEY → Supported (fallback / combined auth on WRITE integrations)
* Base URL: https://api.smartrecruiters.com
* Sync frequency: 1 hour

---

## Data Model Validation

### APPLICATION

* 3-step WRITE chain:
  1. Start-chain (dummy anchor for dependency fields)
  2. GET /v1/companies/{companyIdentity}/postings/{jobAdId} — fetch posting UUID
  3. POST /postings/{postingId}/candidates — create application with full payload
* Candidate fields: firstName, lastName, email, phone, location, resume, work experience, education
* Screening questions, consent decisions, and messageToHiringManager included in payload
* Source mapping: CONVERSATION_APPLY_WEB, CONVERSATION_APPLY_WHATSAPP, CONVERSATION_APPLY_SMS → specific sourceId UUIDs (hardcoded)
* candidateId → returned from create response
* applicationId → returned from create response

### APPLICATION_STAGE

* 3-step enrichment chain after webhook:
  1. GET /jobs/{jobId} — fetch job refNumber
  2. GET /job-applications-api/v202112/job-applications/{jobApplicationId} — fetch application status
  3. GET /candidates/{candidateId}/jobs/{jobId}/status/history — fetch full status history
* atsJobId → refNumber from job lookup
* atsCandidateId → candidateId from webhook
* atsApplicationId → jobApplicationId from webhook
* stageId → concat(subStatus + "_" + createDate) — composite key
* stageName → status field
* appliedAt → timestamp of first "NEW" status from history
* insertedAt / updatedAt → $now()

### JOB_QUESTION

* GET /postings/{postingId}/configuration — fetches full question schema
* Supports: QUESTION, QUESTION_SET, consent questions, UCP questions
* Input types mapped: INPUT_TEXT, SINGLE_SELECT, MULTI_SELECT, CHECKBOX, TEXTAREA → Joveo types
* Conditional dependencies (show/hide) mapped from `conditionals` settings
* Large inline `$eval()` JSONata — performance and maintenance risk

---

## Issues

1. APPLICATION WRITE: fails if posting UUID is not found from `sr-get-posting` step — gated on `response.uuid`; no fallback
2. APPLICATION_STAGE: `stageId` = `status + "_" + createDate` — composite key may cause deduplication issues when two events share the same date
3. APPLICATION WRITE: hardcoded `sourceTypeId: "ORGANIC"`, `sourceSubTypeId: "SMARTRECRUITERS"`, and fallback `sourceId` UUID — not customer-configurable
4. JOB_QUESTION: very large inline `$eval()` JSONata string containing hardcoded phone country list — maintenance burden, performance risk
5. APPLICATION_STAGE: uses versioned API path `/job-applications-api/v202112/` — must be updated when SR deprecates this version
6. Dual auth (OAUTH_2 + API_KEY) on APPLICATION WRITE — authentication complexity; both must be maintained

---

## Final Status

SUPPORTED (application write and funnel tracking functional; minor issues with hardcoded values and versioned API paths)
