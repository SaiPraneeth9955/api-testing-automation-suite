# Interview Technical Defense & QA Explanation Guide

This document contains precise, technically sound answers to common interview questions regarding this project, covering API testing principles, architectural decisions, and portfolio differentiation.

---

## 1. Project Overview & Motivation

### Q1: What problem does this project solve?
**Answer**: Manual API validation becomes slow, repetitive, and error-prone as endpoints and parameter permutations increase. This project solves that by providing an automated API regression suite that validates HTTP contracts, data integrity, security controls, and stateful workflows headlessly from the command line and within CI/CD pipelines.

### Q2: Why did you build this project?
**Answer**: To demonstrate deep competence in backend API testing, complementing my UI test automation portfolio. While UI automation validates visible user interfaces through a browser, backend API automation validates core business logic, schema contracts, HTTP headers, authentication tokens, and direct database-mutating operations without browser rendering overhead.

### Q3: What API did you test and why did you choose it?
**Answer**: I tested `Restful-Booker` (`https://restful-booker.herokuapp.com`). Unlike purely simulated practice APIs (such as ReqRes or DummyJSON), Restful-Booker provides:
1. Genuine in-session stateful CRUD persistence (created records can be retrieved, mutated, and deleted).
2. Token-gated authentication and authorization checks returning `403 Forbidden` when unauthenticated.
3. Realistic request chaining opportunities.

---

## 2. Core REST & HTTP Fundamentals

### Q4: What is REST API testing?
**Answer**: REST (Representational State Transfer) API testing is the programmatic validation of web services that communicate over HTTP using standard methods (GET, POST, PUT, PATCH, DELETE) and standardized payload formats (typically JSON). We validate status codes, response headers, schema conformity, business data correctness, and latency.

### Q5: What are HTTP methods and how did you use them?
**Answer**:
- **GET**: Safe, idempotent retrieval of resources (`GET /booking` catalog, `GET /booking/:id`).
- **POST**: Non-idempotent creation of resources (`POST /booking`) or session creation (`POST /auth`).
- **PUT**: Idempotent full replacement of a resource with a complete updated payload (`PUT /booking/:id`).
- **PATCH**: Partial modification of specified fields (`PATCH /booking/:id`).
- **DELETE**: Removal of a resource (`DELETE /booking/:id`).

### Q6: What is the difference between Path Parameters and Query Parameters?
**Answer**:
- **Path Parameters**: Identify a specific resource in the hierarchical URI path (e.g. `{{baseUrl}}/booking/1` where `1` is the primary key).
- **Query Parameters**: Filter, sort, or paginate collections following the `?` delimiter (e.g. `{{baseUrl}}/booking?firstname=Susan`).

### Q7: What is the difference between Authentication and Authorization?
**Answer**:
- **Authentication**: Verifying *who* you are (identity). In our suite, `POST /auth` submits credentials and verifies the caller's identity by issuing a dynamic session token.
- **Authorization**: Verifying *what* you are permitted to do (permissions). In our suite, sending `PUT`, `PATCH`, or `DELETE` without providing the token cookie returns `403 Forbidden`, proving that unauthenticated callers are denied state-mutating permissions.

---

## 3. Tooling: Postman & Newman

### Q8: Why Postman and JavaScript?
**Answer**: Postman is the industry standard for API exploration and collection authoring. Postman's JavaScript test sandbox (`pm.*` API with Chai assertions) allows writing flexible, expressive assertions for status codes, headers, response JSON structures, dynamic variables, and schema validation.

### Q9: What is Newman and why is it needed?
**Answer**: Newman is Postman's official command-line collection runner. Postman Desktop is a manual graphical interface that cannot be triggered inside automated CI/CD servers. Newman takes the exported Postman collection and environment JSON files, executes all requests headlessly in Node.js, prints terminal results, and generates visual HTML reports.

### Q10: How does Newman communicate with CI/CD?
**Answer**: Newman runs headlessly and returns a standard process exit code (`0` for complete pass, `1` or non-zero if any assertion fails). Continuous Integration runners like GitHub Actions listen to this exit code; if Newman exits with non-zero, GitHub Actions automatically flags the pipeline build as failed.

---

