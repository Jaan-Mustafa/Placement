# JWT vs OAuth2 — What's the Difference?

## The One-Line Answer

```
JWT    = a token FORMAT  (what a token looks like and how it's verified)
OAuth2 = an authorization PROTOCOL (a set of flows for granting access)

They are not alternatives. OAuth2 often USES JWT as its token format.
Confusing them is like confusing JSON (data format) with REST (protocol).
```

---

## What is JWT?

JWT (JSON Web Token) is a **standard for packaging information** into a compact, self-contained, signed token.

### Structure — 3 parts separated by dots

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.4pcPyMD09olPSyXnrXCjTwXyr4BsezdI1AVTmud2fU4

     HEADER                  PAYLOAD                     SIGNATURE
eyJhbGciOiJIUzI1NiJ9  .  eyJzdWIiOiJ1c2VyMTIzIn0  .  4pcPyMD09olPSy...
      │                          │                            │
base64({"alg":"HS256"}) base64({"sub":"user123"})    HMAC(header+payload, secret)
```

**Decoded Header:**
```json
{
  "alg": "HS256",   // signing algorithm
  "typ": "JWT"
}
```

**Decoded Payload (Claims):**
```json
{
  "sub": "user123",           // subject — who this token is about
  "name": "Jaan Mustafa",
  "role": "ADMIN",
  "iat": 1718012400,          // issued at
  "exp": 1718016000           // expiry (1 hour later)
}
```

**Signature:**
```
HMAC_SHA256(
  base64(header) + "." + base64(payload),
  secret_key
)
```

### How JWT verification works

```
Server receives token
    │
    ▼
Split into header.payload.signature
    │
    ▼
Recompute: HMAC(header + "." + payload, server's secret key)
    │
    ▼
Does recomputed signature == token's signature?
    ├── YES → token is valid and untampered → trust the claims inside
    └── NO  → token was tampered → reject
```

**Key property — Stateless:**
```
Server does NOT store JWT anywhere.
Verification needs only the secret key — no DB lookup.
All user info is IN the token itself.

This is why JWTs scale well — any server instance can verify
without shared session storage.
```

---

## What is OAuth2?

OAuth2 is an **authorization framework** — a set of rules defining how one application can request access to resources in another application **on behalf of a user**, without the user sharing their password.

### The core problem OAuth2 solves

```
WITHOUT OAuth2 (old way):
  "I want to let my app read your Google Contacts"
  → User gives their Google password to your app
  → Your app logs into Google as the user
  → TERRIBLE — your app now has full Google access forever

WITH OAuth2:
  → User clicks "Login with Google" on your app
  → Google asks user: "Allow this app to read your contacts?"
  → User approves
  → Google gives your app a limited, time-bound ACCESS TOKEN
  → Your app uses that token — never sees user's Google password
  → User can revoke access anytime from Google settings
```

### OAuth2 Roles

```
Resource Owner    = the user (owns the data)
Client            = your application (wants access to data)
Authorization Server = issues tokens (Google, GitHub, your Auth service)
Resource Server   = holds the actual data (Google Contacts API, GitHub API)
```

### OAuth2 Flows

```
1. Authorization Code Flow (most common, most secure — web apps)
   ─────────────────────────────────────────────────────────────
   User clicks "Login with Google"
       │
       ▼
   Browser redirects to Google:
   https://accounts.google.com/o/oauth2/auth
     ?client_id=YOUR_APP
     &redirect_uri=https://yourapp.com/callback
     &scope=email profile
     &response_type=code
       │
       ▼
   User logs in + approves on Google
       │
       ▼
   Google redirects back:
   https://yourapp.com/callback?code=AUTH_CODE_XYZ
       │
       ▼
   Your backend exchanges code for tokens (server-to-server, secure):
   POST https://oauth2.googleapis.com/token
     code=AUTH_CODE_XYZ
     client_id=YOUR_APP
     client_secret=YOUR_SECRET
       │
       ▼
   Google returns:
   {
     "access_token": "ya29.a0A...",   ← use this to call Google APIs
     "refresh_token": "1//0g...",     ← use to get new access token
     "id_token": "eyJhbGci...",       ← JWT with user identity (OIDC)
     "expires_in": 3600
   }

2. Client Credentials Flow (service-to-service, no user involved)
   ─────────────────────────────────────────────────────────────
   ServiceA wants to call ServiceB internally
   POST /token
     client_id=serviceA
     client_secret=secret
     grant_type=client_credentials
       │
       ▼
   Auth server returns access token
   ServiceA calls ServiceB with Bearer token

3. Implicit Flow — DEPRECATED (tokens in URL, insecure)

4. Resource Owner Password Flow — DEPRECATED (user gives password to client)
```

---

## How JWT and OAuth2 Work Together

```
OAuth2 says:   "Here is an access token"
JWT says:      "Here is what that token looks like"

OAuth2 does NOT specify token format.
Tokens can be:
  → Opaque strings: "a1b2c3d4e5f6..."  (random, stored in DB)
  → JWTs:           "eyJhbGci..."       (self-contained, no DB needed)

Most modern implementations use JWT as the OAuth2 token format
because JWT is stateless — resource servers can verify without
calling the authorization server on every request.
```

```
Flow with JWT as OAuth2 token:

User authenticates
    │
    ▼
Authorization Server creates JWT:
{
  "sub": "user123",
  "scope": "read:invoices write:invoices",
  "client_id": "mobile-app",
  "exp": 1718016000
}
Signs it with Auth Server's private key → JWT access token
    │
    ▼
Client sends JWT in every API call:
GET /invoices
Authorization: Bearer eyJhbGci...
    │
    ▼
Resource Server (your API):
  → Verifies JWT signature using Auth Server's PUBLIC key
  → Checks expiry, scope, claims
  → No call to Auth Server needed — stateless verification
  → Allows or rejects request
```

---

## OpenID Connect (OIDC) — The Missing Piece

```
OAuth2 = authorization only ("what can this app do?")
OIDC   = authentication on top of OAuth2 ("who is this user?")

OAuth2 gives you: access_token (permission to act)
OIDC adds:        id_token     (identity of the user — always a JWT)

"Login with Google" = OIDC (which uses OAuth2 underneath)
```

```json
// id_token (JWT) from OIDC
{
  "iss": "https://accounts.google.com",   // issuer
  "sub": "1234567890",                    // Google's user ID
  "email": "user@gmail.com",
  "name": "Jaan Mustafa",
  "iat": 1718012400,
  "exp": 1718016000
}
```

---

## Side-by-Side Comparison

| | JWT | OAuth2 |
|---|---|---|
| **What it is** | Token format/standard | Authorization protocol/framework |
| **Defined by** | RFC 7519 | RFC 6749 |
| **Purpose** | Encode + verify claims | Delegate access between apps |
| **Stateful?** | No — self-contained | Depends on token type |
| **Token format** | IS the token format | Uses tokens (often JWT) |
| **Verification** | Check signature locally | Depends on token type |
| **Flows** | N/A (just a format) | Auth Code, Client Credentials, etc. |
| **Involves redirect?** | No | Yes (Authorization Code flow) |
| **Third-party auth** | Not specifically | Yes — core use case |

---

## Common Real-World Usage

```
Scenario 1 — Internal JWT auth (no OAuth2)
  User logs in with username/password to YOUR app
  YOUR auth service verifies credentials
  YOUR auth service issues a JWT
  Client sends JWT on every request
  YOUR API verifies JWT signature
  → Just JWT. OAuth2 not involved.

Scenario 2 — "Login with Google" (OAuth2 + OIDC + JWT)
  OAuth2 flow to get access from Google
  OIDC id_token (JWT) tells you who the user is
  Google's access_token (often a JWT) lets you call Google APIs
  → OAuth2 protocol + JWT token format

Scenario 3 — Microservice-to-Microservice auth (OAuth2 Client Credentials + JWT)
  ServiceA gets JWT access token from Auth Server
  ServiceA calls ServiceB with that JWT
  ServiceB verifies JWT signature locally
  → OAuth2 Client Credentials flow + JWT token format

Scenario 4 — Spring Security setup (most common interview context)
  User logs in → server creates JWT → client stores in localStorage
  Client sends JWT as Bearer token
  Spring Security filter validates JWT on every request
  → Just JWT, no OAuth2 unless you're integrating external providers
```

---

## Spring Boot Implementation

### Pure JWT (no OAuth2)

```java
// Generate JWT
public String generateToken(UserDetails user) {
    return Jwts.builder()
        .setSubject(user.getUsername())
        .claim("role", user.getAuthorities())
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 3600_000))
        .signWith(secretKey, SignatureAlgorithm.HS256)
        .compact();
}

