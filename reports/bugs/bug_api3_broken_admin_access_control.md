# Bug Report — Non-admin users can access the Admin user list

## Summary

A normal authenticated user can access `GET /api/admin/users` and receive the global user list even though the API specification states that Admin APIs require an Admin account.

## Endpoint

`GET http://localhost:3000/api/admin/users`

## Severity

**High — Broken Access Control / Authorization Bypass**

## Preconditions

1. Backend is running.
2. A normal non-admin user account exists.
3. The normal user logs in and obtains a valid JWT.

## Steps to reproduce

1. Login as a normal user.
2. Send:

```http
GET /api/admin/users
Authorization: Bearer <normal-user-jwt>
X-Student-Id: 23127531
```

## Expected result

The request is denied with a clear authorization error such as `403 Forbidden`, and no global user data is returned.

## Actual result

The server returns `200 OK` with a JSON array containing user records.

## Evidence from Newman

The following tests all received `200 OK` and failed their authorization assertions:

- `ADMIN-AI-002`
- `ADMIN-AI-017`
- `ADMIN-AI-029`
- `ADMIN-AI-035`
- `ADMIN-HUM-001`
- `ADMIN-HUM-003`

The response was confirmed to be an array of global user objects.

## Root cause

The route uses authentication middleware only:

```js
app.get("/api/admin/users", authenticateToken, (req, res) => {
```

There is no Admin role check before returning the user list.

## Suggested fix

Add authorization middleware or an explicit role check after JWT authentication, for example:

```js
if (req.user.role !== "admin") {
  return res.status(403).json({ error: "Forbidden" });
}
```

Prefer a reusable Admin-authorization middleware for all `/api/admin/...` endpoints.

## Related test cases

`ADMIN-AI-002`, `ADMIN-AI-017`, `ADMIN-AI-029`, `ADMIN-AI-035`, `ADMIN-HUM-001`, `ADMIN-HUM-003`
