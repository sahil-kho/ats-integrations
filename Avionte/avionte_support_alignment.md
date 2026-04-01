# Avionte – Platform Alignment

## Integration Modes

* APPLICATION_STAGE → WEBHOOK → Supported (nomination and pipeline event types)
* APPLICATION_STAGE → REST_API enrichment → Supported (talent lookup after webhook)
* APPLICATION → REST_API → Partially Supported (web application path observed)
* JOB → REST_API → Partially Supported (not fully confirmed from available config sections)

---

## Authentication

* OAUTH_2 → Supported (client_credentials grant)
* Base URL: https://api.avionte.com
* Additional per-request headers required: `Tenant`, `x-api-key` (must be in customerStaticValues)

---

## Data Model Validation

### APPLICATION_STAGE (Nomination Event Path)

* atsJobId → jobId from stage record
* atsCandidateId → talentId from stage record
* atsApplicationId → id (nomination ID)
* stageId / stageName → jobActivityName with fallback to stageType
* appliedAt / createdDate → stagedDate
* lastUpdatedDate → declinedStageDate or jobActivityStartDate or stagedDate (with $now() fallback)
* insertedAt / updatedAt → $now()
* Custom fields: bucket="stage_event", nominationId, email from talent lookup

### APPLICATION_STAGE (Pipeline Event Path)

* atsJobId → jobId from pipeline record
* atsCandidateId → talentId
* atsApplicationId → pipelineId
* stageId / stageName → "Application Declined" if declinedDate exists, else pipelineStage
* appliedAt / createdDate / lastUpdatedDate → declinedDate or pipelinedDate
* insertedAt / updatedAt → $now()
* Custom fields: bucket="pipeline_event", pipelineId, source, email

### Webhook Trigger

* dataModelRules check: `eventName & EventName != ""` — camelCase/PascalCase union pattern
* nominationId/pipelineId extracted by string-parsing raw `resource`/`Resource` JSON field

---

## Issues

1. APPLICATION_STAGE WEBHOOK: dataModelRules concatenate camelCase `eventName` and PascalCase `EventName` — logic is brittle for payloads that only send one casing variant
2. APPLICATION_STAGE: nominationId and pipelineId are extracted via substring parsing of raw JSON string — brittle and will break on schema changes
3. APPLICATION_STAGE: lastUpdatedDate uses `$now()` fallback when field is absent — actual event timestamps may be lost
4. All API paths include absolute base URL (e.g., `https://api.avionte.com/front-office/v1/...`) — does not use customerBaseUrl; non-standard pattern
5. Requires `Tenant` and `x-api-key` custom headers per request — must be correctly configured in customerStaticValues
6. Nomination and pipeline events each trigger a 2-step API enrichment chain — N+2 API calls per webhook event, performance concern at scale

---

## Final Status

PARTIAL (funnel tracking via webhook supported; JOB/APPLICATION ingestion not fully confirmed; fragile webhook parsing)
