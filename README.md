# REST API Test Scenarios Checklist

This checklist outlines key API test scenarios including positive, negative, and destructive tests. It ensures validation of status codes, payloads, headers, and performance to confirm API correctness, robustness, and compliance with specifications. It is based on real-world experience developing and maintaining API test frameworks across multiple companies.

---

## 1. Basic Positive Tests (Happy Paths)

### Test Action:
- Execute API call with **valid required parameters**

### Validate Status Code:
- Confirm response returns a 2XX HTTP status code
- Match returned status code to spec:
    - `200 OK` for GET requests
    - `201 Created` for POST or PUT requests that create a resource
    - `200`, `202 Accepted`, or `204 No Content` for DELETE operations

### Validate Payload:
- Response must be a **well-formed JSON object**
- Check structure against the data model:
    - Correct field names
    - Correct field types
    - All expected fields are present (including nested ones)
    - Field values are valid and make sense
    - Non-nullable fields are **not null**

### Validate State:
- For GET: Ensure **no state change** occurs in the system
- For POST, DELETE, PATCH, PUT:
    - Confirm the operation was performed successfully
    - Make a follow-up GET request to verify updated data
    - If applicable, refresh the UI or API and confirm changes

### Validate Headers:
- Ensure headers are correct:
    - `Content-Type`
    - `Connection`
    - `Cache-Control`
    - `Expires`
    - `Access-Control-Allow-Origin`
    - `Keep-Alive`
    - `Strict-Transport-Security` (HSTS), etc.
- Ensure no sensitive data is leaked (e.g., `X-Powered-By` should **not** be present)

### Performance Sanity:
- Response must be received **within acceptable time limits** defined in the test plan

---

## 2. Positive Tests with Optional Parameters

### Test Action:
- Execute API call with valid required **and** valid optional parameters

### Validate Status Code:
- Confirm response returns appropriate 2XX status code:
    - `200 OK`, `201 Created`, etc., as appropriate to method

### Validate Payload:
- Must be a **valid JSON** and match expected data model
- Check for correct handling of optional parameters:
    - `filter`: Response only includes records matching the filter
    - `sort`: Response is sorted by the specified field (check both ascending and descending)
    - `skip`: Response omits the specified number of initial results
    - `limit`: Response includes only up to the specified number of items
    - `limit + skip`: Pagination works as expected
    - Check combinations: `filter + sort + limit + skip` return expected results

### Validate State:
- For GET: Ensure **no state change** occurs
- For POST, DELETE, PATCH, PUT:
    - Verify action was correctly applied
    - Use a GET request or UI refresh to confirm state

### Validate Headers:
- Headers must be accurate and complete:
    - `Content-Type`, `Connection`, `Cache-Control`, `Expires`, `Access-Control-Allow-Origin`, etc.
- Ensure no unnecessary or insecure headers are present

### Performance Sanity:
- Confirm that the response is **timely and performant**

---

## 3. Negative Testing – Valid Input with Illegal Operations

### Test Action:
Execute calls that are valid in format but logically incorrect:
- Create resource with a duplicate name
- Delete a non-existent resource
- Update resource using legal format but conflicting data (e.g., name already taken)
- Unauthorised actions (e.g., delete without permissions)

### Validate Status Code:
- HTTP status code **must not be 2XX**
- Response must include an **error code** defined by API spec

### Validate Payload:
- Ensure error response is present
- Error format matches API spec (structured JSON or plain text)
- Message must be:
    - Clear and descriptive
    - Relevant to the failure cause
    - Aligned with documented error messages

### Validate Headers:
- Check standard headers:
    - `Content-Type`, `Connection`, `Cache-Control`, etc.
- Confirm that security-related headers are enforced
- No confidential data leaked through headers

### Performance Sanity:
- Even error responses should be returned **quickly and reliably**

---

## 4. Negative Testing – Invalid Input

### Test Action:
Send API requests with invalid/malformed input:
- Missing or invalid auth token
- Missing required parameters
- Invalid UUIDs in path/query
- Payload violating schema (missing fields, bad types)
- Nested object errors
- Illegal values in headers
- Unsupported HTTP methods

### Validate Status Code:
- Response must be a **client error** (4XX)
- Status code should reflect the type of error (e.g., `400`, `401`, `403`, `405`)

### Validate Payload:
- Error message must:
    - Be present and correctly formatted
    - Provide useful, specific details about the problem

### Validate Headers:
- Confirm appropriate headers are present and correct
- Ensure response does not leak data via headers

### Performance Sanity:
- Response time must still meet acceptable limits

---

## 5. Destructive Testing

### Test Action:
Test API robustness by simulating faulty or malicious input:
- Malformed request content
- Incorrect or missing `Content-Type`
- Invalid payload structure
- Parameter overflow:
    - e.g., title > 200 characters
    - huge payloads (large JSON bodies)
    - extremely long UUIDs
- Empty or partially empty payloads/sub-objects
- Special characters or encoding issues in inputs
- Incorrect headers
- Concurrent operations on same resource (race conditions. e.g. DELETE + PATCH, etc)
- Exploratory tests and edge cases

### Validate Status Code:
- API must fail **gracefully** (return proper error codes, no crashes)

### Validate Payload:
- Error responses should follow error format (structured JSON or string)
- Include a meaningful description of what failed

### Validate Headers:
- All response headers should be valid
- Avoid exposing internal implementation details

### Performance Sanity:
- Even when failing, API must respond in a timely manner

