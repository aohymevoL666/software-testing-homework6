# HW06 — API 3 Human Audit

**Feature:** FR-12 Access Control  
**Endpoint:** `GET /api/admin/users`  
**Audited AI cases:** 42  

## Audit basis

The supplied API specification explicitly states that **all Admin APIs require `Authorization: Bearer <token>` and the account must have Admin role**. This is the primary authorization oracle for FR-12.

Human review of the SUT implementation found that `GET /api/admin/users` uses only `authenticateToken`. The middleware verifies JWT validity but the route does **not** check `req.user.role` or re-query the backing account before returning the global user list.

The route currently queries: `id`, `name`, `email`, `role`, `login_attempts`, `locked_until`, and `shipping_address`; it does not query `password` or `reset_token`.

**Audit rule:** test cases backed by the explicit Admin-only requirement remain VALID even if the current implementation is expected to fail them. Cases about token revocation, role-claim refresh, lockout revocation, or exact response fields are treated separately because those behaviors are not explicitly defined in the supplied API specification.

## Summary

- VALID: **26**
- INVALID: **0**
- INCOMPLETE: **16**
- Total: **42**

## Case-by-case audit

| ID | Audit Label | Human Review Reasoning | Corrected / Final Oracle |
|---|---|---|---|
| ADMIN-AI-001 | VALID | The supplied API specification explicitly states that all `/api/admin/...` endpoints require a Bearer token and an Admin account. A valid admin accessing the endpoint is the baseline positive case. | Valid seeded admin token → 200 and a JSON user list. |
| ADMIN-AI-002 | VALID | This is the core FR-12 negative case. The specification explicitly requires Admin role, so a normal authenticated user must not receive the admin user list. | Valid normal-user token → 403 (or another clear authorization-denied 4xx); response must not contain the user list. |
| ADMIN-AI-003 | VALID | The endpoint requires Bearer authentication and the middleware explicitly returns 401 when no token is present. | No Authorization header → 401 with `Unauthorized` error. |
| ADMIN-AI-004 | VALID | The middleware extracts the second Authorization token component; `Bearer` without a token follows the missing-token path. | `Authorization: Bearer` → 401. |
| ADMIN-AI-005 | VALID | Malformed JWT is a valid authentication partition and the middleware explicitly maps verification errors to 403. | Malformed bearer token → 403 with `Forbidden` error. |
| ADMIN-AI-006 | VALID | Tampering with a valid admin token directly tests JWT integrity and should make verification fail. | Tampered admin JWT → 403. |
| ADMIN-AI-007 | VALID | Tampering with a valid normal-user token is another JWT-integrity partition and should fail verification. | Tampered user JWT → 403. |
| ADMIN-AI-008 | INCOMPLETE | The objective is useful, but the AI gave `401 or 403`. The middleware does not verify the Bearer scheme; it simply extracts the second space-separated component, so `Basic abc` attempts to verify `abc` as a JWT. | Current implementation expectation: `Basic abc` → 403. Keep as protocol/authentication characterization. |
| ADMIN-AI-009 | INCOMPLETE | The AI gave an ambiguous oracle. The middleware ignores scheme text and only uses the second token component, so lowercase `bearer <valid-token>` is accepted. | Current implementation: lowercase `bearer` with a valid admin token still authenticates. Authorization must still depend on Admin role. |
| ADMIN-AI-010 | INCOMPLETE | Multiple spaces interact with `split(" ")[1]`; the exact extracted token may be empty. The spec does not define whitespace normalization. | Retain as header-parser robustness. Establish exact 401/403 behavior during execution; do not invent a requirement. |
| ADMIN-AI-011 | INCOMPLETE | Leading/trailing Authorization whitespace behavior is not defined by the supplied spec and may be normalized by the HTTP stack before Express sees it. | Retain as protocol characterization; require safe handling and no 5xx. |
| ADMIN-AI-012 | INCOMPLETE | The AI assumes an ambiguous multi-token header must be rejected, but the middleware simply uses the second segment. The exact outcome depends on the first extracted token. | Retain as parser robustness; expected result should be based on observed extraction/verification, not a generic 4xx assumption. |
| ADMIN-AI-013 | VALID | JWT expiry is part of standard JWT verification behavior; `jwt.verify` rejects an expired token. | Expired token → 403. |
| ADMIN-AI-014 | VALID | A token signed with a different secret must fail `jwt.verify`. | Wrong-secret JWT → 403. |
| ADMIN-AI-015 | VALID | An unsigned `alg:none` token must not be accepted by normal `jwt.verify` with a secret. | Unsigned JWT → 403. |
| ADMIN-AI-016 | VALID | Malformed JWT structure is a valid robustness/authentication case and should fail verification. | Malformed three-part JWT → 403. |
| ADMIN-AI-017 | VALID | A signed token representing a normal user is exactly the role-based denial required by FR-12. | Role=`user` token → authorization denied; must not receive user list. |
| ADMIN-AI-018 | VALID | A signed token representing an Admin is the positive role-partition case. | Role=`admin` token → 200. |
| ADMIN-AI-019 | INCOMPLETE | The supplied specification says Admin role but does not define case normalization. The AI invented an exact comparison policy. | Retain as role-string characterization only if a safely generated token is available; do not claim a conformance defect from case sensitivity alone. |
| ADMIN-AI-020 | INCOMPLETE | Same issue as ADMIN-AI-019: no case-normalization rule is stated. | Retain as role-string characterization; no fabricated policy. |
| ADMIN-AI-021 | VALID | A token without an Admin role cannot satisfy the explicit Admin-only requirement. | Signed token lacking `role` → authorization denied. |
| ADMIN-AI-022 | VALID | A null role is not Admin and therefore cannot satisfy the explicit Admin-only requirement. | Signed token with `role:null` → authorization denied. |
| ADMIN-AI-023 | VALID | An array-valued role is not the Admin role required by the specification. | Signed token with non-string role → authorization denied. |
| ADMIN-AI-024 | INCOMPLETE | The supplied specification requires an Admin account but does not define whether each request must re-query the database to prove the backing account still exists. The current middleware trusts signed JWT claims only. | Retain as stale-identity/security characterization; do not assert a spec-defined 403 without a stronger account-liveness requirement. |
| ADMIN-AI-025 | INCOMPLETE | Token revocation after account deletion is not specified. The AI invented prompt revocation semantics. | Retain as stale-token characterization; do not report acceptance as a confirmed spec defect unless another requirement defines revocation. |
| ADMIN-AI-026 | INCOMPLETE | The spec does not define immediate token claim refresh after a user is promoted. A previously issued JWT may legitimately retain its old role until re-login/expiry. | Retain as role-transition characterization; old token remaining non-admin is not automatically a bug. |
| ADMIN-AI-027 | INCOMPLETE | The AI assumes immediate privilege revocation after demotion, but the supplied spec does not define token revocation or DB role revalidation. | Retain as stale-admin-token characterization. This may be security-relevant but requires a revocation requirement before calling it a conformance defect. |
| ADMIN-AI-028 | VALID | This is supported by the supplied spec: `/api/users/me` is described as allowing only basic personal information updates, while Admin APIs require Admin role. Supplying `role:admin` must not let a normal user become an admin. | Normal user must not obtain Admin access through profile self-update. If role escalation plus re-login enables `/api/admin/users`, report a genuine privilege-escalation bug. |
| ADMIN-AI-029 | VALID | This restates the explicit least-privilege property of FR-12: a normal user cannot access the global admin user list. | Normal user → authorization denied and no global user records returned. |
| ADMIN-AI-030 | INCOMPLETE | The endpoint name implies a user list and current implementation returns an array, but the supplied API spec does not formally define the response schema. | Retain as schema characterization: successful admin response should be valid JSON and list-like; do not claim an exact schema requirement that is absent. |
| ADMIN-AI-031 | INCOMPLETE | The AI refers to 'documented/safe admin-list fields', but the supplied spec does not document the fields returned by this endpoint. | Retain as response-shape characterization using observed fields; do not claim exact-field conformance. |
| ADMIN-AI-032 | VALID | Even though the response schema is not fully documented, returning plaintext account passwords in an administrative list would be a clear sensitive-data exposure. The implementation query intentionally excludes `password`. | Admin response must not expose `password`. |
| ADMIN-AI-033 | VALID | A reset token is authentication-sensitive data and should not be disclosed in a bulk user-list response. The implementation query excludes it. | Admin response must not expose `reset_token`. |
| ADMIN-AI-034 | VALID | Authentication tokens must not be present in a user-list response. This is a defensible security property even though the spec does not enumerate response fields. | No JWT/access token values in response. |
| ADMIN-AI-035 | VALID | This is a direct information-disclosure consequence of FR-12. An unauthorized normal user must not receive the global user data. | Normal user → authorization denied; response contains no user records/PII. |
| ADMIN-AI-036 | VALID | POST is not defined for `/api/admin/users`; exercising an unsupported method is a valid protocol/route test. | POST `/api/admin/users` → 404 or 405; no user-list disclosure. |
| ADMIN-AI-037 | VALID | PUT is not defined for `/api/admin/users`; unsupported method testing is valid. | PUT `/api/admin/users` → 404 or 405. |
| ADMIN-AI-038 | VALID | DELETE is only specified for `/api/admin/users/:id`, not for the collection path without an id. | DELETE `/api/admin/users` → 404 or 405. |
| ADMIN-AI-039 | INCOMPLETE | The spec does not define behavior for a GET request body. The security objective is simply safe handling, not a specific success/error status. | Retain as protocol robustness: no 5xx; authorization rules must still apply. |
| ADMIN-AI-040 | VALID | A bodyless GET does not depend on JSON content parsing; adding `Content-Type:text/plain` should not bypass authentication/authorization. | Admin token still receives normal admin response; normal-user token must still be denied. |
| ADMIN-AI-041 | INCOMPLETE | The supplied materials define login lockout behavior, but they do not state that an already issued token must be revoked when the account becomes locked. | Retain as locked-user stale-token characterization; do not invent revocation semantics. |
| ADMIN-AI-042 | INCOMPLETE | Same as ADMIN-AI-041: prompt revocation of a previously issued admin JWT on lockout is not explicitly specified. | Retain as high-value security characterization, but acceptance is not automatically a spec defect without a token-revocation requirement. |

