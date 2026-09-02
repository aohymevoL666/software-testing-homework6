# HW06 — API 1 Human-Designed Extension

**Feature:** FR-01 Account Registration  
**Endpoint:** `POST /api/register`  
**Human-added cases:** 6  

These cases were added after reviewing the AI-generated suite. They are deliberately targeted at gaps the AI missed, especially protocol handling, request-body shape, account-state interactions, and server-owned security fields.

## Added test cases

| ID | Category | Objective | Request / Test Data | Final Oracle | Why the AI missed it |
|---|---|---|---|---|---|
| REG-HUM-001 | Protocol / robustness | Send a JSON-looking registration body with `Content-Type: text/plain`. | Header `Content-Type: text/plain`; body `{"name":"Human 001","email":"hw06.hum001@example.com","password":"Password123!"}` | The server must handle the request safely and must not crash. Record the actual HTTP status/body; do not treat a particular validation status as specified because the API spec does not define unsupported content-type behavior. | The AI focused mostly on JSON field partitions and malformed JSON, but did not vary the HTTP media type. |
| REG-HUM-002 | Protocol / robustness | Send the normal registration body without a `Content-Type` header. | No `Content-Type`; raw body `{"name":"Human 002","email":"hw06.hum002@example.com","password":"Password123!"}` | The server must fail safely or process the request consistently; no unhandled server crash. Exact status is characterization because the supplied API spec does not define missing-content-type behavior. | The AI treated the request body as if JSON parsing were always correctly configured and did not test the parser precondition. |
| REG-HUM-003 | Body shape / robustness | Send JSON `null` as the entire request body. | Header `Content-Type: application/json`; body `null` | The request must be handled without an unhandled exception or server termination. Exact error status/body is characterized during execution. | The AI tested `{}` and an array, but missed the distinct `null` body shape, which is important because the handler immediately destructures `req.body`. |
| REG-HUM-004 | State / account identity | Register the same email twice but use a different name and password on the second request. | 1) `{"name":"First Owner","email":"hw06.identity@example.com","password":"Password111!"}`  2) `{"name":"Second Owner","email":"hw06.identity@example.com","password":"Different222!"}` | Characterize whether two account records can share the same email. The supplied spec does not define uniqueness, so duplicate acceptance is not automatically a conformance bug; however, the result is security-relevant because login identity may become ambiguous. | The AI only tried an identical duplicate request and did not vary credentials, so it missed the account-identity ambiguity created when duplicate emails have different passwords. |
| REG-HUM-005 | State / normalization | Register two accounts whose emails differ only by letter case. | 1) `{"name":"Case One","email":"CaseUser@example.com","password":"Password123!"}`  2) `{"name":"Case Two","email":"caseuser@example.com","password":"Password456!"}` | Characterize whether the SUT treats the two values as distinct identities. No normalization rule is stated in the supplied spec, so the test documents behavior rather than assuming rejection. | The AI tested uppercase email syntax but did not combine case variation with registration state across two requests. |
| REG-HUM-006 | Security / mass assignment | Attempt to set security-sensitive server-owned account fields during registration. | `{"name":"Human 006","email":"hw06.hum006@example.com","password":"Password123!","login_attempts":99,"locked_until":"2099-01-01T00:00:00Z","reset_token":"attacker-token"}` | Registration may succeed, but the supplied values for `login_attempts`, `locked_until`, and `reset_token` must not be written from the client request. The handler should use only `name`, `email`, and `password` and database defaults/internal state for the other fields. | The AI checked mass assignment for `role` and `id` only. Human review of the user schema exposed additional security-sensitive fields that are also server-owned. |

## Extension rationale

The AI produced broad parameter coverage, but its generation was dominated by individual field partitions. Human review added tests that require understanding interactions outside a single field:

- how Express/JSON parsing behaves when the media type or body shape changes;
- how registration state behaves when account identity collides across multiple requests;
- how server-owned security fields in the database schema should resist client mass assignment.

The extension intentionally avoids inventing unspecified validation rules. Where the supplied API specification does not define a particular rejection status, the case is treated as robustness/characterization and the executable oracle focuses on safe handling or on security properties that can be directly observed.