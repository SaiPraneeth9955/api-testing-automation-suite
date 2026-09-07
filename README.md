# 🚀 Restful-Booker API Testing Automation Suite

[![Newman API Tests](https://img.shields.io/badge/Newman-v6.2.2-orange.svg)](https://www.npmjs.com/package/newman)
[![Reporter](https://img.shields.io/badge/Reporter-htmlextra-blue.svg)](https://www.npmjs.com/package/newman-reporter-htmlextra)
[![Postman](https://img.shields.io/badge/Postman-v12.27.0-FF6C37.svg)](https://www.postman.com/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF.svg)](https://github.com/features/actions)
[![Pass Rate](https://img.shields.io/badge/Tests-100%25%20Passing-brightgreen.svg)]()

An enterprise-grade, automated REST API test suite engineered with **Postman**, **JavaScript**, **Newman**, and **GitHub Actions CI/CD**.

This is **Project 2** in my Quality Assurance Automation portfolio. It directly complements **Project 1 (Playwright + TypeScript Web UI Framework)** by focusing on backend service contracts, direct HTTP protocol validation, authorization gating, JSON schemas, and stateful request chaining without browser UI dependencies.

---

## 📑 Table of Contents
1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Why This Project Was Built](#3-why-this-project-was-built)
4. [Project Objectives](#4-project-objectives)
5. [Technology Stack & Rationale](#5-technology-stack--rationale)
6. [API Under Test](#6-api-under-test)
7. [Project Directory Structure](#7-project-directory-structure)
8. [HTTP & REST Testing](#8-http--rest-testing)
9. [Authentication & Authorization Testing](#9-authentication--authorization-testing)
10. [CRUD Lifecycle Testing](#10-crud-lifecycle-testing)
11. [Positive Testing](#11-positive-testing)
12. [Negative Testing](#12-negative-testing)
13. [Boundary & Stress Testing](#13-boundary--stress-testing)
14. [JSON Schema Validation](#14-json-schema-validation)
15. [Response-Time & SLA Validation](#15-response-time--sla-validation)
16. [Request Chaining & Dynamic Variables](#16-request-chaining--dynamic-variables)
17. [API Dependencies](#17-api-dependencies)
18. [Critical End-to-End API Workflow](#18-critical-end-to-end-api-workflow)
19. [Test Data Management](#19-test-data-management)
20. [Running in Postman Desktop](#20-running-in-postman-desktop)
21. [Running with Newman CLI](#21-running-with-newman-cli)
22. [HTML Test Reports](#22-html-test-reports)
23. [CI/CD Pipeline (GitHub Actions)](#23-cicd-pipeline-github-actions)
24. [Defect Tracking & Discoveries](#24-defect-tracking--discoveries)
25. [API Limitations](#25-api-limitations)
26. [Risk Management](#26-risk-management)
27. [Tool Versions & Environment Matrix](#27-tool-versions--environment-matrix)
28. [Quick Start Guide](#28-quick-start-guide)
29. [Reproducibility Guarantee](#29-reproducibility-guarantee)
30. [Key QA Learnings](#30-key-qa-learnings)
31. [Future Improvements](#31-future-improvements)

---

## 1. Project Overview
The **Restful-Booker API Testing Automation Suite** is an automated testing solution that validates the endpoints of the `Restful-Booker` hotel booking API.

The suite contains **10 cohesive folders**, **31 automated HTTP requests**, and **69 robust JavaScript assertions**, executing headlessly in ~11.7 seconds with a 100% pass rate.

---

## 2. Problem Statement
Manual API validation becomes repetitive, slow, and prone to human oversight as endpoint permutations grow. Testing APIs purely through a browser UI is brittle and slow, as UI failures often mask underlying backend contract changes. 

This project solves that challenge by providing an automated API regression harness that executes directly against backend HTTP endpoints, validating status codes, headers, response schemas, and stateful data persistence before code reaches production.

---

## 3. Why This Project Was Built
As part of my QA portfolio, this project demonstrates backend automation skills that are distinct from browser-based UI automation:
- **Project 1 (Playwright + TypeScript)**: Validates the frontend presentation layer, user journeys, Page Object Models, and DOM elements.
- **Project 2 (Postman + Newman)**: Validates the backend application layer, HTTP methods, headers, query/path parameters, authorization tokens, JSON schemas, and API request chaining.

---

## 4. Project Objectives
- Build a modular, 100% runnable Postman collection with zero hardcoded dynamic IDs.
- Validate the full CRUD lifecycle against an API with genuine in-session persistence.
- Implement strict JSON schema validation using JSON Schema Draft-07 standards.
- Enforce authorization controls on state-mutating requests (`403 Forbidden` verification).
- Build a 7-step critical end-to-end request chaining workflow.
- Automate execution using Newman CLI and generate rich visual HTML reports.
- Implement GitHub Actions CI/CD with automatic non-zero failure propagation.

---

## 5. Technology Stack & Rationale

| Technology | Selected Tool | Why It Was Chosen |
| :--- | :--- | :--- |
| **API Exploration & Authoring** | **Postman v12.27.0** | Industry standard for designing, parameterizing, and organizing API collections. |
| **Test Scripting** | **JavaScript (ES6) & Chai** | Native Postman scripting engine; allows expressive status, header, and body assertions. |
| **Schema Validation** | **JSON Schema Draft-07 (tv4/Ajv)** | Enables structural contract testing, ensuring API responses maintain required types. |
| **CLI Test Runner** | **Newman v6.2.2** | Official Postman CLI runner; executes collections headlessly inside CI/CD environments. |
| **HTML Reporting** | **newman-reporter-htmlextra v1.23.1** | Produces interactive, dark-themed HTML reports containing full request/response logs. |
| **CI/CD Orchestration** | **GitHub Actions** | Automates test execution on push and pull requests with artifact retention. |
| **Version Control** | **Git & GitHub** | Tracks changes, tags releases, and maintains strict version control. |

---

## 6. API Under Test
- **Name**: Restful-Booker
- **Base URL**: `https://restful-booker.herokuapp.com`
- **Architecture**: RESTful HTTP API over JSON / plain-text
- **Key Capabilities**:
  - Genuine in-session CRUD persistence (POST creates an actual record; GET retrieves it; PUT modifies it; DELETE removes it; subsequent GET returns 404).
  - Cookie-based token authentication (`POST /auth` $\rightarrow$ `Cookie: token=<token>`).
  - Authorization gating on mutating endpoints (`PUT`, `PATCH`, `DELETE`).

---

## 7. Project Directory Structure
```
api-testing-suite/
├── .github/
│   └── workflows/
│       └── api-tests.yml                       # GitHub Actions CI/CD configuration
├── collections/
│   └── Restful_Booker_API_Testing_Suite.postman_collection.json  # 31 requests, 10 folders
├── environments/
│   ├── QA_Environment.postman_environment.json # Runtime variables (baseUrl, authToken, IDs)
│   └── QA_Environment.example.postman_environment.json # Configuration template
├── schemas/
│   ├── auth-token.schema.json                  # Draft-07 schema for auth success
│   ├── auth-error.schema.json                  # Draft-07 schema for auth bad creds
│   ├── booking.schema.json                     # Draft-07 schema for booking detail
│   └── booking-summary.schema.json             # Draft-07 schema for booking catalog array
├── data/
│   └── test-data.json                          # Structured payloads and edge test inputs
├── reports/
│   ├── .gitkeep                                # Git placeholder
│   └── api-test-report.html                    # Generated Newman htmlextra report
├── docs/
│   ├── Observed_API_Behavior.md                # Empirical discovery log & test rationale
│   ├── Test_Plan.md                            # Complete QA strategy document
│   ├── Test_Scenarios.md                       # Comprehensive test scenario index
│   ├── Test_Cases.md                           # Detailed test cases with traceability
│   ├── Defect_Log.md                           # Discovered defects & standard template
│   ├── QA_Summary_Report.md                    # Actual verified execution metrics
│   ├── Interview_Explanation.md                # Technical interview Q&A defense
│   ├── Demo_Guide.md                           # 5-10 minute presentation script
│   └── Resume_Evidence.md                      # Audit matrix of all portfolio claims
├── package.json                                # Node scripts and devDependencies
├── README.md                                   # Master documentation
└── .gitignore                                  # Secrets, reports, and node_modules ignore
```

---

## 8. HTTP & REST Testing
The suite exercises every standard HTTP verb and validates status code contracts:
- `GET`: Safe reads (`200 OK`, `404 Not Found`).
- `POST`: Resource creation and token issuance (`200 OK`).
- `PUT`: Full payload replacement (`200 OK`, `403 Forbidden`).
- `PATCH`: Partial field mutation (`200 OK`, `403 Forbidden`).
- `DELETE`: Resource removal (`201 Created`, `403 Forbidden`, `405 Method Not Allowed`).

---

## 9. Authentication & Authorization Testing
- **Valid Authentication**: `POST /auth` validates administrative credentials, verifies 200 OK and JSON structure, and dynamically stores `jsonData.token` as `authToken`.
- **Invalid Authentication**: Tests bad credentials (`reason: "Bad credentials"`).
- **Authorization Gating**: Dedicated security tests confirm that `PUT`, `PATCH`, and `DELETE` requests submitted without an authentication cookie return `403 Forbidden`.
- **Forged Token Testing**: Confirms that submitting an invalid token (`token=bad_token`) is rejected with `403 Forbidden`.

---

## 10. CRUD Lifecycle Testing
The suite tests genuine CRUD state management:
1. **Create**: `POST /booking` adds a new record and returns a generated `bookingid`.
2. **Read**: `GET /booking/:id` retrieves the record and validates field accuracy.
3. **Update**: `PUT /booking/:id` and `PATCH /booking/:id` modify fields in the database.
4. **Delete**: `DELETE /booking/:id` removes the record.
5. **Teardown Confirmation**: Subsequent `GET /booking/:id` returns `404 Not Found`.

---

## 11. Positive Testing
Validates standard business flows:
- Healthy ping responses (`201 Created`).
- Valid credentials issuance (`200 OK`).
- Query filtering (`GET /booking?firstname=Susan`).
- Accurate payload echoing on resource creation.

---

## 12. Negative Testing
Validates application resilience:
- Non-existent resource IDs (`999999999` $\rightarrow$ `404 Not Found`).
- Malformed alphanumeric IDs (`invalid-id` $\rightarrow$ `404 Not Found`).
- Missing required fields in booking creation (`500 Internal Server Error` logged as defect DEF-001).
- Unauthenticated mutation attempts (`403 Forbidden`).

---

## 13. Boundary & Stress Testing
Evaluates empirical edge conditions:
- **Numeric Boundary (`totalprice: 0`)**: Validates that zero pricing is processed without arithmetic errors and correctly reflected in the response.
- **String Stress Testing**: Submits strings exceeding 500 characters to verify backend buffer handling.

---

## 14. JSON Schema Validation
Using JSON Schema Draft-07 definitions, tests validate:
- Type safety (strings, integers, booleans, objects, arrays).
- Mandatory presence of required keys.
- Nested structures (`bookingdates.checkin`, `bookingdates.checkout`).
- Strict rejection of unexpected properties (`additionalProperties: false`).

---

## 15. Response-Time & SLA Validation
- Tests assert response duration thresholds (`pm.expect(pm.response.responseTime).to.be.below(3000)`).
- Documented as a functional testing SLA threshold rather than a micro-benchmarking performance test.

---

## 16. Request Chaining & Dynamic Variables
Request chaining eliminates manual value passing:
```
POST /auth (Extract jsonData.token -> authToken)
       ↓
POST /booking (Extract jsonData.bookingid -> chainedBookingId)
       ↓
GET /booking/{{chainedBookingId}} (Validate created state)
       ↓
PUT /booking/{{chainedBookingId}} [Cookie: token={{authToken}}] (Mutate values)
       ↓
GET /booking/{{chainedBookingId}} (Validate mutated state)
       ↓
DELETE /booking/{{chainedBookingId}} [Cookie: token={{authToken}}] (Teardown)
       ↓
GET /booking/{{chainedBookingId}} (Confirm 404 Not Found)
```

---

## 17. API Dependencies
- Independent exploratory tests utilize `seedBookingId` (dynamically discovered via `GET /booking`).
- Mutation tests strictly depend on `authToken`.
- The End-to-End workflow is completely self-contained, creating and tearing down its own resources via `chainedBookingId`.

---

## 18. Critical End-to-End API Workflow
Folder 10 implements the 7-step critical workflow. It validates the full persistence lifecycle of a single entity and concludes with verified 404 teardown, guaranteeing zero state pollution in the public environment.

---

## 19. Test Data Management
- `data/test-data.json` houses structured payloads for standard bookings, full updates, partial updates, and boundary inputs.
- Test data is utilized intentionally where data variation adds tangible coverage value.

---

## 20. Running in Postman Desktop
1. Launch Postman Desktop.
2. Click **Import** $\rightarrow$ select `collections/Restful_Booker_API_Testing_Suite.postman_collection.json`.
3. Click **Import** $\rightarrow$ select `environments/QA_Environment.postman_environment.json`.
4. Select `QA_Environment` in the top-right environment dropdown.
5. Click **Run Collection** to execute all folders interactively.

---

## 21. Running with Newman CLI
Run the entire automated suite directly from the command line:

```powershell
# Install dependencies
npm install

# Run the complete test suite and generate HTML report
npm test

# Run with console terminal output only
npm run test:cli
```

---

## 22. HTML Test Reports
The suite uses `newman-reporter-htmlextra` to generate dark-themed test dashboards located at:
```
reports/api-test-report.html
```
To view the report in your default browser:
```powershell
npm run test:report
```
The report includes pass/fail metrics, visual charts, complete request headers, payload bodies, and test script assertions.

---

## 23. CI/CD Pipeline (GitHub Actions)
The workflow file `.github/workflows/api-tests.yml` executes on every push and pull request to `main`/`master`:
1. Checks out repository code.
2. Sets up Node.js v20 with npm caching.
3. Installs dependencies via `npm ci`.
4. Runs Newman without `--bail` so all 31 requests and 69 assertions execute.
5. Automatically marks the build as failed if any assertion fails.
6. Archives `reports/api-test-report.html` as a downloadable artifact.

---

## 24. Defect Tracking & Discoveries
Full details are recorded in [`docs/Defect_Log.md`](file:///docs/Defect_Log.md):
- **DEF-001**: `POST /booking` returns `500 Internal Server Error` instead of `400 Bad Request` when required fields are missing.
- **DEF-002**: `POST /auth` returns `200 OK` with `{"reason":"Bad credentials"}` instead of `401 Unauthorized`.
- **DEF-003**: `DELETE /booking/:id` returns `405 Method Not Allowed` when deleting a non-existent ID.

---

## 25. API Limitations
- **Shared In-Memory Persistence**: Restful-Booker runs in a shared Heroku sandbox; database records reset every ~10–15 minutes. The suite is architected to be resilient by dynamically generating new resources on each execution.
- **Plain Text Error Payloads**: Non-2xx responses return plain text strings rather than structured JSON.

---

## 26. Risk Management
- **Third-Party Uptime**: The suite incorporates healthcheck assertions and defensive timeout allowances.
- **Concurrent Test Runs**: Isolation is maintained by generating dynamic resources per iteration.

---

## 27. Tool Versions & Environment Matrix
- **Node.js**: `v24.20.0`
- **npm**: `11.19.0`
- **Git**: `2.48.1.windows.1`
- **Postman**: `12.27.0`
- **Newman**: `6.2.2`
- **newman-reporter-htmlextra**: `1.23.1`

---

## 28. Quick Start Guide
```powershell
# 1. Clone repository
git clone <your-repository-url>
cd "API Testing Automation Suite"

# 2. Install dependencies
npm install

# 3. Execute automated test suite
npm test

# 4. Open generated HTML report
npm run test:report
```

---

## 29. Reproducibility Guarantee
This project was built and verified from scratch on a clean environment. All commands, paths, and package configurations are verified and operate deterministically across fresh clones.

---

## 30. Key QA Learnings
- **Empirical observation trumps assumptions**: Testing actual API endpoints first prevents creating incorrect assertions based on outdated documentation.
- **No `--bail` in CI**: Allowing Newman to run completely provides full visibility across all modules even when individual assertions fail.
- **Request Chaining Architecture**: Proper state isolation prevents tests from interfering with one another in shared test environments.

---

## 31. Future Improvements
- Parameterized Newman execution across multiple environments (Dev, Staging, Prod).
- Mock service simulation with Prism for offline CI execution.
- Slack / Microsoft Teams webhook notification integration on CI failure.