// Validate JWT
public boolean validateToken(String token) {
    try {
        Jwts.parserBuilder()
            .setSigningKey(secretKey)
            .build()
            .parseClaimsJws(token);   // throws if invalid/expired
        return true;
    } catch (JwtException e) {
        return false;
    }
}
```

### OAuth2 with Spring Security

```java
// application.yml — OAuth2 Login with Google
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: YOUR_CLIENT_ID
            client-secret: YOUR_CLIENT_SECRET
            scope: openid, email, profile
```

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService)
                )
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt()  // validate incoming JWT access tokens
            )
            .build();
    }
}
```

---

## Key Interview Points

1. **JWT is a format, OAuth2 is a protocol** — they are not alternatives. OAuth2 uses JWT as its token format.

2. **JWT is stateless** — the server verifies the signature locally with no DB call. This is why it scales.

3. **OAuth2 solves delegated authorization** — "allow app X to access my data in app Y without sharing my password."

4. **OIDC = OAuth2 + identity** — OAuth2 alone doesn't tell you WHO the user is, only what they're allowed to do. OIDC adds the `id_token` (always a JWT) for identity.

5. **Access token vs ID token** — access token = permission to call an API. ID token (OIDC) = proof of who the user is.

6. **Opaque token vs JWT token in OAuth2** — opaque tokens require calling the auth server to validate (introspection endpoint). JWT tokens can be validated locally.

7. **Refresh token** — long-lived token used to get a new access token when the old one expires, without making the user log in again.

---

## One-Line Summary for Interviews

> "JWT is a token format — a signed, self-contained JSON payload that any server can verify locally without a DB call. OAuth2 is an authorization protocol that defines flows for delegating access between applications. They work together: OAuth2 handles the flow of obtaining permission, and JWT is typically the format of the token it issues."
