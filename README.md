# HW06 — AI-Assisted API Testing

**Student ID:** 23127531  
**Repository:** https://github.com/aohymevoL666/software-testing-homework6  
**SUT:** eShop backend API  
**Base URL:** `http://localhost:3000`

## Selected APIs

| API | Feature | Endpoint |
|---|---|---|
| API 1 | FR-01 Account Registration | `POST /api/register` |
| API 2 | FR-07 Shopping Cart | `POST /api/cart` |
| API 3 | FR-12 Access Control | `GET /api/admin/users` |

## Test Summary

| Metric | API 1 | API 2 | API 3 | Overall |
|---|---:|---:|---:|---:|
| AI-generated cases | 42 | 42 | 42 | 126 |
| Human-added cases | 6 | 6 | 6 | 18 |
| Total logical cases | 48 | 48 | 48 | 144 |
| Logical cases executed | 48 | 48 | 35 | 131 |
| Logical PASS | 46 | 48 | 27 | 121 |
| Logical FAIL | 2 | 0 | 8 | 10 |
| Not executed | 0 | 0 | 13 | 13 |
| Confirmed bugs | 1 | 0 | 3 | 4 |

## Confirmed Bugs

1. Registration unsupported/missing `Content-Type` causes HTTP 500  
   GitHub Issue: https://github.com/aohymevoL666/software-testing-homework6/issues/1
2. Non-admin user can access `GET /api/admin/users`  
   GitHub Issue: https://github.com/aohymevoL666/software-testing-homework6/issues/2
3. Valid JWT is accepted under `Basic` instead of the documented `Bearer` scheme  
   GitHub Issue: https://github.com/aohymevoL666/software-testing-homework6/issues/3
4. Normal user can self-assign Admin role through `PUT /api/users/me`  
   GitHub Issue: https://github.com/aohymevoL666/software-testing-homework6/issues/4

## Postman Features Used

- Collections
- Environments
- Environment variables
- Dynamic runtime variables
- Collection-level pre-request scripts
- `X-Student-Id` header injection
- Postman test scripts / assertions
- Multi-request setup and state-validation flows
- Postman Console
- Newman CLI execution
- Newman HTML reports
- GitHub Actions integration

## CI/CD Evidence

- PASS commit: `170ab50` — `ci: add passing API test workflow`
- Intentional FAIL commit: `31fa261` — `ci: demonstrate failing API assertion`
- Final restored PASS state: `916e0ff`
- Screenshots: `evidence/ci_pass_run.png`, `evidence/ci_fail_run.png`

## Agent Skill

The submission includes an AI-driven API test-generator design:

- `agent-skill/HW06_API_Test_Generator_Design.md`
- `agent-skill/api_test_generator_pseudocode.txt`
- `agent-skill/api_test_generator_diagram.png` — self-drawn

## Self-Assessment

The following is a **provisional** self-assessment. It can be adjusted before creating the final ZIP.

| Criterion | Maximum | Self-Assessed |
|---|---:|---:|
| API 1 — Generate + Audit + Extend + Execute + Bugs | 30 | 29 |
| API 2 — Generate + Audit + Extend + Execute + Bugs | 30 | 29 |
| API 3 — Generate + Audit + Extend + Execute + Bugs | 30 | 27 |
| Agent Skill — AI-driven API test generator | 10 | 10 |
| **Total** | **100** | **95** |

The deductions reflect conservative self-assessment for incomplete execution of 13 retained API 3 cases and minor documentation limitations such as exact AI interaction clock times not being preserved in the transcript.

## Main Submission Artifacts

- Main report: `report/HW06_Report.md` and PDF version
- AI audit: `audit/HW06_AI_Audit.md` and PDF version
- AI critique: `audit/HW06_AI_Critique.md`
- Test workbook: `test-cases/HW06_Test_Cases_and_Summary_23127531.xlsx`
- Postman collections/environments: `postman/`
- Newman HTML reports: `reports/newman/`
- Bug reports: `reports/bugs/`
- CI/CD report: `docs/ci_cd_report.md`
- GitHub Issues: `docs/github_issues.md`
- Agent Skill: `agent-skill/`
- Execution evidence: `evidence/`
- Git commit log: `git-commit-log.txt`
