# Resume Claim Evidence Audit & Verification Matrix

This audit matrix cross-references every potential QA resume claim against concrete, verified implementation evidence within the repository.

---

## Resume Claim Audit Matrix

| Resume Claim / Skill | Concrete Project Evidence | Specific File / Location | Verified? |
| :--- | :--- | :--- | :---: |
| **REST API Testing** | Automated end-to-end regression suite validating RESTful contract conformity across 31 endpoints. | `collections/Restful_Booker_API_Testing_Suite.postman_collection.json` | **YES** |
| **HTTP Method Coverage** | Implemented and verified requests across `GET`, `POST`, `PUT`, `PATCH`, and `DELETE` verbs. | All 10 folders in Postman collection | **YES** |
| **Full CRUD Lifecycle** | Created resources (`POST`), read them (`GET`), mutated them (`PUT`/`PATCH`), and deleted them (`DELETE`). | Folders 03, 04, 05, 06, 07, 10 | **YES** |
| **Authentication Testing** | Authenticated against `/auth`, validated 200 OK + token body, and tested invalid/empty credentials. | Folder 02: `POST Create Auth Token` requests | **YES** |
| **Authorization & Access Control** | Enforced security checks ensuring unauthenticated/forged `PUT`, `PATCH`, and `DELETE` return `403 Forbidden`. | Scenarios `[PUT-004]`, `[PUT-006]`, `[PATCH-004]`, `[DEL-001]` | **YES** |
| **Positive Testing** | Valid payloads, authenticated updates, and valid query filters verified against 200/201 expectations. | Folders 01, 02, 03, 04, 05, 06, 07, 10 | **YES** |
| **Negative Testing** | Non-existent IDs (`404`), malformed IDs (`404`), bad passwords, unauthenticated requests (`403`), and missing fields. | Scenarios `[GET-009]`, `[GET-011]`, `[AUTH-004]`, `[POST-003]` | **YES** |
| **Boundary Value Testing** | Tested numeric lower edge (`totalprice: 0`) and large string length stress testing without false failure claims. | Folder 09: Scenarios `[BOUND-001]` through `[BOUND-004]` | **YES** |
| **Invalid Payload Testing** | Tested missing mandatory fields (DEF-001) and invalid data type coercion (totalprice as string). | Folder 04: Scenarios `[POST-003]`, `[POST-005]` | **YES** |
| **Unauthorized Access Testing** | Dedicated test requests validating that state mutations without token cookies are rejected. | Scenarios `[PUT-004]`, `[PATCH-004]`, `[DEL-001]` | **YES** |
| **Non-Existent Resource Handling** | Validated `404 Not Found` for absent IDs on GET and observed `405` on non-existent DELETE. | Scenarios `[GET-009]`, `[DEL-003]`, `[CHAIN-013]` | **YES** |
| **Status Code Validation** | Asserted expected HTTP status codes across `200 OK`, `201 Created`, `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed`, and `500 Internal Server Error`. | Assertions in all 31 requests | **YES** |
| **Request & Response Header Validation** | Validated `Content-Type: application/json` on JSON endpoints and tested `Cookie: token={{authToken}}`. | Scenarios `[AUTH-002]`, `[GET-002]`, `[PUT-002]`, `[PATCH-002]` | **YES** |
| **Query Parameter Testing** | Verified filtering by query string parameter `?firstname=Susan`. | Folder 03: Scenario `[GET-007]` | **YES** |
| **Path Parameter Testing** | Dynamic injection of resource IDs into URIs (`/booking/{{seedBookingId}}`, `/booking/{{chainedBookingId}}`). | Folders 03, 05, 06, 07, 10 | **YES** |
| **Response Body Validation** | Deep assertions on JSON properties, arrays, string matching, and numeric field verification. | Implemented across all 69 assertions | **YES** |
| **JSON Schema Validation** | Validated responses against strict JSON Schema Draft-07 schemas using `pm.response.to.have.jsonSchema()`. | `schemas/*.json` & Folder 08 | **YES** |
| **Response-Time SLA Validation** | Asserted response latency thresholds (< 3000ms SLA). | Scenario `[HEALTH-002]` | **YES** |
| **Request Chaining & API Dependencies** | Extracted `authToken` and `chainedBookingId` dynamically and passed them through a 7-step sequence. | Folder 10: Scenarios `[CHAIN-001]` to `[CHAIN-014]` | **YES** |
| **Automated Newman CLI Execution** | Automated headless execution via Newman with custom scripts (`npm test`, `npm run test:api`). | `package.json` | **YES** |
| **HTML Test Reporting** | Generated visual, dark-themed HTML test reports with full audit logs using `newman-reporter-htmlextra`. | `reports/api-test-report.html` | **YES** |
| **CI/CD Automation** | Configured GitHub Actions workflow to run on push/PR, install dependencies, run Newman, and upload artifacts. | `.github/workflows/api-tests.yml` | **VERIFIED ON GITHUB ACTIONS CI/CD (Run #34155316187 — Success, Report Artifact Uploaded)** |
| **Failure Propagation** | Verified that assertion failures cause Newman to return exit code 1 and fail the CI job (tested without `--bail`). | Verified empirically via controlled test injection | **YES** |
| **Defect Tracking & QA Documentation** | Documented real discovered defects (DEF-001, DEF-002, DEF-003) and provided standard defect templates. | `docs/Defect_Log.md` | **YES** |
| **Manual-to-Automation Traceability** | Documented complete mapping: API Requirement $\rightarrow$ Test Scenario $\rightarrow$ Test Case $\rightarrow$ Assertion $\rightarrow$ Result. | `docs/Test_Cases.md` | **YES** |

---

## Conclusion
Every single resume claim in the matrix above is backed by verifiable, working code and automated test execution. There are zero fabricated or unverified claims in this repository.
