# HW06 — API 1 Human Audit

**Feature:** FR-01 Account Registration  
**Endpoint:** `POST /api/register`  
**Audited AI cases:** 42  

## Audit basis

Primary oracle: the supplied EShop API specification. The registration specification gives the endpoint, an example body containing `name`, `email`, and `password`, and a successful `200` response containing `message` and `id`.

Human-review evidence from the SUT implementation was also used to identify unsupported AI assumptions: the route directly inserts the three body values using parameterized SQL, and the `users` table defines `name`, `email`, and `password` as nullable `TEXT` columns with no `UNIQUE`, `NOT NULL`, `CHECK`, length, email-format, or password-complexity constraints.

**Important audit rule:** implementation behavior is not automatically a requirement. Where the supplied specification does not define a rejection rule, the audit does not fabricate one. Such cases are retained as robustness/characterization tests where useful, but failure to reject is not automatically reported as a specification bug.

## Summary

- VALID: **11**
- INVALID: **5**
- INCOMPLETE: **26**
- Total: **42**

## Case-by-case audit

| ID | Audit Label | Human Review Reasoning | Corrected / Final Oracle |
|---|---|---|---|
| REG-AI-001 | VALID | The API spec explicitly defines this normal registration shape and 200 success response. | 200; exact JSON keys `message` and `id`; message = `User registered successfully`; `id` is numeric. |
| REG-AI-002 | VALID | The spec does not restrict Unicode names and the implementation stores `name` as SQLite TEXT. | 200 with documented success schema. |
| REG-AI-003 | VALID | The spec does not prohibit punctuation in names; the implementation passes the value as a bound SQL parameter. | 200 with documented success schema. |
| REG-AI-004 | VALID | Useful email-domain partition. No case-normalization or lowercase rule exists in the supplied spec/implementation. | 200 with documented success schema. |
| REG-AI-005 | VALID | A subdomain email fits the documented string field and no narrower email rule is specified. | 200 with documented success schema. |
| REG-AI-006 | VALID | A plus-alias email is a legitimate partition and no restriction is specified. | 200 with documented success schema. |
| REG-AI-007 | INCOMPLETE | The negative partition is useful, but AI invented a 4xx/non-empty-name requirement. The supplied API spec does not state it; code/schema contain no such validation. | For conformance, do not assert 4xx. Current implementation accepts/stores the value and should return the normal 200 success schema; record this as behavior, not a spec bug. |
| REG-AI-008 | INCOMPLETE | Same issue as REG-AI-007: blank-name rejection is not specified or implemented. | Do not assert 4xx. Current implementation should return 200; treat as robustness/characterization. |
| REG-AI-009 | INCOMPLETE | AI assumed `name` is formally required. The example contains `name`, but the supplied spec has no required-field declaration and the DB column is nullable. | Do not assert a spec-defined 4xx. Current implementation has no application/schema rejection; characterize execution result and do not call it a conformance defect from this spec alone. |
| REG-AI-010 | INCOMPLETE | Null-name rejection is not defined. `name` is nullable in the database. | Do not assert 4xx; current implementation permits NULL and is expected to return 200. |
| REG-AI-011 | INCOMPLETE | The spec shows `name` as a string example but provides no schema/type-validation rule. The handler performs no type check. | Retain as robustness test, but not as a spec-conformance 4xx assertion. Observe current behavior. |
| REG-AI-012 | INCOMPLETE | Non-string object input is a useful robustness case, but no type-validation oracle is defined and SQLite driver binding behavior must be established by execution. | Retain as robustness test; final executable oracle should be based on observed handling, not an invented 4xx requirement. |
| REG-AI-013 | INCOMPLETE | No maximum name length appears in the supplied spec or schema. | Retain as boundary/robustness test. Assert only safe handling/no crash unless another requirement supplies a maximum. |
| REG-AI-014 | INCOMPLETE | AI invented an empty-email rejection rule; neither spec nor implementation defines it. | Do not assert 4xx. Current implementation should accept/store the empty string and return 200. |
| REG-AI-015 | INCOMPLETE | Whitespace-email rejection/normalization is not specified or implemented. | Do not assert 4xx. Current implementation should return 200; classify as characterization. |
| REG-AI-016 | INCOMPLETE | AI assumed email is formally required, but the supplied spec has no required-field declaration and DB column is nullable. | Do not claim a spec defect solely from this test. Characterize current behavior. |
| REG-AI-017 | INCOMPLETE | Null-email rejection is not specified; DB column is nullable. | Do not assert 4xx; current implementation permits NULL and is expected to return 200. |
| REG-AI-018 | INCOMPLETE | The test partition is relevant, but the supplied API spec does not define email-format validation and the handler performs none. | Retain as negative/robustness case; current implementation is expected to accept it. Do not label failure-to-reject as a spec bug from this document alone. |
| REG-AI-019 | INCOMPLETE | Same as REG-AI-018: no email syntax rule is supplied. | Retain for robustness; do not assert spec-defined 4xx. |
| REG-AI-020 | INCOMPLETE | Same as REG-AI-018: no email syntax rule is supplied. | Retain for robustness; do not assert spec-defined 4xx. |
| REG-AI-021 | INCOMPLETE | Same as REG-AI-018: no email syntax rule is supplied. | Retain for robustness; do not assert spec-defined 4xx. |
| REG-AI-022 | INCOMPLETE | Same as REG-AI-018: no email syntax rule is supplied. | Retain for robustness; do not assert spec-defined 4xx. |
| REG-AI-023 | INCOMPLETE | AI gave two possible outcomes instead of one oracle. Trimming/normalization is not specified or implemented. | Current implementation stores the literal value and should return 200. Do not claim trimming is required. |
| REG-AI-024 | INCOMPLETE | Empty-password rejection is not specified; the password column is nullable and no validation is present. | Do not assert 4xx. Current implementation should return 200. |
| REG-AI-025 | INCOMPLETE | Whitespace-password rejection is not specified or implemented. | Do not assert 4xx. Current implementation should return 200. |
| REG-AI-026 | INCOMPLETE | Password is shown in the request example, but the supplied spec has no explicit required-field rule; DB column is nullable. | Do not assert a spec-defined 4xx. Characterize current behavior. |
| REG-AI-027 | INCOMPLETE | Null-password rejection is not specified; DB column is nullable. | Do not assert 4xx; current implementation permits NULL and is expected to return 200. |
| REG-AI-028 | INVALID | AI invented a minimum password length requirement that is absent from the supplied specification and implementation. | Reframe as a robustness/characterization test. Current implementation is expected to accept a one-character password and return 200; this is not a spec defect without another requirement. |
| REG-AI-029 | INVALID | AI invented a password-complexity rule requiring multiple character classes. | Reframe as characterization. Current implementation is expected to accept letters-only password and return 200. |
| REG-AI-030 | INVALID | AI invented a password-complexity rule requiring multiple character classes. | Reframe as characterization. Current implementation is expected to accept digits-only password and return 200. |
| REG-AI-031 | INVALID | AI invented a password-complexity rule requiring multiple character classes. | Reframe as characterization. Current implementation is expected to accept special-characters-only password and return 200. |
| REG-AI-032 | VALID | Unicode password is a legitimate text-domain partition; no character-set restriction is specified. | 200 with documented success schema. |
| REG-AI-033 | INCOMPLETE | No maximum password length is specified; AI supplied a non-deterministic oracle. | Retain as robustness/boundary test and assert safe handling/no crash. Do not invent a maximum. |
| REG-AI-034 | INCOMPLETE | AI assumed an empty object must be rejected. The supplied spec does not formally declare required properties and the schema columns are nullable. | Do not assert a spec-defined 4xx. Characterize actual implementation behavior. |
| REG-AI-035 | VALID | Malformed JSON is a protocol/parser robustness case. A JSON API should reject syntactically malformed JSON before normal handler processing. | 400 parser error; server remains available. Do not require the registration success schema. |
| REG-AI-036 | INCOMPLETE | The spec illustrates an object body, but no formal schema-validation response is defined. Handler code does not explicitly validate body type. | Retain as body-shape robustness test. Establish the exact implementation result during execution rather than inventing a 4xx oracle. |
| REG-AI-037 | VALID | This directly tests role-escalation/mass-assignment resistance. Handler destructures only `name`, `email`, `password`, and DB defaults role to `user`. | 200 is acceptable only if supplied `role` is ignored and resulting account remains a normal user. |
| REG-AI-038 | VALID | This tests server-owned identifier mass assignment. Handler ignores client `id`; SQLite owns the autoincrement primary key. | 200 is acceptable only if returned/generated id is server-controlled and not forced to `999999`. |
| REG-AI-039 | INCOMPLETE | The security objective is valid, but AI left an ambiguous `4xx or safe literal handling` oracle. The implementation uses parameterized SQL, so the payload should not change SQL semantics. | Correct oracle: request is handled as literal data (normally 200 in current implementation); no SQL execution/bypass/data disclosure occurs. |
| REG-AI-040 | INCOMPLETE | Stored-script input is a useful security probe, but registration alone cannot prove whether stored XSS executes later, and AI gave two possible HTTP outcomes. | At this endpoint, assert safe storage/handling and no server failure. Any execution of the script must be tested at a later rendering sink before calling it an XSS bug. |
| REG-AI-041 | INVALID | AI assumed email uniqueness. The supplied spec does not state it and the `users.email` column has no UNIQUE constraint; the handler performs no duplicate lookup. | Correct current expectation: second registration is accepted and creates another row. Treat uniqueness as a requirement gap/potential issue, not a confirmed API-spec defect from the supplied documents alone. |
| REG-AI-042 | VALID | HW06 explicitly requires schema validation and the API spec documents the success response shape as `message` + `id`. | 200; exact documented success fields are present; password and internal fields are not returned. |

