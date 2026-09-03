# HW06 — Agent Skill G9.5: AI-Driven API Test Generator

**Student ID:** 23127531  
**Artifact type:** Design + pseudocode  
**Implementation status:** Design-only (implementation/video optional)  

## 1. Goal

Design an AI-driven API test generator that accepts an API specification and produces candidate API test cases for later human review.

The generator is intentionally separated into two stages:

1. **AI Generation** — maximize useful test coverage.
2. **Human Audit** — validate every generated oracle against the real API specification before execution.

This separation is important because AI can generate plausible but unsupported expectations.

## 2. Input

The generator receives:

- API specification text;
- endpoint path;
- HTTP method;
- authentication requirements;
- request parameters / request-body schema;
- response examples or response schema;
- explicit functional/security constraints;
- optional implementation observations supplied by the tester.

Example input:

```text
Endpoint: GET /api/admin/users
Authentication: Authorization: Bearer <token>
Authorization: account must have Admin role
Response: list of users
```

## 3. Output

The generator returns a structured list of candidate test cases.

Each test case contains:

- test case ID;
- category;
- objective;
- preconditions;
- request data / headers;
- expected status;
- expected result;
- coverage basis;
- confidence / requirement source.

Example:

```text
ID: ADMIN-AI-002
Category: Authorization
Objective: Normal user attempts to read admin user list
Precondition: valid JWT for role=user
Expected: access denied; no global user list returned
Basis: explicit Admin-only requirement
```

## 4. Main test-generation categories

The generator attempts to create cases from these categories when applicable:

1. Happy path.
2. Required/missing/null request fields.
3. Valid and invalid equivalence partitions.
4. Boundary values.
5. Data type variations.
6. Authentication variants.
7. Authorization / role checks.
8. State transitions.
9. Security-focused inputs.
10. HTTP/protocol variations.
11. Response schema checks.
12. Sensitive-data disclosure checks.

The generator should create at least 35 candidate cases for a selected API when sufficient input dimensions exist.

## 5. Human-audit labels

Every AI-generated case must be reviewed and assigned one label:

### VALID

The expected behavior is explicitly supported by the specification or another authoritative project requirement.

### INVALID

The expected behavior contradicts the specification or assumes a requirement that is known to be false.

### INCOMPLETE

The test idea may be useful, but the specification does not provide enough information for a deterministic oracle.

The audit step can rewrite the expected result while preserving the original AI-generated draft for traceability.

## 6. Design principles

### Requirement-grounded oracles

The generator must distinguish:

```text
"commonly recommended behavior"
```

from:

```text
"behavior explicitly required by this API specification"
```

A security best practice should not automatically become a conformance oracle.

### Traceability

Each case stores a `coverage_basis` field explaining where the expectation came from.

Possible values:

```text
explicit_spec
response_schema
authentication_requirement
authorization_requirement
boundary_partition
security_heuristic
implementation_characterization
```

### No automatic bug declaration

The generator does not decide that every unusual response is a bug. A bug is reported only after human review establishes a violated requirement.

## 7. Processing stages

The system follows these logical stages:

1. Receive API specification.
2. Parse endpoint metadata.
3. Extract explicit requirements.
4. Build parameter and state models.
5. Generate candidate partitions.
6. Add authentication/security scenarios.
7. Generate candidate test cases with traceability.
8. Deduplicate equivalent cases.
9. Send all generated cases to Human Audit.
10. Human reviewer labels VALID / INVALID / INCOMPLETE.
11. Corrected cases are exported to Postman/Newman or another execution tool.
12. Execution results are compared with the human-approved oracle.
13. Confirmed requirement violations are eligible for bug reports.

## 8. Diagram artifact

The final assignment diagram must be **drawn manually by the student** and saved as:

```text
agent-skill/api_test_generator_diagram.png
```

The diagram should visually represent the processing stages described above. Do not submit an AI-generated diagram.

## 9. Expected strengths

- Fast generation of many candidate cases.
- Broad partition and security coverage.
- Consistent test-case structure.
- Traceability from generated test to coverage basis.
- Human review prevents unsupported AI assumptions from becoming false bug reports.

## 10. Expected limitations

- Natural-language API specifications may be incomplete.
- AI may import common industry behavior that the project never specifies.
- Exact status codes may be ambiguous.
- State-transition semantics may require implementation inspection or manual setup.
- Security requirements such as token revocation cannot be assumed without explicit evidence.
- Human audit remains mandatory.

## 11. Example learned from HW06

The generator may initially produce:

```text
POST /api/cart with nonexistent product id
Expected: 404 Not Found
```

If the specification does not state that product existence must be validated, Human Audit should mark the oracle INVALID or INCOMPLETE rather than treating a `200` response as a confirmed defect.

In contrast:

```text
GET /api/admin/users using a normal-user JWT
```

has an explicit Admin-only requirement. Therefore the denial oracle is VALID, and a `200` response can be classified as a genuine access-control defect after execution.
