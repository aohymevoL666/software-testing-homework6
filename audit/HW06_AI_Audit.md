# HW06 — AI Audit Report

**Student ID:** 23127531  
**Assignment:** HW06 — API Testing  
**AI tool:** ChatGPT  
**Model:** GPT-5.6 Sol  
**Primary usage dates:** 2026-09-02 to 2026-09-03  
**Local timezone:** UTC+07:00  
**Repository:** `https://github.com/aohymevoL666/software-testing-homework6`

> **Timestamp note:** The retained project transcript reliably preserves the interaction dates and order, but not an exact clock time for every individual AI exchange. Before final submission, exact times can be copied from the ChatGPT conversation UI if the instructor requires per-message clock timestamps. The audit below does not invent missing times.

## AI usage policy

AI was used for test ideation, audit support, Postman/Newman artifact generation, execution-result interpretation, CI/CD scaffolding, spreadsheet consolidation, and documentation drafting. AI output was not accepted blindly: generated cases were compared with the supplied API specification and inspected SUT implementation, then classified as VALID / INVALID / INCOMPLETE before execution.

## Interaction log

| # | Date | Time | User prompt / interaction summary | AI output / assistance | Human review / action |
|---:|---|---|---|---|---|
| 1 | 2026-09-02 | Not preserved | Establish HW06 scope, environment, and select one API from each pool. | Proposed and retained the baseline: `POST /api/register`, `POST /api/cart`, `GET /api/admin/users`; organized repository structure and execution plan. | Student accepted the three-API baseline and created the workspace. |
| 2 | 2026-09-02 | Not preserved | Inspect API 1 registration specification and implementation. | Identified request fields, response behavior, database schema, and missing validation constraints. | Student verified source and backend availability. |
| 3 | 2026-09-02 | Not preserved | Generate registration API tests with broad partition/security/schema coverage. | Generated 42 AI test cases in `api1_registration_ai_generated.md`. | Student saved and committed the AI-generated artifact. |
| 4 | 2026-09-02 | Not preserved | Audit all AI-generated registration cases against the actual specification/implementation. | Classified 42 cases as 11 VALID, 5 INVALID, 26 INCOMPLETE; explained fabricated validation/uniqueness assumptions. | Student saved and committed the audit. |
| 5 | 2026-09-02 | Not preserved | Extend API 1 coverage after the audit. | Proposed six additional candidate cases covering Content-Type handling, null body, duplicate/case-variant email, and mass assignment. | Student reviewed, accepted, saved, and committed the extension. |
| 6 | 2026-09-02 | Not preserved | Implement and execute API 1 in Postman/Newman. | Generated Postman collection/environment, added collection-level `X-Student-Id`, interpreted Newman results, and drafted execution/bug artifacts. | Student executed Newman; two 500 responses were confirmed and evidence was captured. |
| 7 | 2026-09-02 | Not preserved | Inspect API 2 shopping-cart implementation. | Identified bearer authentication, in-memory `userCarts`, and absence of payload/product validation. | Student confirmed source and retained FR-07 as API 2. |
| 8 | 2026-09-02 | Not preserved | Generate and audit API 2 cases. | Generated 42 cases, then audited them as 13 VALID, 2 INVALID, 27 INCOMPLETE. | Student saved and committed both artifacts. |
| 9 | 2026-09-02 | Not preserved | Extend API 2 after AI audit. | Proposed six candidate state/security cases: metadata inconsistency, user isolation, stale JWT characterization, nested payloads, extra-field persistence, and restart persistence. | Student reviewed, accepted, saved, and committed the extension. |
| 10 | 2026-09-02 | Not preserved | Implement API 2 Postman collection and execute it. | Generated 62-request collection plus environment; automated two-user setup and 47 logical cases, leaving restart persistence for manual execution. | Student ran Newman: 109/109 assertions passed. |
| 11 | 2026-09-03 | Not preserved | Complete manual cart restart-state test. | Diagnosed an invalid first attempt caused by an empty email and backend reseeding; then proposed a seeded-user rerun. | Student reran using `test@eshop.com`; cart contained marker before restart and `[]` after restart. Classified as characterization, not a spec defect. |
| 12 | 2026-09-03 | Not preserved | Inspect API 3 Admin users route and authorization middleware. | Found that `GET /api/admin/users` uses `authenticateToken` only and performs no Admin role check; identified response fields. | Student accepted this as the execution baseline. |
| 13 | 2026-09-03 | Not preserved | Generate API 3 access-control tests. | Generated 42 AI cases covering auth, authorization, role claims, stale state, privilege escalation, response schema, and protocol handling. | Student saved and committed the generation artifact. |
| 14 | 2026-09-03 | Not preserved | Audit all API 3 AI cases. | Classified 26 VALID and 16 INCOMPLETE; separated explicit Admin-only requirements from unsupported token-revocation assumptions. | Student saved and committed the audit. |
| 15 | 2026-09-03 | Not preserved | Extend API 3 coverage. | Proposed six candidate cases emphasizing direct non-admin access, end-to-end self-promotion, PII disclosure, wrong auth scheme, stale privilege, and response privacy. | Student reviewed, accepted, saved, and committed the extension. |
| 16 | 2026-09-03 | Not preserved | Implement and execute API 3 in Postman/Newman. | Generated 47-request collection; retained strict FR-12 assertions rather than forcing a green run. Interpreted 18 failures into three genuine bugs plus one oracle correction. | Student executed Newman and captured evidence. |
| 17 | 2026-09-03 | Not preserved | Draft API 3 bug reports. | Produced Markdown reports for broken Admin access control, Bearer-scheme enforcement, and vertical privilege escalation. | Student committed reports and created GitHub Issues #2–#4. |
| 18 | 2026-09-03 | Not preserved | Publish repository and document confirmed bugs as GitHub Issues. | Provided issue titles/bodies and a linking artifact. | Student created four public GitHub Issues and committed `docs/github_issues.md`. |
| 19 | 2026-09-03 | Not preserved | Add CI/CD evidence with one passing and one intentional failing test. | Generated GitHub Actions workflow, deterministic CI smoke collection/environment, and instructions for PASS → intentional FAIL → restore PASS. | Student produced green commit `170ab50`, failing commit `31fa261`, and restored passing state in `916e0ff`. |
| 20 | 2026-09-03 | Not preserved | Capture anti-cheat `X-Student-Id` evidence in Postman Console. | Gave step-by-step GUI instructions, diagnosed `{{baseurl}}` vs `{{baseUrl}}`, and verified the final screenshot. | Student captured and committed `evidence/postman_console_student_id.png`. |
| 21 | 2026-09-03 | Not preserved | Consolidate test cases and summary into the required spreadsheet artifact. | Generated `HW06_Test_Cases_and_Summary_23127531.xlsx` with Summary, three API sheets, and Bug Register. | Student saved and committed workbook in `402ac63`. |
| 22 | 2026-09-03 | Not preserved | Prepare mandatory AI audit and critique documentation. | Generated this audit and the separate 200–300 word critique. | Student must review wording and fill exact clock times only if required by the instructor. |