## Main AI errors found

1. **Invented validation rules.** The AI repeatedly assumed that empty/missing fields, malformed email syntax, short passwords, or weak passwords must produce `4xx`, although those rules are not present in the supplied API specification.
2. **Invented uniqueness.** The AI assumed duplicate emails must be rejected, but the supplied schema has no `UNIQUE` constraint and the registration route performs no duplicate check.
3. **Ambiguous expected results.** Several cases used alternatives such as `200 or 4xx`, which are not executable test oracles.
4. **Mixed conformance and robustness testing.** Boundary/type/security probes are useful, but they must be distinguished from spec-defined requirements.

## Cases that remain especially valuable for the executable suite

- Normal success and exact response-schema checks.
- Unicode and representative valid-domain cases.
- Malformed JSON handling.
- Mass-assignment attempts using `role` and `id`.
- SQL-injection payload handled as literal data because parameterized queries are used.
- Oversized/type-invalid inputs as robustness cases, with oracles based on safe handling rather than invented validation rules.

## Human-audit conclusion

The AI generated broad input coverage, but its weakest area was **oracle quality**: it frequently converted common industry expectations into requirements that the provided specification never stated. The corrected suite will therefore separate specification-conformance assertions from robustness/security characterization and will avoid reporting unsupported validation expectations as genuine bugs.