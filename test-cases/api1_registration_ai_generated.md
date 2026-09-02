# HW06 — API 1 AI-Generated Test Cases

**Feature:** FR-01 Account Registration  
**Endpoint:** `POST /api/register`  
**Base URL:** `http://localhost:3000`  
**Execution header required later:** `X-Student-Id: 23127531`  

> This file preserves the **AI-generation stage**. Expected results are AI draft expectations derived from the supplied API specification and standard API-testing heuristics. They are **not yet the human-reviewed final oracle**. Items not explicitly defined by the specification are intentionally left for the Audit stage.

## Partition map

| Input / Aspect | Partitions considered |
|---|---|
| `name` | normal, Unicode, punctuation, empty, whitespace, missing, null, wrong type, very long |
| `email` | normal, uppercase, subdomain, plus alias, empty, whitespace, missing, null, malformed syntax, normalization |
| `password` | normal, empty, whitespace, missing, null, short, character-class variants, Unicode, very long |
| Request body | normal object, empty object, malformed JSON, array instead of object, extra fields |
| State | first registration, duplicate registration |
| Security | role mass assignment, id mass assignment, SQL injection, stored-content/XSS-oriented input, sensitive response exposure |
| Schema | success status and documented `message` + `id` response |

**AI-generated cases:** 42

## Test cases