## Main AI issues found

1. **Correctly identified the central authorization requirement.** Unlike API 1 and API 2, many AI cases here are directly grounded in the explicit Admin-only rule.
2. **Invented token-revocation semantics.** The AI repeatedly assumed deleted, demoted, or locked accounts must immediately invalidate previously issued JWTs, but the supplied API spec does not state such a policy.
3. **Ambiguous protocol oracles.** Several cases used `200 or 4xx` / `401 or 403` instead of a deterministic expectation.
4. **Overstated response-schema knowledge.** The API spec does not enumerate the exact fields returned by `GET /api/admin/users`, so exact-field assertions require characterization rather than fabricated documentation.

## Strongest execution cases

- `ADMIN-AI-002` / `ADMIN-AI-017` / `ADMIN-AI-029` / `ADMIN-AI-035`: normal-user JWT must not receive the admin user list.
- `ADMIN-AI-001` / `ADMIN-AI-018`: Admin happy path.
- `ADMIN-AI-003`–`007`, `013`–`016`: authentication/JWT rejection.
- `ADMIN-AI-028`: normal user must not self-promote through profile update and then gain Admin access.
- `ADMIN-AI-032`–`034`: sensitive-data non-disclosure.

## Human-audit conclusion

The most important expected defect is not an AI assumption: it is directly supported by the supplied specification. The current `GET /api/admin/users` implementation verifies that a JWT is valid but does not verify that the authenticated caller is an Admin. Therefore execution with the seeded normal user is expected to provide decisive evidence for or against a genuine FR-12 broken-access-control bug.