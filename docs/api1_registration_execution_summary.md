# HW06 — API 1 Execution Summary

**Feature:** FR-01 Account Registration  
**Endpoint:** `POST /api/register`  
**Tool:** Postman Collection executed with Newman 6.2.2  
**Environment:** `http://localhost:3000`  
**Student header:** `X-Student-Id: 23127531`

## Run summary

| Metric | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 53 | 0 |
| Test scripts | 51 | 0 |
| Pre-request scripts | 51 | 0 |
| Assertions | 134 | 2 |

- Total duration: **4.6 s**
- Average response time: **6 ms**
- Minimum response time: **2 ms**
- Maximum response time: **56 ms**
- Failed assertions: **2**
- Both failures belong to the same root-cause area: request parsing / unsupported or missing `Content-Type`.

> The collection contains 51 request items. Newman reports 53 executed requests because two security tests use `pm.sendRequest()` to perform helper login verification.

## Failed tests

### REG-HUM-001 — `Content-Type: text/plain`

**Request:** `POST /api/register` with a JSON-looking body but `Content-Type: text/plain`.

**Expected test property:** The server should reject or otherwise handle the unsupported media type safely and must not return a server-side 5xx error.

**Actual:** `500 Internal Server Error`.

**Result:** FAIL.

### REG-HUM-002 — missing `Content-Type`

**Request:** `POST /api/register` with a raw JSON-looking body but no `Content-Type` header.

**Expected test property:** The server should reject or otherwise handle the request safely and must not return a server-side 5xx error.

**Actual:** `500 Internal Server Error`.

**Result:** FAIL.

## Human classification

The two assertion failures are **not test-script mistakes**. Both expose the same robustness defect in the registration request-processing path.

The supplied API specification describes a JSON request body, so unsupported or missing media type does not need to be accepted. However, invalid client input should be rejected as a client-side request error (for example `400 Bad Request` or `415 Unsupported Media Type`) rather than causing an internal server error.

The reviewed implementation immediately destructures `req.body`:

```js
const { name, email, password } = req.body;
```

When the JSON body parser does not populate `req.body`, this path can throw before normal registration handling, producing HTTP 500.

## Decision

- Treat **REG-HUM-001** and **REG-HUM-002** as one genuine bug with two reproduction variants.
- Do **not** weaken the assertions merely to make Newman green.
- Keep the failing result as execution evidence for the discovered bug.
- File one GitHub Issue with both variants and attach evidence.
- Preserve `reports/newman/api1_registration.html` as the Newman report for this run.

## API 1 pipeline status

| Stage | Status |
|---|---|
| AI Generate | Complete — 42 cases |
| Human Audit | Complete — 42 audited |
| Human Extend | Complete — 6 added cases |
| Postman implementation | Complete |
| Newman execution | Complete |
| Genuine bug classification | Complete — 1 bug / 2 failing assertions |
| GitHub Issue | Pending |
| Screenshot evidence | Pending |
