# Bug Report — Registration returns HTTP 500 for unsupported or missing Content-Type

## Summary

`POST /api/register` returns `500 Internal Server Error` when the request body is not parsed as JSON because the request uses `Content-Type: text/plain` or omits the `Content-Type` header.

## Endpoint

`POST http://localhost:3000/api/register`

## Environment

- EShop SUT
- Local backend: `http://localhost:3000`
- Newman: 6.2.2
- Required student header: `X-Student-Id: 23127531`

## Severity

**Medium — robustness / error handling**

The endpoint is not required to accept unsupported media types, but malformed or unsupported client requests should not result in an internal server error.

## Preconditions

Backend is running on `localhost:3000`.

## Reproduction A — text/plain

Headers:

```http
Content-Type: text/plain
X-Student-Id: 23127531
```

Body:

```json
{"name":"Human 001","email":"hw06.hum001@example.com","password":"Password123!"}
```

### Actual result

`500 Internal Server Error`

### Expected result

The server should reject the request safely with a client-error response such as `400 Bad Request` or `415 Unsupported Media Type`, and should not expose an internal server failure.

## Reproduction B — missing Content-Type

Headers:

```http
X-Student-Id: 23127531
```

Body:

```json
{"name":"Human 002","email":"hw06.hum002@example.com","password":"Password123!"}
```

### Actual result

`500 Internal Server Error`

### Expected result

The server should reject the request safely as a client error and must not return 5xx.

## Evidence

Newman execution:

- `REG-HUM-001 - text/plain media type` → `500 Internal Server Error`
- `REG-HUM-002 - missing Content-Type` → `500 Internal Server Error`
- 134 assertions executed
- 2 assertions failed

Attach:
1. Screenshot of the Newman failure output.
2. Screenshot showing the `X-Student-Id: 23127531` pre-request-script evidence / Postman Console.
3. Link or screenshot of the Newman HTML report if useful.

## Technical analysis

The registration handler immediately destructures the request body:

```js
const { name, email, password } = req.body;
```

For requests that are not parsed by the JSON middleware, `req.body` may be undefined. The handler does not guard against this condition or return a controlled client-error response.

## Suggested fix

Validate that the request body exists and is a JSON object before destructuring it, and return a controlled 4xx response for unsupported or invalid request bodies. Optionally enforce `application/json` for this endpoint and return `415 Unsupported Media Type` for unsupported content types.

## Related test cases

- `REG-HUM-001`
- `REG-HUM-002`
