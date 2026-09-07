# Detailed API Test Cases Specification & QA Traceability

This document defines the formal test cases, preconditions, execution steps, test data, and QA traceability for the **Restful-Booker API Testing Automation Suite**.

---

## QA Traceability & Test Engineering Framework

Each test case bridges manual QA analytical thinking with automated Postman/Newman assertions:

$$\text{API Requirement} \longrightarrow \text{Test Scenario} \longrightarrow \text{Test Case ID} \longrightarrow \text{Manual Validation} \longrightarrow \text{Automated Postman Assertion} \longrightarrow \text{Execution Result}$$

---

## 01 Health & Service Availability

### TC-HEALTH-001
- **Scenario**: Validate Restful-Booker uptime and health response.
- **Priority**: High (P1 - Smoke)
- **Preconditions**: Target server is reachable.
- **Steps**:
  1. Send `GET {{baseUrl}}/ping`.
  2. Inspect HTTP status code.
  3. Measure total response duration.
  4. Inspect response body string.
- **Test Data**: None.
- **Expected Result**: HTTP 201 Created, latency < 3000ms, body equals `"Created"`.
- **Automated Assertion**:
  ```javascript
  pm.test("[HEALTH-001] Status code is 201 Created", () => pm.response.to.have.status(201));
  pm.test("[HEALTH-002] Response time is within SLA threshold (< 3000ms)", () => pm.expect(pm.response.responseTime).to.be.below(3000));
  pm.test("[HEALTH-003] Response body confirms service health status", () => pm.expect(pm.response.text()).to.equal("Created"));
  ```
- **Execution Result**: PASSED (Avg 255ms).

---

## 02 Authentication & Authorization

### TC-AUTH-001
- **Scenario**: Generate valid session token with administrative credentials.
- **Priority**: High (P1 - Blocking)
- **Preconditions**: Valid admin username/password in test data.
- **Steps**:
  1. Send `POST {{baseUrl}}/auth` with JSON body: `{"username": "admin", "password": "password123"}`.
  2. Verify HTTP status is 200 OK.
  3. Validate `Content-Type` header includes `application/json`.
  4. Verify response body contains non-empty `token` string.
  5. Store token into environment variable `authToken`.
- **Test Data**: `username: "admin"`, `password: "password123"`.
- **Expected Result**: 200 OK, JSON format, non-empty token string captured.
- **Automated Assertion**:
  ```javascript
  pm.test("[AUTH-001] Status code is 200 OK", () => pm.response.to.have.status(200));
  pm.test("[AUTH-002] Content-Type header is application/json", () => pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json"));
  pm.test("[AUTH-003] Response contains non-empty auth token string", () => {
      var json = pm.response.json();
      pm.expect(json.token).to.be.a("string").and.not.be.empty;
      pm.environment.set("authToken", json.token);
  });
  ```
- **Execution Result**: PASSED.

### TC-AUTH-002
- **Scenario**: Attempt authentication with invalid password.
- **Priority**: Medium (P2)
- **Preconditions**: None.
- **Steps**:
  1. Send `POST {{baseUrl}}/auth` with invalid password.
  2. Verify status code is 200 OK (observed API behavior).
  3. Verify JSON body contains `reason: "Bad credentials"`.
  4. Verify `token` property is absent.
- **Test Data**: `username: "admin"`, `password: "wrong_password_999"`.
- **Expected Result**: 200 OK with `reason: "Bad credentials"`.
- **Automated Assertion**: `[AUTH-004]`, `[AUTH-005]`, `[AUTH-006]`.
- **Execution Result**: PASSED.

### TC-AUTH-003
- **Scenario**: Attempt authentication with empty payload.
- **Priority**: Medium (P2)
- **Preconditions**: None.
- **Steps**:
  1. Send `POST {{baseUrl}}/auth` with `{}` body.
  2. Verify response status is 200 OK and `reason: "Bad credentials"`.
- **Expected Result**: 200 OK with `reason: "Bad credentials"`.
- **Automated Assertion**: `[AUTH-007]`, `[AUTH-008]`.
- **Execution Result**: PASSED.

---

## 03 GET Booking Exploration

### TC-GET-001
- **Scenario**: Retrieve list of all existing booking summaries.
- **Priority**: High (P1)
- **Preconditions**: Service is populated with bookings.
- **Steps**:
  1. Send `GET {{baseUrl}}/booking`.
  2. Validate 200 OK and `application/json` Content-Type.
  3. Validate response is an array of objects containing `bookingid` numbers.
  4. Dynamically set `seedBookingId` to the first item's ID.
- **Expected Result**: 200 OK, non-empty array of `{ bookingid: <number> }`.
- **Automated Assertion**: `[GET-001]`, `[GET-002]`, `[GET-003]`.
- **Execution Result**: PASSED.

