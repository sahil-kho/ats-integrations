# ATS Validation Skill (Self-contained)

## Objective

Validate ATS integrations using config-driven CURL generation, mapping validation, and platform compatibility.

---

## Step 1: Input

* Customer integration config
* Data model integration config
* .env variables
* Sample ATS response (optional)

---

## Step 2: CURL Generation (NEW)

* Generate CURL dynamically using:
  * Base URL
  * API path
  * Method
  * Headers
  * Payload
* Inject variables from .env

---

## Step 3: CURL Validation

* Validate:
  * HTTP method
  * Headers (Authorization, Content-Type)
  * Payload structure

---

## Step 4: Execution (Manual / Postman)

* Run CURL in Postman
* Capture API response

---

## Step 5: Mapping Validation

* Apply JSONata mapping
* Verify required fields:

  * atsJobId
  * title
  * url

---

## Step 6: Data Inspection (UPDATED)

* Display mapped output in **table format (in chat)**
* Identify:
  * Missing fields
  * Null values
  * Incorrect mappings

---

## Step 7: Platform Compatibility

Check:

* Integration type supported
* Auth supported
* Data model supported

---

## Step 8: Data Quality

* Missing fields
* Duplicate records
* Incorrect values
* Timestamp issues

---

## Step 9: Output

Generate:

* validation_output.json
* support_alignment.md

---

## Final Result

Classify:

* SUPPORTED
* PARTIAL
* NOT SUPPORTED