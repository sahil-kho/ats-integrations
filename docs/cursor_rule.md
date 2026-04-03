### Cursor Rule: CURL Generation & Validation

#### Objective

Generate CURL dynamically from integration configs and validate it.

#### Inputs

* Customer Integration Config
* Data Model Integration Config
* .env variables

#### Steps

1. Generate CURL

* Extract:

  * Base URL
  * API path
  * Method
  * Headers
  * Payload
* Construct CURL dynamically

2. Inject Variables

* Replace dynamic values using .env

3. Validate CURL

* Check method correctness
* Check headers (auth, content-type)
* Check required fields in payload

4. Execute (manual/Postman)

* Verify response success
* Capture response structure

5. Data Inspection

* Present response in structured table format (in chat)
* Highlight missing or null fields

6. Output

* Generated CURL
* Validation result
* Issues (if any)