## Important human-audit examples

### Example 1 — API 1 validation assumptions

AI initially proposed expectations such as strict email-format validation, password rules, and duplicate-email rejection. Human/source review found that the supplied implementation and database schema did not define those constraints. Those cases were therefore marked INVALID or INCOMPLETE rather than being executed with fabricated oracles.

### Example 2 — API 2 product/cart semantics

AI initially assumed that a nonexistent product id should return 404 and that repeated adds should merge quantities. Source review showed that `POST /api/cart` simply pushes the request body into an in-memory array. These assumptions were rejected during human audit.

### Example 3 — API 3 authorization

The Admin-only rule was explicitly supported by the supplied API specification. Therefore normal-user denial cases remained VALID even though source inspection predicted they would fail. Execution later confirmed the defect: normal users received `200 OK` and the global user list.

## Disclosure about the “human extension” artifacts

AI assisted with proposing and formatting candidate extension cases after the audit. The student reviewed, selected, executed, and accepted the final cases. Because the assignment wording asks for student-added cases that AI missed, the safest academic-integrity approach is for the student to independently revise the wording or substitute at least five personally authored extension cases per API before final submission if the instructor interprets “of your own” as prohibiting AI-assisted ideation.

## Files produced with AI assistance

- `test-cases/api1_registration_ai_generated.md`
- `test-cases/api1_registration_audit.md`
- `test-cases/api1_registration_human_extension.md`
- `postman/HW06_API1_Registration.postman_collection.json`
- `docs/api1_registration_execution_summary.md`
- `reports/bugs/bug_api1_registration_content_type_500.md`
- `test-cases/api2_cart_ai_generated.md`
- `test-cases/api2_cart_audit.md`
- `test-cases/api2_cart_human_extension.md`
- `postman/HW06_API2_Cart.postman_collection.json`
- `docs/api2_cart_execution_summary.md`
- `test-cases/api3_access_control_ai_generated.md`
- `test-cases/api3_access_control_audit.md`
- `test-cases/api3_access_control_human_extension.md`
- `postman/HW06_API3_AccessControl.postman_collection.json`
- `docs/api3_access_control_execution_summary.md`
- three API 3 security bug reports
- `.github/workflows/api-tests.yml`
- `postman/HW06_CI_Smoke.postman_collection.json`
- `postman/HW06_CI.postman_environment.json`
- `docs/ci_cd_report.md`
- `test-cases/HW06_Test_Cases_and_Summary_23127531.xlsx`

## Human responsibility statement

The student remained responsible for running commands, inspecting the SUT, executing Newman/Postman, collecting screenshots, reviewing AI-generated cases, deciding which observed behaviors were genuine specification defects, publishing GitHub Issues, and committing/pushing the final artifacts.
