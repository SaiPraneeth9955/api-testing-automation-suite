# Observed REST API Behavior — Restful-Booker

## Purpose and QA Methodology

In production QA engineering, automated test suites must never be constructed solely on assumptions or outdated Swagger specifications. Instead, QA engineers explore, probe, and document the genuine runtime characteristics of the API under test before codifying assertions.

This document details the **empirically observed behavior** of the `Restful-Booker` API (`https://restful-booker.herokuapp.com`), captured through automated Node.js probes and Newman test execution. Every assertion in the Postman collection directly maps to these verified realities.

---

## Empirical Observation Matrix

| Scenario / Objective | Method & Endpoint | Observed Status | Response Content-Type | Observed Body Format & Content | QA Test Design Decision |
| :--- | :--- | :---: | :--- | :--- | :--- |
| **Healthcheck Ping** | `GET /ping` | `201 Created` | `text/plain; charset=utf-8` | String `"Created"` | Validate `201` status, latency < 3000ms SLA, and string `"Created"`. |
| **Valid Authentication** | `POST /auth` | `200 OK` | `application/json; charset=utf-8` | `{"token":"<hex_string>"}` | Validate `200 OK`, JSON schema compliance, and dynamically capture `authToken`. |
| **Invalid Credentials** | `POST /auth` | `200 OK` | `application/json; charset=utf-8` | `{"reason":"Bad credentials"}` | Validate `200 OK` (observed API design choice) and assert `reason` field equals `"Bad credentials"`. |
| **Empty Auth Body** | `POST /auth` | `200 OK` | `application/json; charset=utf-8` | `{"reason":"Bad credentials"}` | Validate `200 OK` and error reason presence. |
| **Retrieve All Bookings** | `GET /booking` | `200 OK` | `application/json; charset=utf-8` | Array of objects `[{"bookingid": 1}, ...]` | Validate array structure, summary schema, and capture `seedBookingId` for exploratory tests. |
| **Get Booking by Valid ID** | `GET /booking/:id` | `200 OK` | `application/json; charset=utf-8` | Object with `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates` | Validate `200 OK`, all required fields, valid data types, and JSON schema. |
| **Query Parameter Filter** | `GET /booking?firstname=Susan` | `200 OK` | `application/json; charset=utf-8` | Array of matching `[{"bookingid": ...}]` | Validate query parameter parsing and array response. |
| **Non-Existent ID** | `GET /booking/999999999` | `404 Not Found` | `text/plain; charset=utf-8` | Plain text `"Not Found"` | Validate `404 Not Found` and plain text body. Do NOT expect a JSON error schema. |
| **Non-Numeric ID Format** | `GET /booking/invalid-id` | `404 Not Found` | `text/plain; charset=utf-8` | Plain text `"Not Found"` | Validate `404 Not Found` and plain text body. |
| **Create Valid Booking** | `POST /booking` | `200 OK` | `application/json; charset=utf-8` | `{"bookingid": <int>, "booking": {...}}` | Validate `200 OK`, capture `bookingid`, and verify returned data matches sent payload. |
| **Create Booking Missing Fields** | `POST /booking` | `500 Internal Server Error` | `text/plain; charset=utf-8` | Plain text `"Internal Server Error"` | **Document as API Defect (DEF-001)**: Missing fields crash backend handler rather than returning `400 Bad Request`. Assert observed `500` to prevent test flakiness. |
| **Create Booking Invalid Types** | `POST /booking` | `200 OK` | `application/json; charset=utf-8` | `{"bookingid": ..., "booking": {"totalprice": null, ...}}` | **Document loose coercion**: Server coerces string to `null` rather than rejecting with `422/400`. Validate server response integrity. |
| **Boundary: Price = 0** | `POST /booking` | `200 OK` | `application/json; charset=utf-8` | `{"bookingid": ..., "booking": {"totalprice": 0, ...}}` | Server accepts numeric `0`. Assert `totalprice` equals `0` without false rejection claims. |
| **Boundary: Large Strings** | `POST /booking` | `200 OK` | `application/json; charset=utf-8` | Retains full string payload without crashing | Server handles large strings (>500 chars). Validate stability. |
| **PUT with Valid Token** | `PUT /booking/:id` | `200 OK` | `application/json; charset=utf-8` | Updated booking object | Validate `200 OK` and verify mutated fields. Requires `Cookie: token=<token>`. |
| **PUT without Token** | `PUT /booking/:id` | `403 Forbidden` | `text/plain; charset=utf-8` | Plain text `"Forbidden"` | Validate `403 Forbidden` and text body. Real token authorization enforcement! |
| **PUT with Invalid Token** | `PUT /booking/:id` | `403 Forbidden` | `text/plain; charset=utf-8` | Plain text `"Forbidden"` | Validate `403 Forbidden` on forged token. |
| **PATCH with Valid Token** | `PATCH /booking/:id` | `200 OK` | `application/json; charset=utf-8` | Updated booking object | Validate `200 OK` and verify patched fields. |
| **PATCH without Token** | `PATCH /booking/:id` | `403 Forbidden` | `text/plain; charset=utf-8` | Plain text `"Forbidden"` | Validate `403 Forbidden`. |
| **DELETE without Token** | `DELETE /booking/:id` | `403 Forbidden` | `text/plain; charset=utf-8` | Plain text `"Forbidden"` | Validate `403 Forbidden`. |
| **DELETE with Valid Token** | `DELETE /booking/:id` | `201 Created` | `text/plain; charset=utf-8` | Plain text `"Created"` | Validate `201 Created` (Restful-Booker design choice for delete confirmation). |
| **GET After DELETE** | `GET /booking/:id` | `404 Not Found` | `text/plain; charset=utf-8` | Plain text `"Not Found"` | **Proves genuine in-session CRUD persistence and complete teardown!** |
| **DELETE on Non-Existent ID** | `DELETE /booking/999999999` | `405 Method Not Allowed` | `text/plain; charset=utf-8` | Plain text `"Method Not Allowed"` | Server returns `405` instead of `404` when deleting absent IDs. Validate observed status. |

---

## Key Takeaways for Interview Explanation

1. **Why empirical verification matters**:
   - If we had simply followed generic REST conventions, we might have assumed `POST /auth` failure returns `401 Unauthorized`, `DELETE` returns `204 No Content`, or missing fields return `400 Bad Request`.
   - By running empirical probes first, we discovered the real application behavior (`200 OK` with `reason`, `201 Created` on delete, and `500 Internal Server Error` on missing payload fields).
2. **Error formats are plain text, not universal JSON**:
   - Error responses (`403`, `404`, `405`, `500`) return `text/plain` bodies (`"Forbidden"`, `"Not Found"`, etc.). Asserting JSON schemas on these endpoints would have caused artificial test failures.
3. **Genuine Persistence Verified**:
   - Unlike mock services (ReqRes, DummyJSON), `Restful-Booker` actually creates in-memory database records that are retrievable by ID, modifiable, and verify 404 upon deletion.
