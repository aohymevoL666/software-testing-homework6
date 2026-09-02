# HW06 — API 2 Manual Execution Case

## CART-HUM-006 — Cart state across backend restart

This case is intentionally executed manually because Postman/Newman cannot restart the Node.js SUT process by itself.

### Preconditions
1. Backend running on `http://localhost:3000`.
2. A valid user account/token is available.
3. Add a unique marker item to the authenticated user's cart.
4. Confirm the marker exists with `GET /api/cart`.

### Procedure
1. Stop the backend process (`Ctrl+C` in the backend terminal).
2. Restart it with:
   ```bash
   node server.js
   ```
3. Login again if a new token is needed.
4. Call `GET /api/cart` using the same user account.

### Expected / oracle
Record whether the cart marker survives the backend restart.

The reviewed implementation stores carts in the in-process `userCarts` object, so loss after restart is expected implementation behavior. The supplied API specification does not explicitly require cart persistence, therefore this observation must **not** be reported as a specification defect unless another SUT requirement explicitly requires persistent cart state.

### Evidence to capture
One screenshot before restart showing the marker in `GET /api/cart`, and one after restart showing the resulting cart state.
