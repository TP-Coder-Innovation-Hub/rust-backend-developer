`[Mid]`

# Authentication

Authentication verifies who a user is. Authorization determines what they can do. This file covers the concepts and how to implement them in a Rust backend.

## How Authentication Works

The typical flow:

1. User sends credentials (email + password) to a login endpoint
2. Server verifies credentials against the database
3. Server issues a token (JWT) or creates a session
4. Client includes the token in subsequent requests
5. Server validates the token on each request

## JWT — JSON Web Tokens

A JWT is a self-contained token with three parts: header, payload, signature.

```
header.payload.signature
```

- **Header** — algorithm and token type (JSON, base64-encoded)
- **Payload** — claims: user ID, role, expiration time (JSON, base64-encoded)
- **Signature** — HMAC of header + payload using a secret key

The signature prevents tampering. If anyone changes the payload, the signature no longer matches.

### Setup

Add to `Cargo.toml`:

```toml
[dependencies]
jsonwebtoken = "9"
bcrypt = "0.16"
chrono = { version = "0.4", features = ["serde"] }
```

- `jsonwebtoken` — create and verify JWTs
- `bcrypt` — hash and verify passwords (never store plain text passwords)
- `chrono` — date and time handling

### Password Hashing

Never store plain text passwords. Use bcrypt:

```rust
use bcrypt::{hash, verify, DEFAULT_COST};

fn hash_password(password: &str) -> Result<String, bcrypt::BcryptError> {
    hash(password, DEFAULT_COST)
}

fn check_password(password: &str, hash: &str) -> Result<bool, bcrypt::BcryptError> {
    verify(password, hash)
}
```

Step by step:
- `hash(password, DEFAULT_COST)` — hash the password with a salt. `DEFAULT_COST` is the computational cost factor (higher = slower = more secure).
- `verify(password, hash)` — check if a plain text password matches the stored hash
- The hash includes the salt. You store only the hash string.

### JWT Claims

Define what goes into the token:

```rust
use serde::{Deserialize, Serialize};
use chrono::Utc;

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,            // subject: user ID
    role: String,           // user role
    exp: usize,             // expiration time (UNIX timestamp)
    iat: usize,             // issued at
}
```

### Creating a Token

```rust
use jsonwebtoken::{encode, EncodingKey, Header};

fn create_token(user_id: &str, role: &str, secret: &str) -> Result<String, jsonwebtoken::errors::Error> {
    let now = Utc::now();
    let claims = Claims {
        sub: user_id.to_owned(),
        role: role.to_owned(),
        iat: now.timestamp() as usize,
        exp: (now + chrono::Duration::hours(24)).timestamp() as usize,
    };

    encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret(secret.as_bytes()),
    )
}
```

Step by step:
- `sub` — the user ID this token belongs to
- `exp` — when the token expires (24 hours from now)
- `iat` — when the token was created
- `encode(...)` — create the signed JWT string
- `EncodingKey::from_secret(...)` — the secret key used to sign the token

### Verifying a Token

```rust
use jsonwebtoken::{decode, DecodingKey, Validation};

fn verify_token(token: &str, secret: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
    let token_data = decode::<Claims>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &Validation::default(),
    )?;

    Ok(token_data.claims)
}
```

Step by step:
- `decode::<Claims>(...)` — verify the signature and decode the payload into `Claims`
- `Validation::default()` — checks expiration, valid algorithm, etc.
- If the token is expired, tampered with, or has an invalid signature, this returns an error

### Login Handler

```rust
use axum::{Json, http::StatusCode};

#[derive(serde::Deserialize)]
struct LoginRequest {
    email: String,
    password: String,
}

#[derive(serde::Serialize)]
struct LoginResponse {
    token: String,
}

async fn login(
    pool: &sqlx::PgPool,
    input: Json<LoginRequest>,
    secret: &str,
) -> Result<(StatusCode, Json<LoginResponse>), (StatusCode, String)> {
    let user = sqlx::query_as::<_, User>(
        "SELECT id, name, email, password_hash FROM users WHERE email = $1"
    )
    .bind(&input.email)
    .fetch_optional(pool)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    let user = user.ok_or((StatusCode::UNAUTHORIZED, "Invalid credentials".into()))?;

    let valid = check_password(&input.password, &user.password_hash)
        .map_err(|_| (StatusCode::INTERNAL_SERVER_ERROR, "Auth error".into()))?;

    if !valid {
        return Err((StatusCode::UNAUTHORIZED, "Invalid credentials".into()));
    }

    let token = create_token(&user.id.to_string(), "user", secret)
        .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    Ok((StatusCode::OK, Json(LoginResponse { token })))
}
```

Step by step:
- Look up the user by email
- If not found, return 401
- Verify the password against the stored hash
- If wrong, return 401
- Create a JWT and return it

## Protecting Routes with Middleware

Use Axum middleware to require authentication on specific routes:

```rust
use axum::{
    extract::{Request, State},
    middleware::{self, Next},
    response::Response,
};

async fn auth_middleware(
    State(secret): State<String>,
    mut request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let auth_header = request
        .headers()
        .get("Authorization")
        .and_then(|v| v.to_str().ok())
        .ok_or(StatusCode::UNAUTHORIZED)?;

    let token = auth_header
        .strip_prefix("Bearer ")
        .ok_or(StatusCode::UNAUTHORIZED)?;

    let claims = verify_token(token, &secret)
        .map_err(|_| StatusCode::UNAUTHORIZED)?;

    request.extensions_mut().insert(claims);
    Ok(next.run(request).await)
}
```

Step by step:
- Extract the `Authorization` header from the request
- Strip the "Bearer " prefix to get the raw token
- Verify the token
- If valid, insert the `Claims` into request extensions so handlers can access them
- If invalid, return 401 Unauthorized
- `next.run(request)` — pass the request to the next handler in the chain

Apply to routes:

```rust
let protected = Router::new()
    .route("/profile", get(get_profile))
    .layer(middleware::from_fn_with_state(secret.clone(), auth_middleware));
```

## Sessions vs JWT

| Property | JWT | Sessions |
|----------|-----|----------|
| State stored on | Client (token) | Server (database/Redis) |
| Revocation | Difficult (token valid until expiry) | Easy (delete session) |
| Scalability | Stateless (no server storage) | Requires shared session store |
| Size | Larger (contains claims) | Small (just a session ID) |
| Use case | APIs, microservices | Web apps with server-rendered pages |

## Security Checklist

| Practice | Why |
|----------|-----|
| Hash passwords with bcrypt, never store plain text | Data breach does not expose passwords |
| Use HTTPS | Prevents token interception |
| Set short token expiry (1–24 hours) | Limits damage from stolen tokens |
| Validate token on every request | Prevents unauthorized access |
| Use a strong secret key (32+ random bytes) | Prevents token forgery |
| Never put sensitive data in JWT payload | The payload is base64, not encrypted |

## Next

The next module covers production concerns: testing, logging, and deployment.
