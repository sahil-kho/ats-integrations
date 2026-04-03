# Current Support Features (Simplified)

## Integration Modes

* REST_API → Supported
* WRITE_REST_API → Supported
* REST_API_RESPONSE_HANDLER → Supported

---

## Authentication

* API_KEY → Supported
* BASIC → Supported
* OAUTH_2 → Supported

---

## Data Models

* JOB → Supported (requires atsJobId)
* APPLICATION → Supported
* APPLICATION_STAGE → Supported
* ATTACHMENT → Supported

---

## Pagination

* Page-based → Supported
* Offset-based → Supported
* Expression-based → Supported

---

## Response Types

* JSON → Supported
* XML → Supported (converted to JSON)

---

## Important Rules

### JOB

* atsJobId required
* title, url required for downstream

### APPLICATION

* atsJobId, atsCandidateId, atsApplicationId required

### APPLICATION_STAGE

* atsJobId, atsCandidateId, atsApplicationId, stageId required

---

## Notes

* updated_at used for downstream sync
* Missing required fields → NOT SUPPORTED
