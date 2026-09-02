# HW06 — API 3 Human-Designed Extension

**Feature:** FR-12 Access Control  
**Endpoint:** `GET /api/admin/users`  
**Human-added cases:** 6  

These cases were added after auditing the AI-generated suite. They focus on direct evidence for the explicit Admin-only requirement, end-to-end privilege escalation, scheme enforcement, stale privilege characterization, and response disclosure.

## Added test cases

| ID | Category | Objective | Request / Test Data | Final Oracle | Why the AI missed it |
|---|---|---|---|---|---|
| ADMIN-HUM-001 | Authorization / direct access | Use the seeded normal-user account to access the global admin user list. | Login as `test@eshop.com` / `Test1234!`, then call `GET /api/admin/users` with the returned JWT. | Must be denied because the supplied API specification states Admin APIs require an Admin account. Any `200` response containing the user list is a genuine FR-12 broken-access-control defect. | The AI had several equivalent normal-user denial cases, but this human case anchors the test to the actual seeded non-admin account and to directly observable global-user disclosure. |
| ADMIN-HUM-002 | Privilege escalation / profile update | Attempt to self-promote a normal user through `PUT /api/users/me` using `role: "admin"`, then re-login and access the admin user list. | 1) Login as normal user. 2) `PUT /api/users/me` with basic profile fields plus `role:"admin"`. 3) Login again. 4) Call `GET /api/admin/users`. | The user must not gain Admin privilege through the personal-profile endpoint. If re-login returns role `admin` and `/api/admin/users` becomes accessible, report a genuine vertical privilege-escalation defect. | The AI proposed the escalation concept, but did not define the full end-to-end state transition needed to prove persistence of the unauthorized role change. |
| ADMIN-HUM-003 | Authorization / response disclosure | Verify that an unauthorized normal-user response never includes global user records or PII. | Call `GET /api/admin/users` with a valid normal-user JWT and inspect both status and body. | Authorization must be denied, and the body must not contain user-list data such as other users' email, role, or shipping address. | The AI separated role denial and information-disclosure checks; this case combines them into a single evidence-oriented oracle. |
| ADMIN-HUM-004 | Authentication / scheme bypass | Send a valid admin JWT using a non-Bearer scheme to determine whether the middleware actually enforces the documented Bearer scheme. | Use `Authorization: Basic <valid-admin-jwt>` on `GET /api/admin/users`. | The specification requires `Authorization: Bearer <token>`. A non-Bearer scheme should not be accepted as compliant authentication. If the request succeeds, record a protocol/authentication-bypass defect against the documented scheme requirement. | The AI used `Basic abc`, which only proves that an invalid token fails verification. It did not pair the wrong scheme with a valid JWT to expose the middleware's scheme-agnostic parsing. |
| ADMIN-HUM-005 | Authorization / stale privilege | Demote an admin account after issuing an admin JWT, then reuse the old JWT against the admin endpoint. | 1) Obtain admin JWT. 2) Change the account role to `user` through a controlled setup step. 3) Reuse the old JWT on `GET /api/admin/users`. | Characterize whether the old token retains Admin access. The supplied spec does not define immediate token revocation, so continued access is security-relevant but should not be promoted to a conformance bug unless another requirement defines role revalidation/revocation. | The AI listed stale-role behavior but did not clearly separate a high-value security observation from a spec-backed defect criterion. |
| ADMIN-HUM-006 | Response privacy / field exposure | Inspect the successful admin user-list response for sensitive or unnecessary account-state fields. | Call `GET /api/admin/users` with a valid admin JWT and inspect every returned field. | Passwords, reset tokens, and authentication tokens must not be exposed. Other fields such as `login_attempts`, `locked_until`, or `shipping_address` should be documented as observed exposure; do not classify them as defects unless a privacy/minimization requirement explicitly forbids them. | The AI checked several sensitive fields independently, but did not distinguish clearly between definitely sensitive credentials and merely potentially excessive administrative data. |

## Extension rationale

The human extension prioritizes cases that can produce attributable evidence for FR-12 rather than adding more JWT syntax variations. In particular:

- `ADMIN-HUM-001` and `ADMIN-HUM-003` directly test whether a real seeded non-admin user can read the global user list.
- `ADMIN-HUM-002` tests a complete vertical privilege-escalation path through the personal-profile endpoint.
- `ADMIN-HUM-004` tests whether the documented Bearer authentication scheme is actually enforced using a valid JWT under the wrong scheme.
- `ADMIN-HUM-005` is retained as security characterization because the supplied specification does not define immediate JWT revocation on role changes.
- `ADMIN-HUM-006` distinguishes definite credential disclosure from merely potentially excessive administrative data.