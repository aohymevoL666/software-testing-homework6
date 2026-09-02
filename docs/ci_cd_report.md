# HW06 — CI/CD Report

**Repository:** `https://github.com/aohymevoL666/software-testing-homework6`  
**Workflow:** `HW06 API Tests`  
**Workflow file:** `.github/workflows/api-tests.yml`

## Objective

Demonstrate automated API testing in GitHub Actions with two required sample outcomes:

1. one commit/run where the API tests pass;
2. one commit/run where a deliberately incorrect assertion is detected and the workflow fails.

The workflow starts the eShop SUT on the GitHub-hosted runner, waits for `http://localhost:3000/api/products`, installs Newman, executes the real API 2 regression collection, and then executes a deterministic CI smoke assertion.

## Sample Run 1 — PASS

- **Commit:** `170ab50`
- **Commit message:** `ci: add passing API test workflow`
- **CI smoke expectation:** HTTP `200`
- **Observed workflow result:** **PASS**
- **Observed duration:** approximately **28 seconds**
- **Evidence:** `evidence/ci_pass_run.png`
- **GitHub Actions run URL:** `PASTE_PASS_RUN_URL`

This run demonstrates that the SUT can be started in GitHub Actions and that the API regression/smoke tests execute successfully.

## Sample Run 2 — Intentional FAIL

- **Commit:** `31fa261`
- **Commit message:** `ci: demonstrate failing API assertion`
- **Intentional change:** CI smoke expected status changed from `200` to `201`
- **Actual API status:** `200`
- **Observed workflow result:** **FAIL**
- **Observed duration:** approximately **24 seconds**
- **Evidence:** `evidence/ci_fail_run.png`
- **GitHub Actions run URL:** `PASTE_FAIL_RUN_URL`

The second run intentionally changes only the test oracle. The SUT itself is not modified. The purpose is to show that the CI pipeline correctly reports a regression/test failure when an assertion does not match the observed API response.

## Difference between the two sample commits

```diff
"key": "expectedStatus",
-"value": "200",
+"value": "201",
"enabled": true
```

No application behavior was intentionally changed between these two sample runs.

## Final repository state

After collecting the required PASS and intentional-FAIL evidence, the CI smoke expectation should be restored to `200` so the submitted repository finishes in a healthy passing configuration.

## Evidence checklist

| Evidence | Status |
|---|---|
| Passing GitHub Actions run | Complete |
| Intentional failing GitHub Actions run | Complete |
| Passing-run screenshot | Complete |
| Failing-run screenshot | Complete |
| PASS commit hash | `170ab50` |
| FAIL commit hash | `31fa261` |
| Exact run URLs | Add before final submission |