| ID | Category | Objective | Request / Test Data | AI Draft Expected HTTP | AI Draft Expected Result | Coverage Basis |
|---|---|---|---|---|---|---|
| REG-AI-001 | Positive / baseline | Register with typical valid values | {"name":"Nguyen Van A","email":"hw06.reg001@example.com","password":"Password123!"} | 200 | Response contains success message and integer id | API specification success example |
| REG-AI-002 | Name / valid partition | Vietnamese Unicode name | {"name":"Nguyễn Văn Ánh","email":"hw06.reg002@example.com","password":"Password123!"} | 200 | Registration succeeds and returns message + id | Unicode text should be accepted unless restricted |
| REG-AI-003 | Name / valid partition | Name containing hyphen/apostrophe | {"name":"Anne-Marie O'Neil","email":"hw06.reg003@example.com","password":"Password123!"} | 200 | Registration succeeds | Common valid name characters |
| REG-AI-004 | Email / valid partition | Uppercase characters in email | {"name":"User 004","email":"HW06.REG004@EXAMPLE.COM","password":"Password123!"} | 200 | Registration succeeds | Email syntax valid; case handling to audit |
| REG-AI-005 | Email / valid partition | Email with subdomain | {"name":"User 005","email":"user@sub.example.com","password":"Password123!"} | 200 | Registration succeeds | Valid email syntax |
| REG-AI-006 | Email / valid partition | Email with plus alias | {"name":"User 006","email":"user+hw06@example.com","password":"Password123!"} | 200 | Registration succeeds | Valid email syntax |
| REG-AI-007 | Name / invalid partition | Empty name | {"name":"","email":"hw06.reg007@example.com","password":"Password123!"} | 4xx | Request rejected with validation error | AI assumes name is required/non-empty |
| REG-AI-008 | Name / invalid partition | Whitespace-only name | {"name":"   ","email":"hw06.reg008@example.com","password":"Password123!"} | 4xx | Request rejected with validation error | AI assumes blank names invalid |
| REG-AI-009 | Name / invalid partition | Missing name field | {"email":"hw06.reg009@example.com","password":"Password123!"} | 4xx | Request rejected with validation error | Body field shown as required by operation example; confirm in audit |
| REG-AI-010 | Name / invalid partition | name = null | {"name":null,"email":"hw06.reg010@example.com","password":"Password123!"} | 4xx | Request rejected with validation error | Null partition |
| REG-AI-011 | Name / type partition | Numeric name | {"name":12345,"email":"hw06.reg011@example.com","password":"Password123!"} | 4xx | Request rejected because name should be string | Type robustness |
| REG-AI-012 | Name / type partition | Object as name | {"name":{"first":"Nguyen"},"email":"hw06.reg012@example.com","password":"Password123!"} | 4xx | Request rejected because name should be string | Type robustness |
| REG-AI-013 | Name / boundary | Extremely long name (~1000 chars) | {"name":"<1000-character string>","email":"hw06.reg013@example.com","password":"Password123!"} | 4xx or bounded acceptance | Server enforces a reasonable maximum or safely accepts without failure | Maximum length not specified; must audit |
| REG-AI-014 | Email / invalid partition | Empty email | {"name":"User 014","email":"","password":"Password123!"} | 4xx | Request rejected with validation error | AI assumes valid email required |
| REG-AI-015 | Email / invalid partition | Whitespace-only email | {"name":"User 015","email":"   ","password":"Password123!"} | 4xx | Request rejected with validation error | AI assumes blank email invalid |
| REG-AI-016 | Email / invalid partition | Missing email field | {"name":"User 016","password":"Password123!"} | 4xx | Request rejected with validation error | Required-field partition |
| REG-AI-017 | Email / invalid partition | email = null | {"name":"User 017","email":null,"password":"Password123!"} | 4xx | Request rejected with validation error | Null partition |
| REG-AI-018 | Email / syntax | Email without @ | {"name":"User 018","email":"userexample.com","password":"Password123!"} | 4xx | Request rejected as invalid email | Email syntax partition |
| REG-AI-019 | Email / syntax | Email missing local part | {"name":"User 019","email":"@example.com","password":"Password123!"} | 4xx | Request rejected as invalid email | Email syntax partition |
| REG-AI-020 | Email / syntax | Email missing domain | {"name":"User 020","email":"user@","password":"Password123!"} | 4xx | Request rejected as invalid email | Email syntax partition |
| REG-AI-021 | Email / syntax | Email containing multiple @ characters | {"name":"User 021","email":"a@b@example.com","password":"Password123!"} | 4xx | Request rejected as invalid email | Email syntax partition |
| REG-AI-022 | Email / syntax | Email with consecutive dots | {"name":"User 022","email":"user..name@example.com","password":"Password123!"} | 4xx | Request rejected as invalid email | Email syntax partition |
| REG-AI-023 | Email / normalization | Valid email surrounded by spaces | {"name":"User 023","email":"  hw06.reg023@example.com  ","password":"Password123!"} | 200 or 4xx | Server should either trim then accept or reject consistently; must not silently create an unusable account | Normalization behavior not specified |
| REG-AI-024 | Password / invalid partition | Empty password | {"name":"User 024","email":"hw06.reg024@example.com","password":""} | 4xx | Request rejected with validation error | AI assumes password required/non-empty |
| REG-AI-025 | Password / invalid partition | Whitespace-only password | {"name":"User 025","email":"hw06.reg025@example.com","password":"   "} | 4xx | Request rejected with validation error | AI assumes blank password invalid |
| REG-AI-026 | Password / invalid partition | Missing password field | {"name":"User 026","email":"hw06.reg026@example.com"} | 4xx | Request rejected with validation error | Required-field partition |
| REG-AI-027 | Password / invalid partition | password = null | {"name":"User 027","email":"hw06.reg027@example.com","password":null} | 4xx | Request rejected with validation error | Null partition |
| REG-AI-028 | Password / boundary | One-character password | {"name":"User 028","email":"hw06.reg028@example.com","password":"A"} | 4xx | Request rejected if a minimum password length is required | Password complexity not specified; must audit |
| REG-AI-029 | Password / complexity | Letters-only password | {"name":"User 029","email":"hw06.reg029@example.com","password":"abcdefgh"} | 4xx | Request rejected if complexity requires multiple character classes | Complexity not specified; must audit |
| REG-AI-030 | Password / complexity | Digits-only password | {"name":"User 030","email":"hw06.reg030@example.com","password":"12345678"} | 4xx | Request rejected if complexity requires multiple character classes | Complexity not specified; must audit |
| REG-AI-031 | Password / complexity | Special-characters-only password | {"name":"User 031","email":"hw06.reg031@example.com","password":"!@#$%^&*"} | 4xx | Request rejected if complexity requires multiple character classes | Complexity not specified; must audit |
| REG-AI-032 | Password / valid partition | Unicode password | {"name":"User 032","email":"hw06.reg032@example.com","password":"MậtKhẩu123!"} | 200 or 4xx | Behavior should be deterministic and documented | Character-set restriction not specified |
| REG-AI-033 | Password / boundary | Extremely long password (~5000 chars) | {"name":"User 033","email":"hw06.reg033@example.com","password":"<5000-character string>"} | 4xx or bounded acceptance | Server handles oversized input safely without crash | Maximum length not specified; robustness case |
| REG-AI-034 | Body / required fields | Empty JSON object | {} | 4xx | Request rejected because required registration data is absent | Required-field robustness |
| REG-AI-035 | Body / parser robustness | Malformed JSON | {"name":"User 035","email":"hw06.reg035@example.com","password":"Password123!" | 400 | Request rejected as malformed JSON; server remains available | JSON parser robustness |
| REG-AI-036 | Body / type robustness | JSON array instead of object | [{"name":"User 036","email":"hw06.reg036@example.com","password":"Password123!"}] | 4xx | Request rejected because request body shape is invalid | Schema/body-shape validation |
| REG-AI-037 | Security / mass assignment | Attempt to self-register as admin by adding role | {"name":"User 037","email":"hw06.reg037@example.com","password":"Password123!","role":"admin"} | 200 | Account may be created, but supplied role must be ignored and privilege must remain normal user | Role escalation / mass assignment |
| REG-AI-038 | Security / mass assignment | Attempt to control primary key with id field | {"id":999999,"name":"User 038","email":"hw06.reg038@example.com","password":"Password123!"} | 200 | Account may be created, but client-supplied id must not control generated user id | Mass assignment / server-owned fields |
| REG-AI-039 | Security / SQL injection | SQL injection payload in email | {"name":"User 039","email":"x@example.com' OR 1=1 --","password":"Password123!"} | 4xx or safe literal handling | Payload must not alter SQL semantics, expose data, or bypass registration rules | SQL injection |
| REG-AI-040 | Security / stored content | Script payload in name | {"name":"<script>alert(1)</script>","email":"hw06.reg040@example.com","password":"Password123!"} | 200 or 4xx | Input must be handled as inert data; no script execution or server failure | Stored-XSS-oriented input handling |
| REG-AI-041 | State / duplicate | Register the same email twice | Step 1: {"name":"First","email":"hw06.duplicate@example.com","password":"Password123!"}; Step 2: same email again | 4xx on second request | Second registration should not create another account for the same email | Duplicate-account state behavior; uniqueness must be audited |
| REG-AI-042 | Response schema / security | Verify success response exposes only documented fields | {"name":"User 042","email":"hw06.reg042@example.com","password":"Password123!"} | 200 | Response contains documented message and id and does not echo password or sensitive/internal fields | Exact response shape / information exposure |

## Audit columns to add in the next stage

For every case, the human-review artifact will record:

- `VALID / INVALID / INCOMPLETE`
- audit reasoning
- corrected/final expected result
- whether the case should remain in the executable suite

Cases whose requirements are not explicitly stated by the supplied specification (for example password complexity, maximum lengths, email uniqueness, or trimming) must **not** be silently treated as specification facts during audit.