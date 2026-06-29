# API Reference

Base URL: `http://localhost:8080` (dev) or your production URL.

## Authentication

Cookie-based. On login/register, the server sets two `HttpOnly` cookies:

| Cookie | TTL | Purpose |
|---|---|---|
| `access_token` | 15 min | Authenticates requests |
| `refresh_token` | 7 days | Rotates tokens via `/auth/refresh` |

**All protected endpoints** require the `access_token` cookie. Browsers send cookies automatically. For curl, store and send cookies with `-c`/`-b`.

---

## Auth — Public

### Register

Creates a new user account. Sets auth cookies on success.

```
POST /api/v1/auth/register
```

**Request body:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

**Responses:**

| Status | Body |
|---|---|
| `201 Created` | `{"message":"Registration successful"}` + sets cookies |
| `400 Bad Request` | `{"error":"invalid request body"}` |
| `409 Conflict` | `{"error":"email already exist"}` |

```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","password":"secret123"}' \
  -c cookies.txt
```

---

### Login

Authenticates an existing user. Sets auth cookies on success.

```
POST /api/v1/auth/login
```

**Request body:**

```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

**Responses:**

| Status | Body |
|---|---|
| `201 Created` | `{"message":"Login successful"}` + sets cookies |
| `400 Bad Request` | `{"error":"invalid request body"}` |
| `401 Unauthorized` | `{"error":"user does not exist"}` |

```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"secret123"}' \
  -c cookies.txt
```

---

### Logout

Revokes the refresh token and clears cookies.

```
POST /api/v1/auth/logout
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"Logged out successfully"}` + clears cookies |

```bash
curl -X POST http://localhost:8080/api/v1/auth/logout \
  -b cookies.txt -c cookies.txt
```

---

### Refresh Token

Rotates tokens using the existing refresh token. Sets new cookies.

```
POST /api/v1/auth/refresh
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"Token refreshed successfully"}` + new cookies |
| `401 Unauthorized` | `{"error":"unauthorized"}` |

```bash
curl -X POST http://localhost:8080/api/v1/auth/refresh \
  -b cookies.txt -c cookies.txt
```

---

### Verify Email

Verifies a user's email address using a token sent by email.

```
POST /api/v1/auth/verify-email?token=<verification_token>
```

**Query params:**

| Param | Description |
|---|---|
| `token` | Verification token from email |

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"verification successfull"}` |
| `400 Bad Request` | `{"error":"missing token"}` |
| `500 Internal Server Error` | `{"error":"Verification failed"}` |

```bash
curl -X POST "http://localhost:8080/api/v1/auth/verify-email?token=abc123"
```

---

### Forgot Password

Sends a password reset email to the given address.

```
POST /api/v1/auth/forgot-password
```

**Request body:**

```json
{
  "email": "john@example.com"
}
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"forgot password email successfully sended"}` |

```bash
curl -X POST http://localhost:8080/api/v1/auth/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com"}'
```

---

### Reset Password

Resets the password using a token from the forgot-password email.

```
POST /api/v1/auth/reset-password?token=<reset_token>
```

**Query params:**

| Param | Description |
|---|---|
| `token` | Reset token from email |

**Request body:**

```json
{
  "password": "newpassword456"
}
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"password reset successfull"}` |
| `400 Bad Request` | `{"error":"missing token"}` or `{"error":"invalid JSON"}` |
| `500 Internal Server Error` | `{"error":"password reset failed"}` |

```bash
curl -X POST "http://localhost:8080/api/v1/auth/reset-password?token=abc123" \
  -H "Content-Type: application/json" \
  -d '{"password":"newpassword456"}'
```

---

## Auth — Protected (requires `access_token` cookie)

### Get Current User

Returns the authenticated user's profile.

```
GET /api/v1/auth/me
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"id":1,"name":"John Doe","email":"john@example.com","is_email_verified":false,"created_at":"2026-06-29T14:00:00Z"}` |
| `500 Internal Server Error` | `{"error":"internal server error"}` |

```bash
curl http://localhost:8080/api/v1/auth/me -b cookies.txt
```

---

### Delete Current User

Deletes the authenticated user's account.

```
DELETE /api/v1/auth/me
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"user deleted successfully"}` |

```bash
curl -X DELETE http://localhost:8080/api/v1/auth/me -b cookies.txt
```

---

## URLs — Protected (requires `access_token` cookie)

### Create Short URL

Creates a shortened URL from a long URL.

```
POST /api/v1/urls
```

**Request body:**

```json
{
  "longURL": "https://example.com/very/long/url"
}
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `"aB3xZ"` (raw JSON string — the short code) |
| `400 Bad Request` | `{"error":"invalid request body"}` or `{"error":"URL already exist"}` |

The full short URL is: `http://localhost:8080/aB3xZ`

```bash
curl -X POST http://localhost:8080/api/v1/urls \
  -H "Content-Type: application/json" \
  -d '{"longURL":"https://example.com/very/long/url"}' \
  -b cookies.txt
```

---

### List User URLs

Returns all URLs created by the authenticated user.

```
GET /api/v1/urls
```

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `[{"id":1,"short_code":"aB3xZ","long_url":"https://example.com","user_id":1,"created_at":"...","expires_at":null,"click_count":5}]` |

```bash
curl http://localhost:8080/api/v1/urls -b cookies.txt
```

---

### Get Single URL

Returns details for a specific short URL.

```
GET /api/v1/urls/{slug}
```

**Path params:**

