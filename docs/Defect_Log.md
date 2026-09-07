# QA Defect Log — Restful-Booker Suite

## 1. Overview
This document records genuine defects and non-standard architectural behaviors identified during empirical exploration and automated testing of the `Restful-Booker` REST API. A standard QA Defect Report Template is also provided at the bottom of this document for future defect tracking.

---

## 2. Genuinely Discovered Defects

### Defect DEF-001: Unhandled Server Exception (500) on Missing Required Booking Fields

- **Defect ID**: `DEF-001`
- **Module / API**: Booking Engine (`/booking`)
- **Endpoint**: `/booking`
- **HTTP Method**: `POST`
- **Summary**: Omitting required payload fields causes an unhandled backend crash returning `500 Internal Server Error` instead of `400 Bad Request`.
- **Severity**: Major (S2)
- **Priority**: High (P2)
- **Preconditions**: Target API is operational.
- **Request Details**:
  - **Headers**: `Content-Type: application/json`, `Accept: application/json`
  - **Payload**:
    ```json
    {
      "firstname": "IncompletePayloadOnly"
    }
    ```
- **Steps to Reproduce**:
  1. Open Postman or terminal HTTP client.
  2. Issue a `POST` request to `https://restful-booker.herokuapp.com/booking`.
  3. Provide a JSON payload containing only `firstname`, omitting `lastname`, `totalprice`, `depositpaid`, and `bookingdates`.
  4. Send request and inspect response status and body.
- **Expected Result**: The API should validate incoming request schemas and return `400 Bad Request` or `422 Unprocessable Entity` with a descriptive JSON error message indicating which fields are missing.
- **Actual Result**: The API crashes internally with `500 Internal Server Error` and returns plain text `"Internal Server Error"`.
- **Evidence**:
  ```http
  HTTP/1.1 500 Internal Server Error
  Content-Type: text/plain; charset=utf-8
  Content-Length: 21

  Internal Server Error
  ```
- **Status**: Open (Backend Public API behavior)
- **Notes / Test Decision**: Handled in test scenario `[POST-003]` by asserting the observed `500` status to ensure reliable test execution while tracking the root defect here.

---

### Defect DEF-002: Inappropriate 200 OK Status Code on Authentication Failure

- **Defect ID**: `DEF-002`
- **Module / API**: Authentication Service (`/auth`)
- **Endpoint**: `/auth`
- **HTTP Method**: `POST`
- **Summary**: Invalid credentials return HTTP `200 OK` status with an error body instead of standard HTTP `401 Unauthorized`.
- **Severity**: Minor / Architectural (S3)
- **Priority**: Medium (P3)
- **Preconditions**: None.
- **Request Details**:
  - **Payload**:
    ```json
    {
      "username": "admin",
      "password": "wrong_password_999"
    }
    ```
- **Steps to Reproduce**:
  1. Issue `POST` request to `https://restful-booker.herokuapp.com/auth` with invalid credentials.
  2. Inspect response status code and body.
- **Expected Result**: The server should return `401 Unauthorized` with an error payload.
- **Actual Result**: The server returns `200 OK` with JSON body: `{"reason": "Bad credentials"}`.
- **Evidence**:
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/json; charset=utf-8

  {"reason":"Bad credentials"}
  ```
- **Status**: Open (Public API design quirk)
- **Notes**: In production REST services, HTTP status codes should reflect transport/authorization status. Automated test scenario `[AUTH-004]` asserts `200 OK` and inspects `reason` to validate the contract accurately.

---

### Defect DEF-003: 405 Method Not Allowed Returned When Deleting Non-Existent ID

- **Defect ID**: `DEF-003`
- **Module / API**: Booking Engine (`/booking`)
- **Endpoint**: `/booking/:id`
- **HTTP Method**: `DELETE`
- **Summary**: Deleting a non-existent booking ID with valid auth token returns `405 Method Not Allowed` instead of `404 Not Found`.
- **Severity**: Minor (S3)
- **Priority**: Medium (P3)
- **Request Details**:
  - **URL**: `https://restful-booker.herokuapp.com/booking/999999999`
  - **Headers**: `Cookie: token=<valid_token>`
- **Expected Result**: HTTP `404 Not Found` (the target resource does not exist).
- **Actual Result**: HTTP `405 Method Not Allowed` with plain text body `"Method Not Allowed"`.
- **Status**: Open
- **Notes**: Documented in test case `[DEL-003]`.

---

## 3. Standard QA Defect Report Template (Sample Reference)

```markdown
### [DEF-XXX] Short Defect Title

- **Defect ID**: DEF-XXX
- **Module / API**: [e.g. Payments / Booking / Auth]
- **Endpoint**: [e.g. /api/v1/orders]
- **HTTP Method**: [GET / POST / PUT / PATCH / DELETE]
- **Summary**: Clear one-line summary of the bug.
- **Severity**: [Critical / Major / Moderate / Minor]
- **Priority**: [P1 / P2 / P3 / P4]
- **Preconditions**: Any necessary system state or prerequisite resources.
- **Request Details**:
  - Headers:
  - Query/Path Parameters:
  - Payload:
- **Steps to Reproduce**:
  1. Step 1...
  2. Step 2...
  3. Step 3...
- **Expected Result**: What should happen per API specification.
- **Actual Result**: What actually occurred.
- **Evidence**: Raw HTTP request/response headers and body.
- **Status**: [New / Open / In Review / Resolved / Closed]
- **Notes**: Additional context or QA recommendations.
```