## 4. Advanced Concepts: Request Chaining & State Management

### Q11: What is Request Chaining?
**Answer**: Request chaining is extracting dynamic runtime data from the response of Request A, storing it in an environment variable, and passing it as a parameter, header, or payload field into subsequent Request B.

### Q12: Walk me through the exact Request Chaining workflow in your project.
**Answer**:
Our critical 7-step workflow demonstrates complete end-to-end chaining:
1. **Step 1 (POST /auth)**: Send admin credentials $\rightarrow$ Extract `jsonData.token` $\rightarrow$ Store in `pm.environment.set("authToken", jsonData.token)`.
2. **Step 2 (POST /booking)**: Send booking payload $\rightarrow$ Extract generated `jsonData.bookingid` $\rightarrow$ Store in `pm.environment.set("chainedBookingId", jsonData.bookingid)`.
3. **Step 3 (GET /booking/{{chainedBookingId}})**: Use chained ID in path parameter $\rightarrow$ Verify newly created record.
4. **Step 4 (PUT /booking/{{chainedBookingId}})**: Pass `Cookie: token={{authToken}}` and `{{chainedBookingId}}` $\rightarrow$ Mutate price to 650 and lastname to "E2EUpdated".
5. **Step 5 (GET /booking/{{chainedBookingId}})**: Verify the server persisted the mutations.
6. **Step 6 (DELETE /booking/{{chainedBookingId}})**: Pass `Cookie: token={{authToken}}` $\rightarrow$ Remove resource from server (`201 Created`).
7. **Step 7 (GET /booking/{{chainedBookingId}})**: Query deleted ID $\rightarrow$ Verify `404 Not Found` to confirm complete teardown.

---

## 5. QA Test Strategy & Methodologies

### Q13: How did you design Positive vs. Negative testing?
**Answer**:
- **Positive Testing**: Valid payloads, correct data types, authenticated updates, and valid query filters expecting 200/201 responses.
- **Negative Testing**: Non-existent resource IDs (`999999999` $\rightarrow$ `404`), malformed IDs (`invalid-id` $\rightarrow$ `404`), invalid passwords (`reason: "Bad credentials"`), unauthenticated updates (`403 Forbidden`), and missing mandatory payload fields (`500` server exception defect).

### Q14: How did you approach Boundary Value Testing?
**Answer**: Rather than assuming theoretical boundaries, I tested empirical edge conditions:
1. `totalprice = 0`: Testing whether the server accepts zero pricing without arithmetic errors. The server accepted 0 and returned `totalprice: 0`.
2. String stress testing: Submitting strings exceeding 500 characters. The server handled the payload cleanly without truncation crashes.

### Q15: What is JSON Schema Validation and why is it superior to simple field checks?
**Answer**: Checking individual fields (`pm.expect(body.firstname).to.be.a("string")`) only validates fields you explicitly mention. JSON Schema Validation compares the entire JSON response against a formal schema contract (Draft-07). It validates:
- Presence of all required properties.
- Exact data types (string, number, boolean, object, array).
- Nested object structure (`bookingdates`).
- Disallowance of unexpected properties (`additionalProperties: false`).

---

## 6. Portfolio Differentiation: Project 1 vs. Project 2

### Q16: "How is this API Testing project different from your Playwright UI Automation project?"
**Answer**:
| Dimension | Project 1: Playwright + TypeScript (UI) | Project 2: Postman + Newman (API) |
| :--- | :--- | :--- |
| **Layer Under Test** | Presentation Layer (Frontend / DOM) | Service / Application Layer (Backend REST) |
| **Testing Mechanism** | Browser automation (Chromium, Firefox, WebKit) | Direct HTTP requests (GET, POST, PUT, DELETE) |
| **Verification Focus** | Visual elements, user clicks, forms, client routing | Status codes, response headers, schemas, server state |
| **Execution Speed** | Slower (requires browser startup & DOM rendering) | Ultra-fast (~11 seconds for 31 requests & 69 assertions) |
| **Failure Detection** | UI regressions, selector breakage, visual issues | Contract violations, backend unhandled errors, auth leaks |
| **CI Integration** | Browser runners, trace viewers, screenshots/video | Headless Newman CLI, visual HTML extra reports |
