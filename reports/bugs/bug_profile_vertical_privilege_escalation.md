# Bug Report — Normal user can self-promote to Admin through profile update

## Summary

`PUT /api/users/me` accepts a client-supplied `role` field. A normal user can set `role:"admin"`, re-login, and receive an Admin role in the newly issued JWT.

## Endpoint

Primary vulnerable endpoint:

`PUT http://localhost:3000/api/users/me`

Impact verification endpoint:

`GET http://localhost:3000/api/admin/users`

## Severity

**Critical/High — Vertical Privilege Escalation**

## Preconditions

A normal user is authenticated.

## Steps to reproduce

1. Login as a normal user.
2. Send:

```http
PUT /api/users/me
Authorization: Bearer <normal-user-jwt>
Content-Type: application/json
X-Student-Id: 23127531
```

Body:

```json
{
  "name": "HW06 Access User",
  "shipping_address": "Test Address",
  "phone": "0900000000",
  "role": "admin"
}
```

3. Login again with the same account.
4. Inspect the returned user role / new JWT.
5. Access `GET /api/admin/users`.

## Expected result

The personal-profile endpoint must not allow a normal user to change a security-sensitive role. The user remains role `user`.

## Actual result

- Profile update returns `200 OK`.
- Re-login returns role `admin`.
- The new token can access the admin user list.

## Evidence from Newman

`ADMIN-HUM-002A`
- Profile update: `200 OK`

`ADMIN-HUM-002B`
- Re-login: `200 OK`
- Failed assertion: expected role not to equal `admin`, but actual role was `admin`

`ADMIN-HUM-002C`
- Admin user-list request: `200 OK`

## Root cause

The profile update handler accepts `role` directly from the client:

```js
const { name, shipping_address, phone, role } = req.body;

if (role) {
  query += ", role = ?";
  params.push(role);
}
```

This contradicts the API specification description that the personal-profile endpoint only allows basic personal information to be updated.

## Suggested fix

Remove `role` from the user-controlled profile-update fields. Role changes should only be possible through a protected Admin-only endpoint with explicit authorization.

## Related test cases

`ADMIN-HUM-002A`, `ADMIN-HUM-002B`, `ADMIN-HUM-002C`