### TC-GET-002
- **Scenario**: Retrieve single booking details using valid ID.
- **Priority**: High (P1)
- **Preconditions**: `seedBookingId` exists.
- **Steps**:
  1. Send `GET {{baseUrl}}/booking/{{seedBookingId}}`.
  2. Validate 200 OK and JSON body.
  3. Validate presence of `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates.checkin`, `bookingdates.checkout`.
  4. Validate data types for each field.
- **Expected Result**: 200 OK, valid booking structure.
- **Automated Assertion**: `[GET-004]`, `[GET-005]`, `[GET-006]`.
- **Execution Result**: PASSED.

### TC-GET-003
- **Scenario**: Filter bookings using query parameter `firstname`.
- **Priority**: Medium (P2)
- **Steps**:
  1. Send `GET {{baseUrl}}/booking?firstname=Susan`.
  2. Validate 200 OK and array response.
- **Automated Assertion**: `[GET-007]`, `[GET-008]`.
- **Execution Result**: PASSED.

### TC-GET-004
- **Scenario**: Negative test — Retrieve non-existent numeric booking ID.
- **Priority**: High (P1)
- **Steps**:
  1. Send `GET {{baseUrl}}/booking/999999999`.
  2. Verify HTTP status 404 Not Found.
  3. Verify plain text body equals `"Not Found"`.
- **Automated Assertion**: `[GET-009]`, `[GET-010]`.
- **Execution Result**: PASSED.

### TC-GET-005
- **Scenario**: Negative test — Retrieve non-numeric alphanumeric booking ID.
- **Priority**: Medium (P2)
- **Steps**:
  1. Send `GET {{baseUrl}}/booking/invalid-alphanumeric-id`.
  2. Verify HTTP status 404 Not Found.
  3. Verify body equals `"Not Found"`.
- **Automated Assertion**: `[GET-011]`, `[GET-012]`.
- **Execution Result**: PASSED.

---

## 04 Create Resources (POST)

### TC-POST-001
- **Scenario**: Create a new booking with all valid required fields.
- **Priority**: High (P1)
- **Steps**:
  1. Send `POST {{baseUrl}}/booking` with valid booking payload.
  2. Verify 200 OK.
  3. Verify response contains integer `bookingid` and `booking` object echoing submitted values.
- **Automated Assertion**: `[POST-001]`, `[POST-002]`.
- **Execution Result**: PASSED.

### TC-POST-002
- **Scenario**: Negative test — Submit payload missing mandatory fields (`lastname`, `totalprice`, dates).
- **Priority**: Medium (P2)
- **Steps**:
  1. Send `POST {{baseUrl}}/booking` with only `firstname`.
  2. Verify observed server exception returns 500 Internal Server Error (Defect DEF-001).
  3. Verify text body `"Internal Server Error"`.
- **Automated Assertion**: `[POST-003]`, `[POST-004]`.
- **Execution Result**: PASSED.

### TC-POST-003
- **Scenario**: Submit payload with invalid data types (string for totalprice).
- **Priority**: Low (P3)
- **Steps**:
  1. Send `POST {{baseUrl}}/booking` with string in `totalprice`.
  2. Verify observed loose server coercion returns 200 OK with `totalprice: null`.
- **Automated Assertion**: `[POST-005]`, `[POST-006]`.
- **Execution Result**: PASSED.

---

## 05 Update Resources (PUT)

### TC-PUT-001
- **Scenario**: Full update of existing booking with valid auth cookie token.
- **Priority**: High (P1)
- **Preconditions**: `seedBookingId` and `authToken` exist.
- **Steps**:
  1. Send `PUT {{baseUrl}}/booking/{{seedBookingId}}` with `Cookie: token={{authToken}}` and updated payload.
  2. Verify 200 OK.
  3. Verify updated fields (`lastname: "LeadQAEngineer"`, `totalprice: 380`).
- **Automated Assertion**: `[PUT-001]`, `[PUT-002]`, `[PUT-003]`.
- **Execution Result**: PASSED.

### TC-PUT-002
- **Scenario**: Security test — Full update without authorization token cookie.
- **Priority**: High (P1 - Security)
- **Steps**:
  1. Send `PUT {{baseUrl}}/booking/{{seedBookingId}}` without Cookie header.
  2. Verify HTTP status 403 Forbidden.
  3. Verify text body `"Forbidden"`.
- **Automated Assertion**: `[PUT-004]`, `[PUT-005]`.
- **Execution Result**: PASSED.

### TC-PUT-003
- **Scenario**: Security test — Full update with forged/invalid token.
- **Priority**: High (P1 - Security)
- **Steps**:
  1. Send `PUT {{baseUrl}}/booking/{{seedBookingId}}` with `Cookie: token=invalid_token_99999`.
  2. Verify HTTP status 403 Forbidden.
  3. Verify text body `"Forbidden"`.
- **Automated Assertion**: `[PUT-006]`, `[PUT-007]`.
- **Execution Result**: PASSED.

---

## 06 Partial Update (PATCH)

