# 5-to-10 Minute Technical Demo Script & Walkthrough

This guide provides a structured, professional narrative to showcase this project during technical interviews, portfolio walkthroughs, or peer reviews.

---

## Demo Agenda (Total Duration: 8–10 Minutes)

| Section | Topic | Key Talking Points | Duration |
| :---: | :--- | :--- | :---: |
| **1** | Project & Problem Overview | Manual testing bottleneck, API vs UI layer | 1.0 min |
| **2** | Target Architecture & Tooling | Why Restful-Booker, Postman collection structure | 1.5 min |
| **3** | Deep-Dive into Postman Requests | Auth token extraction, JSON schema, security 403s | 2.5 min |
| **4** | The Critical Request Chaining Flow | 7-step stateful lifecycle demonstration | 2.0 min |
| **5** | Live Newman Run & HTML Report | Headless execution, failure behavior, report inspection | 1.5 min |
| **6** | CI/CD Pipeline & Portfolio Summary | GitHub Actions workflow, differentiation from Playwright | 1.5 min |

---

## Step-by-Step Demo Walkthrough Script

### 1. Introduce the Problem & Objective (1 min)
- *"In modern microservice architectures, validating backend services solely through frontend browser UI tests is inefficient and provides late feedback. When an API contract breaks, UI tests often fail with vague timeout errors.*
- *To solve this, I built an automated API Testing Suite using Postman, JavaScript, Newman, and GitHub Actions. This project validates HTTP behaviors, schema compliance, and authorization controls directly against backend REST endpoints."*

### 2. Show the Project Structure (1.5 min)
- Open VS Code or project root:
  - `collections/`: Houses the 10-folder Postman collection (`Restful_Booker_API_Testing_Suite.postman_collection.json`).
  - `environments/`: Contains environment variables (`QA_Environment.postman_environment.json`). Point out that secrets are sanitized and dynamic values are stored during runtime.
  - `schemas/`: Show JSON Schema Draft-07 schemas (`booking.schema.json`, `auth-token.schema.json`).
  - `docs/`: Highlight `Observed_API_Behavior.md`, `Defect_Log.md`, and `Test_Plan.md`.

### 3. Open Postman Desktop & Walk Through Key Scenarios (2.5 min)
- **Folder 02 (Authentication)**: Show `POST /auth`. Point out the test script:
  ```javascript
  var jsonData = pm.response.json();
  pm.environment.set("authToken", jsonData.token);
  ```
  *Explain: "We dynamically capture the session token into the environment so subsequent state-mutating requests can use it without manual copy-pasting."*
- **Folder 05 (Update PUT - Security)**: Show `PUT Update Booking - Without Auth Token`:
  - Show that sending the request without the token cookie strictly returns `403 Forbidden`.
  - *Explain: "This proves that our suite validates backend authorization enforcement, not just happy paths."*
- **Folder 08 (JSON Schema Validation)**: Show `Schema - Booking Detail Object`:
  - Show the schema assertion:
    ```javascript
    pm.response.to.have.jsonSchema(schema);
    ```
  - *Explain: "Instead of asserting individual fields one by one, JSON schema validation confirms that every required field, data type, and nested structure strictly conforms to the contractual specification."*

### 4. Demonstrate the Critical 7-Step Request Chaining Flow (2 min)
- Navigate to **Folder 10 (Critical End-to-End Request Chaining Workflow)**:
  - **Step 1**: Generates and extracts `authToken`.
  - **Step 2**: Creates a fresh booking via `POST` and dynamically captures `chainedBookingId`.
  - **Step 3**: Retrieves `GET /booking/{{chainedBookingId}}` to verify created state.
  - **Step 4**: Mutates total price and lastname via `PUT` with `Cookie: token={{authToken}}`.
  - **Step 5**: Retrieves via `GET` to prove that backend storage persisted the update.
  - **Step 6**: Deletes the booking via `DELETE` with `Cookie: token={{authToken}}`.
  - **Step 7**: Issues a final `GET` to verify `404 Not Found`.
  - *Explain: "This proves complete in-session CRUD persistence and guarantees zero test-data pollution through automated teardown."*

### 5. Execute Newman Live in Terminal & Open HTML Report (1.5 min)
- Open terminal and run:
  ```powershell
  npm test
  ```
- Highlight the real-time terminal output:
  - 31 requests executed across all 10 folders.
  - 69 assertions passing (0 failures).
  - Average latency ~290ms, total execution ~11 seconds.
- Open the generated HTML report:
  ```powershell
  npm run test:report
  ```
  (or open `reports/api-test-report.html` in browser).
  - Walk the interviewer through the dark theme dashboard, pass rate graphs, request/response headers, and full test script audit trails.

### 6. Show CI/CD Pipeline & Wrap Up (1.5 min)
- Open `.github/workflows/api-tests.yml`:
  - Point out that GitHub Actions triggers on every commit, installs dependencies, runs Newman without `--bail` so a full test report is produced, fails the build on any assertion failure, and archives the HTML report as an artifact.
- Deliver the Portfolio Differentiation closing statement:
  - *"In Project 1 (Playwright + TypeScript), I validated user journeys, Page Objects, and browser interactions. In Project 2, I demonstrated direct backend REST API automation, request chaining, JSON schemas, and command-line CI integration. Together, these demonstrate full-stack QA automation competence across both frontend and backend layers."*
