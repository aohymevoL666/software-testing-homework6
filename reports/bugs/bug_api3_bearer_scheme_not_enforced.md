# Bug Report — Bearer authentication scheme is not enforced

## Summary

The authentication middleware accepts a valid JWT even when it is supplied using the `Basic` scheme instead of the documented `Bearer` scheme.

## Endpoint

`GET http://localhost:3000/api/admin/users`

## Severity

**Medium — Authentication Protocol Validation**

## Steps to reproduce

1. Login as the seeded Admin account and obtain a valid Admin JWT.
2. Send:

```http
GET /api/admin/users
Authorization: Basic <valid-admin-jwt>
X-Student-Id: 23127531
```

## Expected result

The request is rejected because the API specification requires:

```http
Authorization: Bearer <token>
```

## Actual result

The request returns `200 OK` and the Admin user list.

## Evidence from Newman

`ADMIN-HUM-004 - valid admin JWT under Basic scheme`

- Actual status: `200 OK`
- Failed assertion: `Documented Bearer scheme is enforced`

## Root cause

The middleware extracts the second whitespace-separated value but never validates the authentication scheme:

```js
const authHeader = req.headers["authorization"];
const token = authHeader && authHeader.split(" ")[1];
```

Therefore any scheme label can work if the second value is a valid JWT.

## Suggested fix

Parse the Authorization header explicitly and require:

```text
scheme === "Bearer"
```

before calling `jwt.verify`.

## Related test cases

`ADMIN-HUM-004`, `ADMIN-EXEC-002`
