# HW06 — API 2 Human Audit

**Feature:** FR-07 Shopping Cart  
**Endpoint:** `POST /api/cart`  
**Audited AI cases:** 42  

## Audit basis

The supplied API specification defines `POST /api/cart`, requires `Authorization: Bearer <token>`, and illustrates a JSON body with `id`, `name`, `price`, and `quantity`.

Human review also inspected the SUT implementation. The route derives `userId` from the verified JWT, initializes `userCarts[userId]`, then directly executes `userCarts[userId].push(req.body)` and returns `{message: "Added to cart"}`. There is no database cart table and no route-level validation of product existence, field types, price, quantity, duplicate items, or unexpected fields.

**Audit rule:** implementation behavior is not automatically a requirement. Where the supplied specification does not define a validation/business rule, the audit retains useful cases as robustness/security characterization rather than fabricating a rejection oracle.

## Summary

- VALID: **13**
- INVALID: **2**
- INCOMPLETE: **27**
- Total: **42**

## Case-by-case audit

| ID | Audit Label | Human Review Reasoning | Corrected / Final Oracle |
|---|---|---|---|
| CART-AI-001 | VALID | The API spec defines `POST /api/cart`, Bearer authentication, the four illustrated cart fields, and a successful add operation. | Valid token + documented body → 200 with `message: Added to cart`. |
| CART-AI-002 | VALID | Representative positive quantity using the documented body shape; no conflicting rule exists. | 200 with documented success message. |
| CART-AI-003 | VALID | Quantity 1 is a normal positive-domain case and the implementation accepts it. | 200 with documented success message. |
| CART-AI-004 | INCOMPLETE | The partition is useful, but the AI invented the rule `quantity > 0`. The supplied API spec does not state that rule and the route performs no validation. | Retain as characterization. Current implementation is expected to accept quantity 0 and return 200; do not call that a spec defect from this document alone. |
| CART-AI-005 | INCOMPLETE | Negative quantity is a useful business-rule probe, but no rejection rule is supplied or implemented. | Retain as characterization; current implementation is expected to return 200. |
| CART-AI-006 | INCOMPLETE | The spec example shows a number, but no formal type-validation response is defined; the route stores `req.body` as-is. | Retain as robustness/type characterization; do not assert 4xx from the supplied spec. |
| CART-AI-007 | INCOMPLETE | Integer-only quantity is not specified. | Retain as type/boundary characterization; current implementation should accept and return 200. |
| CART-AI-008 | INCOMPLETE | Null quantity rejection is not specified and no validation exists. | Retain as characterization; current implementation should return 200. |
| CART-AI-009 | INCOMPLETE | The example contains `quantity`, but the supplied spec does not formally declare required properties or rejection behavior. | Retain as characterization; do not invent a 4xx requirement. |
| CART-AI-010 | INCOMPLETE | The AI used an ambiguous `4xx or bounded acceptance` oracle and no upper quantity limit is specified. | Retain as robustness boundary; assert safe handling/no crash, not an invented maximum. |
| CART-AI-011 | VALID | Normal positive price using the documented body shape. | 200 with documented success message. |
| CART-AI-012 | INCOMPLETE | The AI invented `price > 0`. No such rule appears in the supplied spec and the route performs no price validation. | Retain as characterization; current implementation is expected to return 200. |
| CART-AI-013 | INCOMPLETE | Negative-price rejection is not specified or implemented. | Retain as business-logic characterization; current implementation should return 200. |
| CART-AI-014 | INCOMPLETE | The spec example uses a numeric price but does not define type validation or a 4xx response. | Retain as type robustness; characterize actual behavior. |
| CART-AI-015 | INCOMPLETE | The AI supplied two possible outcomes and the spec does not state whether prices must be integer-only. | Retain as characterization; current implementation stores the decimal value and returns 200. |
| CART-AI-016 | INCOMPLETE | Null-price rejection is not specified and no validation exists. | Retain as characterization; current implementation should return 200. |
| CART-AI-017 | INCOMPLETE | Missing-price rejection is not formally defined in the supplied spec. | Retain as characterization; current implementation should accept the partial body and return 200. |
| CART-AI-018 | INCOMPLETE | Price tampering is a valuable security/business-logic probe, but the spec does not say that the server must look up canonical product pricing. | Retain as security characterization. Observe whether client-supplied price is stored unchanged; treat any defect claim as a requirement/security finding that needs explicit justification. |
| CART-AI-019 | VALID | Existing product id with the documented body shape is a normal positive case. | 200 with documented success message. |
| CART-AI-020 | INCOMPLETE | The AI assumed `id=0` must be rejected as an invalid product. The route never verifies product existence and the spec does not define this rule. | Retain as characterization; current implementation is expected to return 200. |
| CART-AI-021 | INCOMPLETE | Negative product-id rejection is not specified or implemented. | Retain as characterization; current implementation should return 200. |
| CART-AI-022 | INVALID | The AI invented a resource-existence requirement and `404` oracle for cart addition. The supplied spec does not state product lookup/validation for this endpoint. | Reframe as business-logic characterization. Current implementation is expected to accept the non-existing id and return 200. |
| CART-AI-023 | INCOMPLETE | Numeric-only id validation is not defined in the supplied spec. | Retain as type robustness; current implementation should store the string id and return 200. |
| CART-AI-024 | INCOMPLETE | Null-id rejection is not specified. | Retain as characterization; current implementation should return 200. |
| CART-AI-025 | INCOMPLETE | Missing-id rejection is not formally specified. | Retain as characterization; current implementation should return 200. |
| CART-AI-026 | INCOMPLETE | Unicode handling is useful, but the AI mixed it with an unsupported assumption about canonical name consistency. | Retain the Unicode aspect; current implementation should return 200 and store the supplied name literally. |
| CART-AI-027 | INCOMPLETE | Non-empty product-name validation is not stated or implemented. | Retain as characterization; current implementation should return 200. |
| CART-AI-028 | INCOMPLETE | Whitespace-name rejection is not stated or implemented. | Retain as characterization; current implementation should return 200. |
| CART-AI-029 | INCOMPLETE | Missing-name rejection is not formally stated in the supplied spec. | Retain as characterization; current implementation should return 200. |
| CART-AI-030 | INCOMPLETE | The tampering objective is valuable, but the spec does not require canonical product metadata lookup. The route trusts the submitted body. | Retain as business-logic/security characterization; observe literal storage and avoid inventing a 4xx oracle. |
| CART-AI-031 | VALID | The middleware explicitly returns 401 when no token is present. | 401 with `{error: "Unauthorized"}`. |
| CART-AI-032 | VALID | `Authorization: Bearer` produces no usable second token; middleware follows its missing-token path. | 401 with `{error: "Unauthorized"}`. |
| CART-AI-033 | VALID | Malformed random JWT reaches `jwt.verify` and the middleware explicitly returns 403 on verification error. | 403 with `{error: "Forbidden"}`. |
| CART-AI-034 | VALID | JWT signature tampering should fail verification and maps directly to the middleware's 403 path. | 403 with `{error: "Forbidden"}`; cart must not be modified. |
| CART-AI-035 | VALID | With `Authorization: Basic abc`, the middleware still extracts `abc` as the token and `jwt.verify` fails. | 403 with `{error: "Forbidden"}`. Note that the middleware does not validate the Bearer scheme explicitly. |
| CART-AI-036 | INCOMPLETE | The AI assumed an empty object must be rejected. The route pushes any body and the spec has no explicit required-field/rejection rule. | Retain as characterization; current implementation is expected to return 200. |
| CART-AI-037 | VALID | Malformed JSON is rejected by the JSON parser before normal route processing; this is a valid protocol robustness case. | 400 parser error; server remains available. |
| CART-AI-038 | INCOMPLETE | The spec illustrates an object but does not define schema-validation behavior; the route does not validate shape. | Retain as body-shape characterization; current implementation is expected to accept the array and return 200. |
| CART-AI-039 | INVALID | The AI invented a requirement that repeated adds must merge/update quantity. The supplied spec does not define duplicate-item semantics, and the implementation simply pushes another item. | Correct characterization: both POSTs return 200 and two entries may exist. Do not report duplication as a spec bug without another requirement. |
| CART-AI-040 | VALID | Per-user isolation is a meaningful security/state property because the authenticated user id indexes each cart; another user's token must not expose User A's cart. | User A add succeeds; User B's cart state remains separate. Verify using helper `GET /api/cart`. |
| CART-AI-041 | INCOMPLETE | Unexpected-field probing is useful, but the AI assumed they must be rejected/ignored. The route stores the entire body. Extra `user_id`/`isAdmin` fields do not control authenticated ownership because ownership comes from `req.user.id`, but they can still persist as arbitrary item data. | Retain as mass-assignment/data-integrity characterization. Assert that extra fields cannot change actual cart ownership or authentication privilege. |
| CART-AI-042 | VALID | HW06 requires schema validation, and the documented successful add response is the message `Added to cart`. | 200; response contains exactly the documented success message and no sensitive authentication data. |

