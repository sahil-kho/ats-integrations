# Workday – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported (SOAP/XML via GET_JOB_POSTINGS_REQUEST + OAuth token enrichment)
* JOB_QUESTION → REST_API → Supported
* JOB_QUESTION → REST_API_RESPONSE_HANDLER → Supported
* APPLICATION → WRITE_REST_API → Supported (SOAP multi-step chain)
* APPLICATION_STAGE → REST_API → Partial (placeholder expression — not implemented)
* GET_JOB_INFO → REST_API → Supported
* GET_JOB_INFO → REST_API_RESPONSE_HANDLER → Supported

---

## Authentication

* NO_AUTH (customerAuthType) → Credentials passed via customerStaticValues
* WS-Security → Supported (SOAP UsernameToken in XML header — user_name, password, tenant)
* OAuth2 refresh_token → Supported (exchanged inline via /ccx/oauth2/{tenant}/token; Bearer used for REST calls)

---

## Data Model Validation

### JOB

* atsJobId → Present (Job_Requisition_ID)
* title → Present (Job_Posting_Title)
* description → Present (Job_Posting_Description)
* url → Present (External_Apply_URL via REST enrichment step)
* status → Present (hardcoded "OPEN" for active postings; SOAP filter excludes inactive)
* department → Present (Job_Family_Group_Reference ID)
* remoteType → Present (Time_Type_Reference / Position_Time_Type_ID)
* locations → Present (city from Location_ID only — state/country/postalCode null)
* customFields → Present (jobProfileId, jobPostingSiteId)
* Pagination → Present (ExpressionConfig, 300/page via SOAP Response_Filter)
* DateRange → Not present (full catalog fetched each sync)

### APPLICATION (WRITE)

* Multi-step SOAP chain with WS-Security authentication
* Candidate lookup + creation → Present
* Resume/attachment upload → Present
* Application submission → Present
* Job question answer mapping → Present (via JOB_QUESTION response handler)

### APPLICATION_STAGE (READ)

* dataModelJsonataExpression → **NOT IMPLEMENTED** (value is `${transformStageJsonataExpression}` placeholder)
* RaaS report integration defined but mapping expression not populated
* stageId → Missing (placeholder not resolved)
* stageName → Missing (placeholder not resolved)
* atsJobId, atsCandidateId, atsApplicationId → Unknown (depend on placeholder expression)

### JOB_QUESTION

* Questionnaire fetch → Present (REST API)
* Question type mapping → Present (SELECT, TEXT, RESUME, etc.)
* Response handler transformation → Present (maps to ucpQuestions, additionalQuestions, surveyQuestions, consentQuestions)

### GET_JOB_INFO

* Single job lookup via SOAP by Job_Requisition_ID → Present
* Returns: atsJobId, title, description, url, department, status, remoteType, location (city only)
* Error handling → Present (404 for invalid job ID)

### CANDIDATE (READ)

* No READ integration defined → **NOT SUPPORTED**

---

## Issues

1. APPLICATION_STAGE `dataModelJsonataExpression` is the literal string `${transformStageJsonataExpression}` — funnel tracking produces no output until this is implemented
2. No `dateRangeConfig` on JOB ingestion — entire job catalog is fetched via SOAP every sync (expensive for large tenants)
3. Candidate ingestion not available — no REST_API or WEBHOOK integration for CANDIDATE data model
4. Dual auth model (WS-Security + OAuth2) adds operational complexity; password embedded in SOAP XML body
5. Location mapping resolves city only from Location_ID — state, country, postalCode are null
6. `customerAuthType: NO_AUTH` — all credentials must be stored in customerStaticValues (user_name, password, tenant, clientId, clientSecret, refreshToken)

---

## Final Status

SUPPORTED (PARTIAL for funnel tracking — APPLICATION_STAGE expression is a placeholder; NOT SUPPORTED for candidate ingestion)