| Param | Description |
|---|---|
| `slug` | Short code (e.g. `aB3xZ`) |

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"id":1,"short_code":"aB3xZ","long_url":"https://example.com","user_id":1,"created_at":"...","expires_at":null,"click_count":5}` |

```bash
curl http://localhost:8080/api/v1/urls/aB3xZ -b cookies.txt
```

---

### Delete Short URL

Deletes a short URL by its slug.

```
DELETE /api/v1/urls/{slug}
```

**Path params:**

| Param | Description |
|---|---|
| `slug` | Short code (e.g. `aB3xZ`) |

**Responses:**

| Status | Body |
|---|---|
| `200 OK` | `{"message":"long url deleted successfully"}` |
| `500 Internal Server Error` | `{"error":"URL does not exist"}` |

```bash
curl -X DELETE http://localhost:8080/api/v1/urls/aB3xZ -b cookies.txt
```

---

## Redirect — Public

### Resolve Short URL

Redirects (301) to the original long URL.

```
GET /{slug}
```

**Path params:**

| Param | Description |
|---|---|
| `slug` | Short code (e.g. `aB3xZ`) |

**Responses:**

| Status | Description |
|---|---|
| `301 Moved Permanently` | Redirects to the original URL |
| `404 Not Found` | Short code does not exist |

```bash
# Follow redirect with -L
curl -L http://localhost:8080/aB3xZ

# Or inspect the redirect target
curl -v http://localhost:8080/aB3xZ 2>&1 | grep Location
```

---

## Health

```
GET /health
```

**Response:** `200 OK` with body `OK`

```bash
curl http://localhost:8080/health
```

---

## Debug (pprof)

```
GET /debug/pprof/
GET /debug/pprof/profile
```

---

## OpenAPI 3.0 Spec

```yaml
openapi: "3.0.3"
info:
  title: URL Shortener API
  version: "1.0.0"
servers:
  - url: http://localhost:8080
paths:
  /api/v1/auth/register:
    post:
      summary: Register a new user
      tags: [Auth]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/UserRequest"
      responses:
        "201":
          description: Registration successful, cookies set
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/MessageResponse"
        "400":
          $ref: "#/components/responses/BadRequest"
        "409":
          description: Email already exists
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"

  /api/v1/auth/login:
    post:
      summary: Log in
      tags: [Auth]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
      responses:
        "201":
          description: Login successful, cookies set
        "401":
          $ref: "#/components/responses/Unauthorized"

  /api/v1/auth/logout:
    post:
      summary: Log out
      tags: [Auth]
      responses:
        "200":
          description: Logged out, cookies cleared

  /api/v1/auth/refresh:
    post:
      summary: Refresh access token
      tags: [Auth]
      responses:
        "200":
          description: Token refreshed, new cookies set
        "401":
          $ref: "#/components/responses/Unauthorized"

  /api/v1/auth/verify-email:
    post:
      summary: Verify email address
      tags: [Auth]
      parameters:
        - in: query
          name: token
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Email verified

  /api/v1/auth/forgot-password:
    post:
      summary: Request password reset email
      tags: [Auth]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                  format: email
      responses:
        "200":
          description: Password reset email sent

  /api/v1/auth/reset-password:
    post:
      summary: Reset password
      tags: [Auth]
      parameters:
        - in: query
          name: token
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                password:
                  type: string
      responses:
        "200":
          description: Password reset successful

  /api/v1/auth/me:
    get:
      summary: Get current user profile
      tags: [Auth]
      responses:
        "200":
          description: User profile
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserResponse"
    delete:
      summary: Delete current user
      tags: [Auth]
      responses:
        "200":
          description: User deleted

  /api/v1/urls:
    post:
      summary: Create a short URL
      tags: [URLs]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                longURL:
                  type: string
                  format: uri
      responses:
        "200":
          description: Short code string
          content:
            application/json:
              schema:
                type: string
    get:
      summary: List all user URLs
      tags: [URLs]
      responses:
        "200":
          description: Array of URLs
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/URLResponse"

  /api/v1/urls/{slug}:
    get:
      summary: Get a short URL by slug
      tags: [URLs]
      parameters:
        - in: path
          name: slug
          required: true
          schema:
            type: string
      responses:
        "200":
          description: URL details
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/URLResponse"
    delete:
      summary: Delete a short URL
      tags: [URLs]
      parameters:
        - in: path
          name: slug
          required: true
          schema:
            type: string
      responses:
        "200":
          description: URL deleted

  /{slug}:
    get:
      summary: Redirect to the original URL
      tags: [Redirect]
      parameters:
        - in: path
          name: slug
          required: true
          schema:
            type: string
      responses:
        "301":
          description: Redirect to long URL
        "404":
          description: Slug not found

  /health:
    get:
      summary: Health check
      tags: [System]
      responses:
        "200":
          description: OK

components:
  schemas:
    UserRequest:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
          format: email
        password:
          type: string
    UserResponse:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
        is_email_verified:
          type: boolean
        created_at:
          type: string
          format: date-time
    URLResponse:
      type: object
      properties:
        id:
          type: integer
        short_code:
          type: string
        long_url:
          type: string
        user_id:
          type: integer
        created_at:
          type: string
          format: date-time
        expires_at:
          type: string
          format: date-time
          nullable: true
        click_count:
          type: integer
    MessageResponse:
      type: object
      properties:
        message:
          type: string
    ErrorResponse:
      type: object
      properties:
        error:
          type: string
  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
    Unauthorized:
      description: Unauthorized
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
```
