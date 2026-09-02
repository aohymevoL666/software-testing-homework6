# HW06 — API 3 AI-Generated Test Cases

**Feature:** FR-12 Access Control  
**Endpoint:** `GET /api/admin/users`  
**Requirement context:** All `/api/admin/...` APIs require `Authorization: Bearer <token>` and the account must have Admin role.  
**Execution header required later:** `X-Student-Id: 23127531`  

> This file preserves the **AI-generation stage**. Expected results are AI draft expectations. They intentionally include some generic security expectations that must be checked during the mandatory Human Audit stage against the supplied API specification and actual SUT implementation.

## Coverage map

| Aspect | Partitions considered |
|---|---|
| Authentication | valid admin/user token, missing token, empty Bearer, malformed/tampered/expired/wrong-secret/unsigned JWT |
| Authorization | admin vs user, missing/null/non-string role, stale role transitions, nonexistent/deleted identity |
| Privilege escalation | self-service role change followed by admin access attempt |
| Response schema | array shape, user fields, sensitive-data exclusion |
| HTTP/protocol | unsupported methods, unusual GET body/content type, malformed Authorization spacing |
| State | account role changes, lockout, stale tokens |

**AI-generated cases:** 42

## Test cases

| ID | Category | Objective | Authentication / Setup | AI Draft Expected HTTP | AI Draft Expected Result | Coverage Basis |
|---|---|---|---|---|---|---|
| ADMIN-AI-001 | Positive / admin | Valid seeded admin token accesses admin user list | Bearer token for `admin@eshop.com` | 200 | Returns JSON array of users | Admin happy path |
| ADMIN-AI-002 | Authorization / role | Valid normal-user token accesses admin user list | Bearer token for `test@eshop.com` | 403 | Access denied because caller is not Admin | FR-12 / admin-only requirement |
| ADMIN-AI-003 | Authentication | No Authorization header | None | 401 | Returns Unauthorized error | Missing-token partition |
| ADMIN-AI-004 | Authentication | `Authorization: Bearer` with no token | Bearer only | 401 | Returns Unauthorized error | Empty-token partition |
| ADMIN-AI-005 | Authentication | Malformed random bearer token | Bearer `not-a-jwt` | 403 | Returns Forbidden error | Invalid JWT |
| ADMIN-AI-006 | Authentication | Tampered signature of otherwise valid admin JWT | Tampered admin JWT | 403 | Request is forbidden | JWT integrity |
| ADMIN-AI-007 | Authentication | Tampered signature of otherwise valid user JWT | Tampered user JWT | 403 | Request is forbidden | JWT integrity |
| ADMIN-AI-008 | Authentication / scheme | Use `Basic abc` instead of Bearer | Basic auth header | 401 or 403 | Request is rejected | Wrong auth scheme |
| ADMIN-AI-009 | Authentication / scheme | Use lowercase `bearer <token>` | Lowercase scheme + valid admin JWT | 200 or 4xx | Behavior is consistent and documented | Scheme parsing |
| ADMIN-AI-010 | Authentication / whitespace | Authorization header has multiple spaces before token | Bearer + multiple spaces + admin token | 200 or 4xx | Server handles malformed spacing safely | Header parser robustness |
| ADMIN-AI-011 | Authentication / whitespace | Authorization header has leading/trailing whitespace | Whitespace around Bearer/admin token | 200 or 4xx | Server handles header safely | Header parser robustness |
| ADMIN-AI-012 | Authentication | Authorization header contains two tokens | Bearer token1 token2 | 4xx | Ambiguous token header is rejected | Header robustness |
| ADMIN-AI-013 | Authentication | Expired JWT | Expired signed token | 403 | Expired token is forbidden | JWT expiry |
| ADMIN-AI-014 | Authentication | JWT signed with wrong secret | Wrong-secret token | 403 | Request is forbidden | JWT signature validation |
| ADMIN-AI-015 | Authentication | Unsigned JWT using `alg:none` | Unsigned crafted token | 403 | Request is forbidden | JWT algorithm security |
| ADMIN-AI-016 | Authentication | JWT with malformed three-part structure | Malformed JWT | 403 | Request is forbidden | JWT parser robustness |
| ADMIN-AI-017 | Authorization / role claim | Valid JWT carrying role=`user` | Signed normal-user JWT | 403 | Access denied | Role enforcement |
| ADMIN-AI-018 | Authorization / role claim | Valid JWT carrying role=`admin` | Signed admin JWT | 200 | Access allowed | Role enforcement |
| ADMIN-AI-019 | Authorization / role claim | JWT role value `Admin` with uppercase A | Signed token | 403 | Role comparison should be exact or normalized per policy | Role partition |
| ADMIN-AI-020 | Authorization / role claim | JWT role value `ADMIN` | Signed token | 403 | Access denied unless policy allows case-insensitive roles | Role partition |
| ADMIN-AI-021 | Authorization / role claim | JWT with missing `role` claim | Signed token without role | 403 | Access denied | Missing-role partition |
| ADMIN-AI-022 | Authorization / role claim | JWT with role = null | Signed token with null role | 403 | Access denied | Null-role partition |
| ADMIN-AI-023 | Authorization / role claim | JWT with role as array containing `admin` | Signed token with non-string role | 403 | Access denied | Type robustness |
| ADMIN-AI-024 | Authorization / identity | Valid JWT for nonexistent user id but role=`admin` | Signed token | 403 | Access denied because backing account does not exist | Token/account state |
| ADMIN-AI-025 | Authorization / identity | Valid JWT for deleted admin account | Previously issued admin token | 403 | Deleted account token is no longer authorized | Stale identity |
| ADMIN-AI-026 | Authorization / state | Issue user token, then change DB/account role to admin, reuse old token | Old token still says user | 403 | Authorization follows current account role or requires re-login | Role transition |
| ADMIN-AI-027 | Authorization / state | Issue admin token, then demote account to user, reuse old token | Old token still says admin | 403 | Demotion revokes admin access promptly | Role transition / stale token |
| ADMIN-AI-028 | Authorization / privilege escalation | Normal user attempts profile update with `role: admin`, then accesses admin users | User token + profile update helper | 403 for admin users endpoint | Self-service profile update must not grant admin privileges | Role escalation |
| ADMIN-AI-029 | Authorization / horizontal access | One normal user cannot access the global user list even if their own account id is valid | Normal-user token | 403 | No global user disclosure | Least privilege |
| ADMIN-AI-030 | Response schema | Admin success response is a JSON array | Valid admin token | 200 | Body is an array | Schema validation |
| ADMIN-AI-031 | Response schema | Each returned user contains documented/safe admin-list fields | Valid admin token | 200 | User entries contain expected fields only | Schema validation |
| ADMIN-AI-032 | Security / sensitive data | Admin list must not expose password | Valid admin token | 200 | No entry contains `password` | Sensitive-data exposure |
| ADMIN-AI-033 | Security / sensitive data | Admin list must not expose reset token | Valid admin token | 200 | No entry contains `reset_token` | Sensitive-data exposure |
| ADMIN-AI-034 | Security / sensitive data | Admin list must not expose JWT/token values | Valid admin token | 200 | No auth tokens are present in response | Sensitive-data exposure |
| ADMIN-AI-035 | Security / privacy | Normal user response must not reveal any user records in unauthorized case | Normal-user token | 403 | No user list or PII is returned | Authorization + information disclosure |
| ADMIN-AI-036 | HTTP method | POST request to `/api/admin/users` | Valid admin token | 404/405 | Unsupported method is rejected | Method partition |
| ADMIN-AI-037 | HTTP method | PUT request to `/api/admin/users` | Valid admin token | 404/405 | Unsupported method is rejected | Method partition |
| ADMIN-AI-038 | HTTP method | DELETE request to `/api/admin/users` without id | Valid admin token | 404/405 | Unsupported route/method is rejected | Method/route partition |
| ADMIN-AI-039 | Protocol | GET with JSON request body | Valid admin token | 200 or 4xx | Server handles unusual GET body safely | Protocol robustness |
| ADMIN-AI-040 | Protocol | GET with unsupported `Content-Type: text/plain` and no body | Valid admin token | 200 | Content-Type should not affect bodyless GET authorization | Protocol robustness |
| ADMIN-AI-041 | State / lockout | Locked normal user token attempts admin users endpoint | Valid token from account later locked | 403 | Locked/non-admin user cannot access admin data | Account-state authorization |
| ADMIN-AI-042 | State / lockout | Locked admin account token attempts admin users endpoint | Previously issued admin token | 403 | Locked admin should not retain privileged access | Account-state authorization |

## Notes for Human Audit

The reviewed route currently uses only `authenticateToken`; no role-check middleware or `req.user.role === 'admin'` condition is present in the `GET /api/admin/users` handler. This observation must be used during audit to distinguish genuine FR-12 authorization failures from AI assumptions about other behaviors such as token revocation, lockout enforcement, or backing-account lookups.

The response query selects `id`, `name`, `email`, `role`, `login_attempts`, `locked_until`, and `shipping_address`; it does not select `password` or `reset_token`.