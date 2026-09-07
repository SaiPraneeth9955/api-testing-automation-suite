# API Test Automation Plan: Restful-Booker Suite

## 1. Project Objective
The objective of this project is to implement an enterprise-standard, automated REST API regression and functional test suite for the `Restful-Booker` hotel booking platform. The suite provides deterministic validation of backend RESTful behaviors, request/response headers, schema conformity, authentication gating, stateful CRUD workflows, and CI/CD integration.

This project serves as **Project 2** in the QA Portfolio, complementing UI browser automation (Project 1: Playwright + TypeScript) by focusing strictly on direct HTTP protocol communication, payload validation, and server-side logic without browser overhead.

---

## 2. Testing Scope

### In-Scope:
- Service health check & latency validation (`GET /ping`).
- Token-based authentication, credential verification, and token extraction (`POST /auth`).
- Booking inventory exploration via primary keys and query parameter filters (`GET /booking`, `GET /booking/:id`).
- Resource creation with full payload validation (`POST /booking`).
- Full mutation via PUT and partial mutation via PATCH using authorization session cookies (`Cookie: token=<token>`).
- Security validation: access control enforcement on PUT, PATCH, and DELETE when tokens are missing or forged (`403 Forbidden`).
- Resource deletion and lifecycle teardown (`DELETE /booking/:id`).
- JSON schema validation against formal JSON Schema Draft-07 definitions.
- Boundary condition evaluation (zero-pricing, large payload strings).
- 7-step critical end-to-end request chaining workflow with state verification.
- Automated CLI execution via Newman and dark-themed HTML reporting (`newman-reporter-htmlextra`).
- CI/CD execution triggered via GitHub Actions on push and pull requests.

### Out-of-Scope:
- User interface (UI) rendering and DOM element interactions (covered in Project 1).
- High-volume stress and distributed load testing (e.g., JMeter, k6).
- Database internal replication testing (black-box API testing only).
- Non-REST protocols (e.g., GraphQL, gRPC, WebSockets).

---

## 3. APIs and Endpoints Under Test
- **Base URL**: `https://restful-booker.herokuapp.com`
- **Endpoints**:
  - `GET /ping` — Service Healthcheck
  - `POST /auth` — Session Authentication
  - `GET /booking` — Booking Catalog
  - `GET /booking/:id` — Single Booking Detail
  - `POST /booking` — Create Booking
  - `PUT /booking/:id` — Full Booking Mutation
  - `PATCH /booking/:id` — Partial Booking Mutation
  - `DELETE /booking/:id` — Booking Deletion

---

## 4. Testing Types Conducted
1. **Functional Testing**: Validating status codes, business payload values, and required fields.
2. **Negative Testing**: Validating invalid IDs, non-existent endpoints, empty payloads, and missing fields.
3. **Security & Authorization Testing**: Ensuring state-mutating requests (PUT, PATCH, DELETE) fail with `403 Forbidden` without a valid session token.
4. **JSON Schema Testing**: Validating strict structural and data-type compliance against draft-07 schemas.
5. **Boundary Value & Robustness Testing**: Testing numeric edge inputs (`totalprice: 0`) and long string stress scenarios.
6. **Stateful Chaining / End-to-End Workflow Testing**: Validating persistent lifecycle: Auth -> Create -> Capture ID -> Retrieve -> Mutate -> Verify -> Delete -> Verify 404.
7. **Performance SLA Validation**: Verifying that API response latencies adhere to a baseline SLA (< 3000ms).

---

## 5. Test Approach & Architecture
1. **Empirical Exploration**: Endpoints are probed using Node.js to verify actual runtime behavior before writing test scripts.
2. **Postman Collection (v2.1.0)**: Structured modularly into 10 cohesive test folders.
3. **JavaScript Assertions**: Utilizing Postman's built-in Chai assertion library (`pm.test`, `pm.expect`, `pm.response.to.have.status`, `pm.response.to.have.jsonSchema`).
4. **Dynamic Environment Variable Management**:
   - `baseUrl`: Centralized target endpoint.
   - `authToken`: Extracted dynamically during authentication.
   - `seedBookingId`: Pre-existing booking ID captured for independent GET/read tests.
   - `chainedBookingId`: Created, tracked, mutated, and deleted exclusively in the end-to-end workflow.
5. **Newman CLI Automation**: Running tests headlessly with console logging and full HTML report generation.
6. **Failure Propagation Policy**: Suite executes completely without `--bail` to provide exhaustive coverage data, returning a non-zero exit code on failure to block CI pipelines.

---

## 6. Entry and Exit Criteria

### Entry Criteria:
- Node.js LTS (v24.x) and npm (11.x) installed and operational.
- Postman Desktop and Newman CLI operational locally.
- Target API `https://restful-booker.herokuapp.com/ping` returns `201 Created`.
- Test collection and environment files structurally and syntactically validated.

### Exit Criteria:
- 100% of planned test requests (31/31) executed.
- 100% of planned assertions (69/69) passing successfully.
- Zero unexplained test failures.
- Failure propagation verified via a controlled test run.
- Comprehensive HTML report generated and verified in `reports/api-test-report.html`.
- CI/CD workflow defined, validated, and ready for deployment.

---

## 7. Test Environment & Data Strategy
- **Environment**: Staging / Public Heroku Cloud Sandbox (`https://restful-booker.herokuapp.com`).
- **Data Strategy**:
  - `data/test-data.json` houses structured payloads for valid creation, updates, and negative test inputs.
  - Runtime isolation: The suite creates dedicated dynamic resources for test execution and removes them in teardown steps.

---

## 8. Tools and Technologies
| Tool / Technology | Version | Purpose |
| :--- | :---: | :--- |
| **Postman Desktop** | 12.27.0 | Test case authoring, interactive debugging, collection management |
| **JavaScript (ES6)** | Built-in | Postman test scripts, assertions, schema validation, and dynamic variable storage |
| **Newman** | 6.2.2 | Headless command-line collection execution |
| **newman-reporter-htmlextra** | 1.23.1 | Visual dark-themed HTML test reporting with full request/response audit trails |
| **Git** | 2.48.1 | Source control and structured versioning |
| **GitHub Actions** | CI/CD | Continuous integration pipeline triggered on push/PR |

---

## 9. Risks and Mitigations
1. **Public Sandbox State Resets**:
   - *Risk*: Heroku dyno restarts or shared environment resets could disrupt pre-existing IDs.
   - *Mitigation*: The suite dynamically discovers existing IDs on each run (`GET /booking` sets `seedBookingId`) and creates dedicated resources for chaining tests (`chainedBookingId`).
2. **Third-Party Network Latency**:
   - *Risk*: Network fluctuations over the internet could trigger false positive latency failures.
   - *Mitigation*: The latency threshold is set to a realistic testing SLA (< 3000ms) rather than an overly strict micro-benchmark.
3. **Public Shared Usage**:
   - *Risk*: Concurrent users creating or modifying bookings simultaneously.
   - *Mitigation*: The suite tests mutations against its own dynamically created IDs (`chainedBookingId`), ensuring test isolation.
