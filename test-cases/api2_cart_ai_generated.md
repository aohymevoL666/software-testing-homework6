# HW06 — API 2 AI-Generated Test Cases

**Feature:** FR-07 Shopping Cart  
**Endpoint:** `POST /api/cart`  
**Authentication:** `Authorization: Bearer <token>`  
**Execution header required later:** `X-Student-Id: 23127531`  

> This file preserves the **AI-generation stage**. Expected results are AI draft expectations and have not yet been human-audited. Some assumptions are intentionally left visible so that the required audit stage can distinguish specification facts from generic API-testing expectations.

## Partition map

| Input / Aspect | Partitions considered |
|---|---|
| `id` | existing, zero, negative, non-existing, string, null, missing |
| `name` | normal, Unicode, empty, whitespace, missing, manipulated metadata |
| `price` | positive, zero, negative, string, decimal, null, missing, tampered low price |
| `quantity` | positive, zero, negative, string, decimal, null, missing, very large |
| Authentication | valid token, missing token, empty Bearer, malformed JWT, tampered JWT, wrong scheme |
| Request body | normal object, empty object, malformed JSON, array, unexpected fields |
| State | repeated add, per-user cart isolation |
| Security | JWT integrity, price/name tampering, unexpected ownership/privilege fields |
| Schema | exact success response |

**AI-generated cases:** 42

## Test cases

