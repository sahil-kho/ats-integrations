# ATS Validation Reference (Self-contained)

## Purpose

Defines how ATS integrations are validated using a config-driven approach.

---

## Validation Structure

### 1. Metadata

* ATS Name
* Data Model (JOB / APPLICATION / APPLICATION_STAGE)
* Integration Type

---

### 2. Field Mapping

Map ATS fields → platform fields (DB schema)

---

### 3. Platform Alignment

Check against supported features:

* Integration type (REST_API / WRITE_REST_API)
* Auth type (API_KEY / OAUTH / etc.)
* Data model support
* Required identifiers

---

### 4. Request Generation (NEW)

* Generate CURL dynamically using:
  * Customer Integration Config
  * Data Model Integration Config
  * .env variables

---

### 5. Data Inspection (UPDATED)

* Inspect API response in **tabular format (in chat / console)**
* No DB interaction
* No file generation required

---

### 6. Data Quality Checks

* Missing required fields
* Duplicate records
* Incorrect mappings
* Placeholder values

---

### 7. Final Classification

* SUPPORTED
* PARTIAL
* NOT SUPPORTED