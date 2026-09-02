# HW06 — API 2 Execution Summary

**Feature:** FR-07 Shopping Cart  
**Primary endpoint:** `POST /api/cart`  
**Tool:** Postman + Newman 6.2.2, plus one manual restart-state check  
**Environment:** `http://localhost:3000`  
**Required student header:** `X-Student-Id: 23127531`

## Automated Newman run

| Metric | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 62 | 0 |
| Test scripts | 62 | 0 |
| Pre-request scripts | 64 | 0 |
| Assertions | 109 | 0 |

- Total duration: **5.5 s**
- Average response time: **2 ms**
- Minimum response time: **1 ms**
- Maximum response time: **21 ms**
- Failed assertions: **0**

## Automated observations

The automated run confirmed:

- Valid authenticated cart additions return `200 OK` with `{"message":"Added to cart"}`.
- Missing token returns `401 Unauthorized`.
- Malformed or tampered JWT returns `403 Forbidden`.
- Malformed JSON returns `400 Bad Request`.
- The current implementation accepts zero/negative quantities, zero/negative prices, missing fields, non-existing product ids, array bodies, nested values, and arbitrary extra fields.
- Repeated additions of the same product are stored as multiple entries rather than merged.
- Two-user cart isolation passed in both directions.
- Contradictory client-supplied metadata for the same product id can coexist in the cart.
- Arbitrary extra fields can be persisted in a cart item, but they do not override JWT-based cart ownership.
- A previously issued valid JWT remains usable after non-id profile changes.

These permissive behaviors are documented as characterization results unless a separate requirement explicitly defines stricter behavior.

## Manual case — CART-HUM-006

### Objective

Characterize cart state across backend restart.

### Before restart

Using the seeded user `test@eshop.com`, a marker item was added:

```json
{"id":990006,"name":"RESTART_MARKER","price":606,"quantity":1}
```

`GET /api/cart` returned:

```json
[{"id":990006,"name":"RESTART_MARKER","price":606,"quantity":1}]
```

### After restart

The backend was restarted and the same seeded account logged in again.

`GET /api/cart` returned:

```json
[]
```

### Human classification

**PASS as a characterization test.**

The reviewed implementation stores carts in the in-process `userCarts` object, so cart contents are lost when the Node.js process restarts. The supplied API specification does not explicitly require cart persistence across server restarts; therefore this behavior is **not reported as a confirmed specification bug**.

## Bug decision

**Confirmed API 2 bugs: 0**

Potential production-quality concerns such as trusting client-supplied price/product metadata or volatile cart storage are not promoted to genuine defects because the supplied specification does not provide a requirement that they violate.

## API 2 pipeline status

| Stage | Status |
|---|---|
| AI Generate | Complete — 42 cases |
| Human Audit | Complete — 42 audited |
| Human Extend | Complete — 6 added cases |
| Postman implementation | Complete |
| Newman automated execution | Complete — 109/109 assertions passed |
| Manual restart-state execution | Complete |
| Confirmed bugs | 0 |
| Newman HTML report | Generated |
