# QA Test Execution Summary Report

## 1. Executive Summary

This QA Summary Report presents the results of the automated test execution for the **API Testing Automation Suite**, evaluated against the `Restful-Booker` REST API platform.

The automated suite was executed using Newman v6.2.2 and the `newman-reporter-htmlextra` visual reporter, without execution-aborting flags (`--bail`), allowing 100% of planned test requests and assertions to run to completion.

---

## 2. Test Execution Metrics (Actual Verified Results)

| Metric | Result |
| :--- | :--- |
| **Execution Tool** | Newman CLI v6.2.2 |
| **Reporter** | newman-reporter-htmlextra v1.23.1 |
| **Target Host** | `https://restful-booker.herokuapp.com` |
| **Total Test Iterations** | 1 |
| **Total HTTP Requests** | 31 |
| **Total Test Scenarios** | 31 |
| **Total Executed Assertions** | 69 |
| **Passed Assertions** | **69** |
| **Failed Assertions** | **0** |
| **Success Rate** | **100%** |
| **Total Execution Duration** | 11.7 seconds |
| **Average Latency** | 293 ms |
| **Min Latency / Max Latency** | 223 ms / 917 ms |
| **Total Data Received** | ~87.93 kB |

---

## 3. Test Coverage Breakdown by Module

| Module / Folder | Requests | Assertions | Pass Rate | Key Validations |
| :--- | :---: | :---: | :---: | :--- |
| **01 Health & Service Availability** | 1 | 3 | 100% | Status 201, latency SLA (<3000ms), service status text |
| **02 Authentication & Authorization** | 3 | 8 | 100% | Valid auth, invalid password, empty body, token capture |
| **03 GET Booking Exploration** | 5 | 12 | 100% | Catalog list, single record, query param, 404 non-existent & malformed |
| **04 Create Resources (POST)** | 3 | 6 | 100% | Valid creation, missing fields (500), type coercion (200) |
| **05 Update Resources (PUT)** | 3 | 7 | 100% | Valid update with token, 403 without token, 403 invalid token |
| **06 Partial Update (PATCH)** | 2 | 5 | 100% | Valid partial update with token, 403 without token |
| **07 Delete Resources (DELETE)** | 2 | 4 | 100% | 403 without token, 405 on non-existent ID |
| **08 JSON Schema Validation** | 3 | 6 | 100% | Auth token schema, booking detail schema, summary array schema |
| **09 Boundary & Format Testing** | 2 | 4 | 100% | Lower boundary `totalprice: 0`, string length stress testing |
| **10 End-to-End Request Chaining** | 7 | 14 | 100% | Auth $\rightarrow$ Create $\rightarrow$ Read $\rightarrow$ Mutate $\rightarrow$ Read $\rightarrow$ Delete $\rightarrow$ Verify 404 |

---

## 4. Controlled Failure Behavior Verification

In compliance with QA automation standards, failure propagation was directly validated:
1. A deliberate failing assertion was injected into scenario `[HEALTH-001]` (expecting `200` instead of `201`).
2. Newman executed the entire suite without `--bail`, reported the specific assertion failure in console and HTML report, and exited with exit code `1`.
3. The assertion was restored to `201`, returning the suite to a clean exit code `0`.

---

## 5. Identified Defects and Architectural Observations

1. **DEF-001 (Major - Backend Exception)**: Missing mandatory fields in `POST /booking` triggers an unhandled `500 Internal Server Error` instead of a validation error (`400 Bad Request`).
2. **DEF-002 (Minor - Contract Quirk)**: `POST /auth` returns `200 OK` with `{"reason":"Bad credentials"}` for invalid logins instead of `401 Unauthorized`.
3. **DEF-003 (Minor - HTTP Status)**: `DELETE /booking/:id` on a non-existent ID returns `405 Method Not Allowed` rather than `404 Not Found`.

---

## 6. Known API Limitations

- **Transient In-Memory Persistence**: Restful-Booker runs in a shared cloud sandbox (Heroku). In-memory database records persist during active execution cycles, but periodic dyno resets occur (~10-15 min). The suite mitigates this by dynamically creating and managing its own test data in each run.
- **Plain Text Error Bodies**: Error statuses (`403`, `404`, `405`, `500`) return plain text rather than JSON error payloads.

---

## 7. Final QA Assessment

The **API Testing Automation Suite** is verified as **robust, fully reproducible, and production-ready for portfolio presentation**. It provides full regression confidence, handles dynamic state cleanly, enforces security boundaries, and integrates seamlessly with headless CI/CD execution.
