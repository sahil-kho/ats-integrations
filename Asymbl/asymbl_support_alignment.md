# Asymbl – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported (SOQL query, ExpressionConfig cursor pagination, dateRangeConfig ✓)
* APPLICATION → WRITE_REST_API → Supported (contact check → create contact → create applicant → upload resume)
* APPLICATION_STAGE → REST_API → Supported (SOQL query, ExpressionConfig pagination, dateRangeConfig ✓)
* APPLICATION_STAGE → WRITE_REST_API → Supported (PATCH status update)

---

## Authentication

* OAUTH_2 → Supported (client_credentials grant)
* Base URL: https://asymbl-joveo-tso.my.salesforce.com (Salesforce tenant — hardcoded)

---

## Data Model Validation

### JOB

* atsJobId → Present (Id)
* title → Present (Name)s
* status → Mapped (bpats__Job_Status__c)
* department → Present (bpats__Department__r.Name)
* locations → Present (city, state, country fields)
* createdDate → CreatedDate
* lastUpdatedDate → LastModifiedDate
* insertedAt / updatedAt → $now() (not actual timestamps)
* dateRangeConfig → LastModifiedDate (delta sync supported)
* paginationConfig → ExpressionConfig (cursor-based)

### APPLICATION

* Multi-step WRITE chain:
  1. Contact lookup by email
  2. Contact create if not exists
  3. Applicant record create (bpats__Applicant__c)
  4. Resume file upload
* firstStageId → from customFields or hardcoded fallback "a06bn00000DStwZAAT"
* applicationId / candidateId → returned from create steps via response handler

### APPLICATION_STAGE

* atsApplicationId → Present
* atsCandidateId → Present
* stageId / stageName → bpats__Status__c (Salesforce field name — no label mapping)
* appliedAt → bpats__Applied_Date__c
* createdDate → CreatedDate
* lastUpdatedDate → LastModifiedDate
* insertedAt / updatedAt → $now()
* dateRangeConfig → LastModifiedDate (delta sync supported)
* paginationConfig → ExpressionConfig (cursor-based)

---

## Issues

1. APPLICATION WRITE: firstStageId fallback hardcodes Salesforce ID `a06bn00000DStwZAAT` — must be updated per customer
2. APPLICATION WRITE: WRITE_API_RESPONSE_HANDLER references `assymbl_contact_create[0].candidateId` — typo risk if integration ID is spelled differently
3. JOB and APPLICATION_STAGE: insertedAt/updatedAt use `$now()` — does not reflect actual record creation/modification time
4. APPLICATION_STAGE READ: stageId is raw Salesforce Status__c value — no human-readable label transformation
5. All API paths reference hardcoded Salesforce tenant URL — not portable across customers without configuration change

---

## Final Status

SUPPORTED (minor issues with hardcoded values and timestamp accuracy)
