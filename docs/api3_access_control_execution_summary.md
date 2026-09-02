# HW06 — API 3 Execution Summary

**Feature:** FR-12 Access Control  
**Primary endpoint:** `GET /api/admin/users`  
**Tool:** Postman + Newman 6.2.2  
**Environment:** `http://localhost:3000`  
**Required student header:** `X-Student-Id: 23127531`

## Newman run summary

| Metric | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 47 | 0 |
| Test scripts | 47 | 0 |
| Pre-request scripts | 50 | 0 |
| Assertions | 57 | 18 |

- Total duration: **4.2 s**
- Average response time: **3 ms**
- Minimum response time: **1 ms**
- Maximum response time: **26 ms**
- Failed assertions: **18**

## Failure classification

The 18 failed assertions reduce to **three genuine defects plus one test-oracle correction**.

### Root cause 1 — Broken admin access control

Affected failures:

- 1–12
- 16
- 17

Observed behavior:

- A valid normal-user JWT receives `200 OK` from `GET /api/admin/users`.
- The response is the global JSON user array.
- This occurs with normal `Bearer`, lowercase `bearer`, and even `Basic <valid-user-JWT>` because the route verifies JWT validity but never enforces Admin role.

Classification: **Genuine bug — Broken Access Control / Missing Role Authorization**

### Root cause 2 — Bearer authentication scheme is not enforced

Affected failure:

- 13
- Contributes to 17

Observed behavior:

- `Authorization: Basic <valid-admin-JWT>` returns `200 OK`.
- The authentication middleware extracts the second space-separated value and verifies it as a JWT without checking that the scheme is `Bearer`.

Classification: **Genuine bug — Authentication scheme enforcement**

### Root cause 3 — Vertical privilege escalation through profile update

Affected failures:

- 14
- 15 is downstream evidence

Observed behavior:

1. A normal user calls `PUT /api/users/me` with `role:"admin"`.
2. The request returns `200 OK`.
3. Re-login returns the account with role `admin`.
4. The new token successfully accesses the admin user list.

Classification: **Genuine bug — Vertical Privilege Escalation**

Note: failure 15 also overlaps root cause 1 because `/api/admin/users` lacks role enforcement. Failure 14 is the decisive proof that the personal-profile endpoint persisted an unauthorized Admin role.

### Failure 18 — Empty Authorization header

Observed:

- Expected by the test: `401`
- Actual: `403`

Reason:

The middleware evaluates an empty Authorization header differently from a completely missing header and eventually calls JWT verification on an empty token. The supplied API specification does not define the exact status code for an empty Authorization header.

Classification: **Test-oracle correction / characterization, not a confirmed bug**

Corrected oracle: request must be rejected with a 4xx response and must not return admin data.

## Important successful checks

- Valid admin JWT → `200 OK`.
- Missing Authorization header → `401 Unauthorized`.
- Missing Bearer token → `401 Unauthorized`.
- Malformed/tampered/unsigned/invalid JWTs → `403 Forbidden`.
- Unsupported POST/PUT/DELETE methods on the collection route → `404`.
- Admin list does not expose `password`, `reset_token`, JWT, or token fields.
- Malformed Authorization variants do not cause 5xx responses.

## Final API 3 bug count

**3 confirmed bugs**

1. Broken Admin Access Control — non-admin users can read `/api/admin/users`.
2. Bearer scheme not enforced — a valid JWT works under `Basic`.
3. Vertical privilege escalation — normal users can self-assign `role:"admin"` via `PUT /api/users/me`.

## API 3 pipeline status

| Stage | Status |
|---|---|
| AI Generate | Complete — 42 cases |
| Human Audit | Complete — 42 audited |
| Human Extend | Complete — 6 added cases |
| Postman implementation | Complete |
| Newman execution | Complete |
| Assertions | 57 total / 18 failed |
| Genuine bugs | 3 |
| Newman HTML report | Generated |
| GitHub Issues | Pending |
| Screenshot evidence | Pending |