### TC-PATCH-001
- **Scenario**: Partially update booking fields (`totalprice`, `additionalneeds`) with auth token.
- **Priority**: High (P1)
- **Steps**:
  1. Send `PATCH {{baseUrl}}/booking/{{seedBookingId}}` with `Cookie: token={{authToken}}`.
  2. Verify 200 OK and verify mutated fields.
- **Automated Assertion**: `[PATCH-001]`, `[PATCH-002]`, `[PATCH-003]`.
- **Execution Result**: PASSED.

### TC-PATCH-002
- **Scenario**: Security test — Partial update without auth token.
- **Priority**: High (P1 - Security)
- **Steps**:
  1. Send `PATCH {{baseUrl}}/booking/{{seedBookingId}}` without Cookie header.
  2. Verify 403 Forbidden.
- **Automated Assertion**: `[PATCH-004]`, `[PATCH-005]`.
- **Execution Result**: PASSED.

---

## 07 Delete Resources (DELETE)

### TC-DEL-001
- **Scenario**: Security test — Delete booking without auth token.
- **Priority**: High (P1 - Security)
- **Steps**:
  1. Send `DELETE {{baseUrl}}/booking/{{seedBookingId}}` without Cookie header.
  2. Verify 403 Forbidden and text `"Forbidden"`.
- **Automated Assertion**: `[DEL-001]`, `[DEL-002]`.
- **Execution Result**: PASSED.

### TC-DEL-002
- **Scenario**: Negative test — Delete non-existent booking ID with auth token.
- **Priority**: Medium (P2)
- **Steps**:
  1. Send `DELETE {{baseUrl}}/booking/999999999` with `Cookie: token={{authToken}}`.
  2. Verify observed 405 Method Not Allowed and text `"Method Not Allowed"`.
- **Automated Assertion**: `[DEL-003]`, `[DEL-004]`.
- **Execution Result**: PASSED.

---

## 08 JSON Schema Validation

### TC-SCHEMA-001
- **Scenario**: Strict schema conformance of authentication response.
- **Priority**: High (P1)
- **Steps**: Validate `POST /auth` response against JSON Schema Draft-07 specification (`auth-token.schema.json`).
- **Automated Assertion**: `[SCHEMA-001]`, `[SCHEMA-002]`.
- **Execution Result**: PASSED.

### TC-SCHEMA-002
- **Scenario**: Strict schema conformance of booking detail response.
- **Priority**: High (P1)
- **Steps**: Validate `GET /booking/:id` response against `booking.schema.json`.
- **Automated Assertion**: `[SCHEMA-003]`, `[SCHEMA-004]`.
- **Execution Result**: PASSED.

### TC-SCHEMA-003
- **Scenario**: Strict schema conformance of booking summary array.
- **Priority**: High (P1)
- **Steps**: Validate `GET /booking` response against `booking-summary.schema.json`.
- **Automated Assertion**: `[SCHEMA-005]`, `[SCHEMA-006]`.
- **Execution Result**: PASSED.

---

## 09 Boundary & Input Format Testing

### TC-BOUND-001
- **Scenario**: Boundary test — Create booking with `totalprice = 0`.
- **Priority**: Medium (P2)
- **Steps**: Send `POST /booking` with `totalprice: 0`. Confirm API accepts zero and returns `totalprice: 0`.
- **Automated Assertion**: `[BOUND-001]`, `[BOUND-002]`.
- **Execution Result**: PASSED.

### TC-BOUND-002
- **Scenario**: Robustness / Stress test — Create booking with large string inputs (>500 chars).
- **Priority**: Low (P3)
- **Steps**: Send `POST /booking` with extensive strings. Confirm API returns 200 OK and retains fields.
- **Automated Assertion**: `[BOUND-003]`, `[BOUND-004]`.
- **Execution Result**: PASSED.

---

## 10 Critical End-to-End Request Chaining Workflow

### TC-CHAIN-001 through TC-CHAIN-007
- **Scenario**: Complete stateful resource lifecycle with dynamic variable propagation:
  1. `POST /auth` $\rightarrow$ Extract `authToken`.
  2. `POST /booking` $\rightarrow$ Extract `chainedBookingId`.
  3. `GET /booking/{{chainedBookingId}}` $\rightarrow$ Verify created state.
  4. `PUT /booking/{{chainedBookingId}}` $\rightarrow$ Mutate values with `authToken`.
  5. `GET /booking/{{chainedBookingId}}` $\rightarrow$ Verify persisted mutation.
  6. `DELETE /booking/{{chainedBookingId}}` $\rightarrow$ Delete resource with `authToken`.
  7. `GET /booking/{{chainedBookingId}}` $\rightarrow$ Verify 404 Not Found teardown.
- **Priority**: Critical (P0 - Core Portfolio Demonstration)
- **Automated Assertions**: `[CHAIN-001]` through `[CHAIN-014]` (14 distinct assertions).
- **Execution Result**: ALL 14 ASSERTIONS PASSED.
