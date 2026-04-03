# Apploi – Platform Alignment

## Integration Modes

* JOB → REST_API → Supported
* APPLICATION → WRITE_REST_API → Supported
* APPLICATION_STAGE → REST_API → Partially Supported

---

## Authentication

* API_KEY → Supported

---

## Data Model Validation

### JOB

* atsJobId → Present
* title → Present
* url → Present

### APPLICATION

* Candidate + Application creation supported

### APPLICATION_STAGE

* Missing:

  * atsApplicationId
  * atsCandidateId
  * atsJobId

---

## Issues

1. updatedAt uses system time ($now)
2. No pagination configured
3. Funnel tracking incomplete

---

## Final Status

SUPPORTED (PARTIAL for APPLICATION_STAGE / funnel tracking)