## Main AI errors found

1. **Invented cart validation rules.** The AI repeatedly assumed positive-only quantity/price, required fields, numeric types, and existing product ids even though the supplied spec does not define those rejection rules.
2. **Invented duplicate-item semantics.** The AI assumed repeated adds must merge/update quantity, but the spec does not say this and the implementation appends another item.
3. **Ambiguous oracles.** Several generated cases used alternatives such as `200 or 4xx`, which are not executable final expectations.
4. **Mixed security objective with unsupported conformance rules.** Price/name tampering is worth testing, but the supplied spec does not explicitly say the server must canonicalize product metadata from the product database.

## Strong cases retained for execution

- Happy-path cart additions and exact response-schema checks.
- Missing, malformed, and tampered authentication tokens.
- Malformed JSON handling.
- User-cart isolation using two independently authenticated accounts.
- Price/name tampering and unexpected fields as security/business-logic characterization.
- Zero/negative/missing/type-invalid fields as robustness characterization rather than invented 4xx requirements.

## Human-audit conclusion

The AI achieved broad parameter coverage but again over-assumed business rules that were not stated by the specification. The corrected suite separates **documented authentication/response behavior** from **characterization of the SUT's permissive cart implementation**, which is essential to avoid reporting unsupported validation expectations as genuine specification bugs.