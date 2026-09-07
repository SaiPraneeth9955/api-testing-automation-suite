# API Test Scenarios Specification

This document maps all high-level test scenarios implemented in the **Restful-Booker API Testing Automation Suite**. Every scenario corresponds directly to an executable request within the Postman collection.

---

## Folder 01: Health & Service Availability

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-HEALTH-01** | Service Health & Uptime Check | `GET /ping` | Confirm service responds with `201 Created` status, response time within < 3000ms SLA, and body confirms `"Created"`. |

---

## Folder 02: Authentication & Authorization

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-AUTH-01** | Valid Administrative Authentication | `POST /auth` | Verify valid credentials return `200 OK`, `Content-Type: application/json`, a non-empty `token` string, and store `authToken` dynamically in the environment. |
| **TS-AUTH-02** | Authentication with Invalid Password | `POST /auth` | Verify invalid credentials return `200 OK` (observed API design), `reason: "Bad credentials"`, and no `token` property is present. |
| **TS-AUTH-03** | Authentication with Empty Payload | `POST /auth` | Verify empty JSON object payload `{}` returns `200 OK` with `reason: "Bad credentials"`. |

---

## Folder 03: GET Booking Exploration

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-GET-01** | Retrieve Booking Catalog | `GET /booking` | Validate `200 OK`, JSON array response structure, presence of `bookingid` integers, and set `seedBookingId` from the first element. |
| **TS-GET-02** | Retrieve Single Booking by Valid ID | `GET /booking/:id` | Validate `200 OK`, JSON format, presence of all required fields (`firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates`), and correct data types. |
| **TS-GET-03** | Filter Bookings by Query Parameter | `GET /booking?firstname=Susan` | Verify `200 OK` and that response is a JSON array matching query filter criteria. |
| **TS-GET-04** | Retrieve Non-Existent Booking ID | `GET /booking/999999999` | Validate `404 Not Found` response code and plain text `"Not Found"` body for out-of-range primary key. |
| **TS-GET-05** | Retrieve Non-Numeric Booking ID | `GET /booking/invalid-id` | Validate `404 Not Found` response code and plain text `"Not Found"` body for malformed path parameter. |

---

## Folder 04: Create Resources (POST)

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-POST-01** | Create Booking with Complete Valid Payload | `POST /booking` | Validate `200 OK`, generation of a numeric `bookingid`, and echo validation of sent fields (`firstname`, `lastname`, `totalprice`, `depositpaid`, `additionalneeds`). |
| **TS-POST-02** | Create Booking with Missing Required Fields | `POST /booking` | Verify unhandled server exception returns `500 Internal Server Error` (documented API defect DEF-001) with text `"Internal Server Error"`. |
| **TS-POST-03** | Create Booking with Invalid Data Types | `POST /booking` | Verify loose type coercion behavior where non-numeric price results in `200 OK` with `totalprice: null`. |

---

## Folder 05: Update Resources (PUT)

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-PUT-01** | Full Booking Update with Valid Token | `PUT /booking/:id` | Validate `200 OK`, header `Cookie: token={{authToken}}`, and verify persisted changes to `lastname`, `totalprice`, and `additionalneeds`. |
| **TS-PUT-02** | Full Booking Update Without Auth Token | `PUT /booking/:id` | Security test: verify omission of cookie header returns `403 Forbidden` and text `"Forbidden"`. |
| **TS-PUT-03** | Full Booking Update with Invalid Auth Token | `PUT /booking/:id` | Security test: verify forged token returns `403 Forbidden` and text `"Forbidden"`. |

---

## Folder 06: Partial Update (PATCH)

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-PATCH-01** | Partial Booking Update with Valid Token | `PATCH /booking/:id` | Validate `200 OK`, header `Cookie: token={{authToken}}`, and verify only requested fields (`totalprice`, `additionalneeds`) are updated. |
| **TS-PATCH-02** | Partial Booking Update Without Auth Token | `PATCH /booking/:id` | Security test: verify unauthenticated partial update returns `403 Forbidden`. |

---

## Folder 07: Delete Resources (DELETE)

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-DEL-01** | Delete Booking Without Auth Token | `DELETE /booking/:id` | Security test: verify unauthenticated delete attempt returns `403 Forbidden`. |
| **TS-DEL-02** | Delete Non-Existent Booking ID | `DELETE /booking/999999999` | Negative test: verify deletion of absent resource returns `405 Method Not Allowed` (observed API behavior). |

---

## Folder 08: JSON Schema Validation

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-SCHEMA-01** | Auth Token Schema Conformance | `POST /auth` | Validate response strictly conforms to JSON Schema Draft-07 specification for `{ "token": string }`. |
| **TS-SCHEMA-02** | Booking Detail Schema Conformance | `GET /booking/:id` | Validate full booking object conforms to strict JSON schema definitions for all fields and nested `bookingdates`. |
| **TS-SCHEMA-03** | Booking Summary List Schema Conformance | `GET /booking` | Validate array response conforms to schema ensuring each item contains integer `bookingid`. |

---

## Folder 09: Boundary & Input Format Testing

| Scenario ID | Scenario Title | Method & Route | Target Validation |
| :--- | :--- | :--- | :--- |
| **TS-BOUND-01** | Numeric Lower Boundary: Price = 0 | `POST /booking` | Boundary test: verify API accepts `totalprice: 0` without arithmetic error and preserves `0` in response. |
| **TS-BOUND-02** | String Length Stress: Large Payload | `POST /booking` | Robustness test: verify server accepts large string fields (>500 chars) without truncation crash. |

---

## Folder 10: Critical End-to-End Request Chaining Workflow

| Scenario ID | Step Name | Method & Route | Target Validation & Chained Dependencies |
| :--- | :--- | :--- | :--- |
| **TS-CHAIN-01** | Step 1: Authenticate | `POST /auth` | Validate `200 OK` and extract `authToken` into environment. |
| **TS-CHAIN-02** | Step 2: Create Booking | `POST /booking` | Validate `200 OK` and extract generated `bookingid` as `chainedBookingId`. |
| **TS-CHAIN-03** | Step 3: Retrieve Created | `GET /booking/{{chainedBookingId}}` | Validate `200 OK` and verify created state matches initial payload. |
| **TS-CHAIN-04** | Step 4: Mutate Booking | `PUT /booking/{{chainedBookingId}}` | Send `Cookie: token={{authToken}}`, validate `200 OK`, and verify mutated values in response. |
| **TS-CHAIN-05** | Step 5: Verify Mutation | `GET /booking/{{chainedBookingId}}` | Query server to confirm persisted modifications in the backend. |
| **TS-CHAIN-06** | Step 6: Delete Booking | `DELETE /booking/{{chainedBookingId}}` | Send `Cookie: token={{authToken}}`, validate `201 Created` deletion status. |
| **TS-CHAIN-07** | Step 7: Teardown Audit | `GET /booking/{{chainedBookingId}}` | Confirm resource non-existence with `404 Not Found` and text `"Not Found"`. |