| ID | Category | Objective | Request / Test Data | Authentication | AI Draft Expected HTTP | AI Draft Expected Result | Coverage Basis |
|---|---|---|---|---|---|---|---|
| CART-AI-001 | Positive / baseline | Add a typical valid cart item with an authenticated user | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Valid user token | 200 | Response contains `message: Added to cart` | API specification happy path |
| CART-AI-002 | Quantity / valid partition | Add quantity = 2 | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":2} | Valid user token | 200 | Item is added successfully | Representative positive quantity |
| CART-AI-003 | Quantity / boundary | Add quantity = 1 | {"id":2,"name":"Samsung Galaxy S24 Ultra","price":28000000,"quantity":1} | Valid user token | 200 | Item is added successfully | Lower positive boundary candidate |
| CART-AI-004 | Quantity / invalid partition | Add quantity = 0 | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":0} | Valid user token | 4xx | Request is rejected because quantity must be positive | AI assumes cart quantity > 0 |
| CART-AI-005 | Quantity / invalid partition | Add negative quantity | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":-1} | Valid user token | 4xx | Request is rejected because quantity must be positive | Negative domain partition |
| CART-AI-006 | Quantity / type | Quantity as numeric string | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":"2"} | Valid user token | 4xx | Request is rejected because quantity must be numeric | Type partition |
| CART-AI-007 | Quantity / type | Quantity as decimal | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1.5} | Valid user token | 4xx | Request is rejected because quantity should be an integer | Type/boundary partition |
| CART-AI-008 | Quantity / null | quantity = null | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":null} | Valid user token | 4xx | Request is rejected | Null partition |
| CART-AI-009 | Quantity / missing | Omit quantity | {"id":1,"name":"iPhone 15 Pro Max","price":30000000} | Valid user token | 4xx | Request is rejected | Required-field partition |
| CART-AI-010 | Quantity / boundary | Extremely large quantity | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":2147483647} | Valid user token | 4xx or bounded acceptance | Server handles upper-bound input safely | Robustness boundary |
| CART-AI-011 | Price / valid partition | Typical positive integer price | {"id":3,"name":"MacBook Pro M3","price":45000000,"quantity":1} | Valid user token | 200 | Item is added successfully | Normal price domain |
| CART-AI-012 | Price / boundary | price = 0 | {"id":1,"name":"iPhone 15 Pro Max","price":0,"quantity":1} | Valid user token | 4xx | Request is rejected because price must be greater than 0 | AI assumes positive price requirement |
| CART-AI-013 | Price / invalid partition | Negative price | {"id":1,"name":"iPhone 15 Pro Max","price":-1000,"quantity":1} | Valid user token | 4xx | Request is rejected | Negative domain partition |
| CART-AI-014 | Price / type | Price as numeric string | {"id":1,"name":"iPhone 15 Pro Max","price":"30000000","quantity":1} | Valid user token | 4xx | Request is rejected because price must be numeric | Type partition |
| CART-AI-015 | Price / type | Price as decimal | {"id":1,"name":"iPhone 15 Pro Max","price":29999.99,"quantity":1} | Valid user token | 200 or 4xx | Behavior follows numeric-price rule and is documented | Decimal partition |
| CART-AI-016 | Price / null | price = null | {"id":1,"name":"iPhone 15 Pro Max","price":null,"quantity":1} | Valid user token | 4xx | Request is rejected | Null partition |
| CART-AI-017 | Price / missing | Omit price | {"id":1,"name":"iPhone 15 Pro Max","quantity":1} | Valid user token | 4xx | Request is rejected | Required-field partition |
| CART-AI-018 | Security / price tampering | Use real product id/name but submit a much lower client price | {"id":1,"name":"iPhone 15 Pro Max","price":1,"quantity":1} | Valid user token | 4xx | Server rejects manipulated price or uses canonical product price | Business-logic tampering |
| CART-AI-019 | Product id / valid partition | Use an existing product id | {"id":2,"name":"Samsung Galaxy S24 Ultra","price":28000000,"quantity":1} | Valid user token | 200 | Item is added successfully | Existing resource |
| CART-AI-020 | Product id / invalid partition | Use product id = 0 | {"id":0,"name":"Unknown","price":1000,"quantity":1} | Valid user token | 4xx | Request is rejected as invalid product | ID boundary |
| CART-AI-021 | Product id / invalid partition | Use negative product id | {"id":-1,"name":"Unknown","price":1000,"quantity":1} | Valid user token | 4xx | Request is rejected as invalid product | Negative ID |
| CART-AI-022 | Product id / state | Use a non-existing large product id | {"id":999999,"name":"Ghost Product","price":1000,"quantity":1} | Valid user token | 404 or 4xx | Request is rejected because product does not exist | Resource-existence validation |
| CART-AI-023 | Product id / type | Use string product id | {"id":"1","name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected because id should be numeric | Type partition |
| CART-AI-024 | Product id / null | id = null | {"id":null,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected | Null partition |
| CART-AI-025 | Product id / missing | Omit id | {"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected | Required-field partition |
| CART-AI-026 | Name / valid partition | Vietnamese Unicode product name | {"id":1,"name":"Điện thoại cao cấp","price":30000000,"quantity":1} | Valid user token | 200 or 4xx | Name is safely handled and canonical consistency is preserved | Unicode text |
| CART-AI-027 | Name / invalid partition | Empty product name | {"id":1,"name":"","price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected | AI assumes non-empty name |
| CART-AI-028 | Name / invalid partition | Whitespace-only product name | {"id":1,"name":"   ","price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected | Blank-string partition |
| CART-AI-029 | Name / missing | Omit name | {"id":1,"price":30000000,"quantity":1} | Valid user token | 4xx | Request is rejected | Required-field partition |
| CART-AI-030 | Security / product tampering | Use a real product id but falsify the product name | {"id":1,"name":"Attacker Controlled Name","price":30000000,"quantity":1} | Valid user token | 4xx or canonicalized | Server must not trust manipulated product metadata | Business-logic integrity |
| CART-AI-031 | Authentication | No Authorization header | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | No token | 401 | Response contains Unauthorized error | authenticateToken missing-token path |
| CART-AI-032 | Authentication | Authorization header contains `Bearer` but no token | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | `Authorization: Bearer` | 401 | Response contains Unauthorized error | Missing token after scheme |
| CART-AI-033 | Authentication | Malformed random bearer token | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | `Bearer not-a-jwt` | 403 | Response contains Forbidden error | Invalid JWT path |
| CART-AI-034 | Authentication / security | Tamper one character of an otherwise valid JWT | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Tampered bearer token | 403 | Request is forbidden; cart is not modified | JWT integrity |
| CART-AI-035 | Authentication / scheme | Use Basic authentication instead of Bearer | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | `Authorization: Basic abc` | 403 | Request is forbidden | Authentication scheme robustness |
| CART-AI-036 | Body / robustness | Empty JSON object | {} | Valid user token | 4xx | Request is rejected due to missing cart fields | Body schema |
| CART-AI-037 | Body / parser robustness | Malformed JSON | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1 | Valid user token | 400 | Malformed JSON is rejected; server remains available | JSON parser robustness |
| CART-AI-038 | Body / shape | JSON array instead of object | [{"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1}] | Valid user token | 4xx | Request is rejected because body shape is invalid | Schema/body shape |
| CART-AI-039 | State transition | Add the same product twice to the same user's cart | Two consecutive valid POST requests for product id 1 | Valid user token | 200 for both | Cart should merge/update quantity rather than create inconsistent duplicates | Repeated-add cart state |
| CART-AI-040 | State / user isolation | User A adds an item, then User B reads/uses their cart | User A POSTs product id 1; User B uses separate token | Two valid user tokens | 200 | User B must not gain access to User A's cart contents | Per-user state isolation |
| CART-AI-041 | Security / mass assignment | Add unexpected server-like fields in the cart item | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1,"user_id":999,"isAdmin":true} | Valid user token | 200 or 4xx | Unexpected fields must not change authenticated ownership or privileges | Mass assignment / ownership |
| CART-AI-042 | Response schema | Verify exact successful response schema | {"id":1,"name":"iPhone 15 Pro Max","price":30000000,"quantity":1} | Valid user token | 200 | Response exactly matches documented success message `Added to cart` without sensitive fields | Schema validation |

## Notes for the Human Audit stage

The supplied implementation shows that the authenticated user's `id` selects an in-memory cart, and the route currently pushes `req.body` directly into that cart without validating product existence, fields, price, quantity, or duplicate state. These implementation facts must be used carefully during audit: implementation behavior is evidence about the SUT, but it does not automatically create requirements that are absent from the API specification.

The audit should pay special attention to AI assumptions about positive price/quantity, canonical product metadata, product existence, required fields, and duplicate-item merging.