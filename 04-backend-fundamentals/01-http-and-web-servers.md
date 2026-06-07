`[Entry]`

# HTTP and Web Servers

When you type `https://example.com/users` into a browser, a chain of events happens. Understanding this chain is fundamental to building backend services.

## What Happens When You Type a URL

1. **DNS resolution** — Your browser asks a DNS server: "What IP address is example.com?" DNS returns something like `93.184.216.34`.
2. **TCP connection** — Your browser connects to that IP address on port 443 (HTTPS). This is a reliable two-way connection.
3. **TLS handshake** — Your browser and the server establish an encrypted connection. This is why HTTPS is secure.
4. **HTTP request** — Your browser sends a request message:
   ```
   GET /users HTTP/1.1
   Host: example.com
   Accept: application/json
   ```
5. **Server processes** — The server receives the request, runs your code, queries the database, builds a response.
6. **HTTP response** — The server sends back:
   ```
   HTTP/1.1 200 OK
   Content-Type: application/json

   [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
   ```
7. **Browser renders** — The browser receives the JSON and displays it.

Your job as a backend developer: handle step 5. Receive a request, do work, return a response.

## HTTP Request Anatomy

```
POST /users HTTP/1.1          ← method, path, version
Host: api.example.com         ← headers (metadata)
Content-Type: application/json
Authorization: Bearer eyJhbG...

{"name": "Alice", "email": "alice@example.com"}  ← body (optional)
```

| Part | Purpose | Example |
|------|---------|---------|
| Method | What action to perform | GET, POST, PUT, DELETE |
| Path | Which resource | /users, /users/1 |
| Headers | Metadata about the request | Content-Type, Authorization |
| Body | Data sent to the server | JSON, form data, file |

## HTTP Response Anatomy

```
HTTP/1.1 201 Created          ← status code and reason
Content-Type: application/json
Location: /users/3

{"id": 3, "name": "Alice", "email": "alice@example.com"}
```

| Part | Purpose | Example |
|------|---------|---------|
| Status code | Result of the request | 200 OK, 404 Not Found, 500 Internal Server Error |
| Headers | Metadata about the response | Content-Type, Set-Cookie |
| Body | Data returned to the client | JSON, HTML, file |

## HTTP Methods

| Method | Purpose | Has body? | Idempotent? |
|--------|---------|-----------|-------------|
| GET | Retrieve a resource | No | Yes |
| POST | Create a new resource | Yes | No |
| PUT | Replace a resource entirely | Yes | Yes |
| PATCH | Partially update a resource | Yes | No |
| DELETE | Remove a resource | Optional | Yes |

Idempotent means: making the same request multiple times produces the same result as making it once. GET, PUT, and DELETE are idempotent. POST is not (each POST may create a new resource).

## Status Codes

| Range | Category | Common codes |
|-------|----------|-------------|
| 200–299 | Success | 200 OK, 201 Created, 204 No Content |
| 300–399 | Redirection | 301 Moved Permanently, 304 Not Modified |
| 400–499 | Client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 500–599 | Server error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

## What a Web Server Does

A web server is a program that:
1. Listens on a port (e.g., 3000)
2. Accepts incoming TCP connections
3. Reads HTTP requests
4. Routes each request to a handler function based on method and path
5. The handler does work (database query, business logic)
6. Sends an HTTP response back

In Rust with Axum, you define routes and handlers. Axum handles steps 1–3 and 6. You write step 4 and 5.

```
[Client] --HTTP request--> [Axum router] --route match--> [Your handler function]
                                                                    |
                                                              [Database query]
                                                                    |
                          [Axum sends response] <--[Your handler returns response]
```

## Content Types

The `Content-Type` header tells the receiver what format the body is in:

| Header value | Format |
|-------------|--------|
| `application/json` | JSON (most common for APIs) |
| `text/html` | HTML page |
| `text/plain` | Plain text |
| `application/x-www-form-urlencoded` | Form data |

APIs typically send and receive JSON. Rust handles this with `serde_json`.

## Next

The next file covers REST API design: how to structure your URLs, use HTTP verbs correctly, and design a clean API.
