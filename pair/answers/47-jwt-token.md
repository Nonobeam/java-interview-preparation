# 47. JWT Token

JSON Web Token — a compact, self-contained token for stateless authentication. The server doesn't store session state; everything the server needs is encoded in the token itself.

---

## Structure

```
header.payload.signature
```

Each part is Base64Url-encoded:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← header
.eyJzdWIiOiJ1c2VyMTIzIiwicm9sZSI6IkFETUlOIiwiZXhwIjoxNzE3MDAwMDAwfQ==  ← payload
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← signature
```

**Header** — algorithm + token type:
```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** — claims (registered + custom):
```json
{
  "sub": "user123",
  "role": "ADMIN",
  "iat": 1716000000,
  "exp": 1716003600
}
```

**Signature** — prevents tampering:
```
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

---

## Authentication flow

```
1. Client  → POST /login { username, password }
2. Server  → validates credentials
           → creates JWT (signs with secret/private key)
           → returns { accessToken, refreshToken }
3. Client  → stores tokens (memory or httpOnly cookie)
4. Client  → GET /api/orders  Authorization: Bearer <accessToken>
5. Server  → verifies signature + expiry
           → extracts claims (no DB lookup needed)
           → serves response
```

---

## Access token vs Refresh token

| | Access token | Refresh token |
|---|---|---|
| Lifetime | Short (15 min – 1 hr) | Long (7–30 days) |
| Used for | Every API request | Getting new access token |
| Storage | Memory / JS | httpOnly cookie / secure storage |
| Revocation | Hard (stateless) | Easy (stored in DB) |

---

## Spring Boot integration

```java
// build.gradle
implementation 'io.jsonwebtoken:jjwt-api:0.12.3'
runtimeOnly    'io.jsonwebtoken:jjwt-impl:0.12.3'
runtimeOnly    'io.jsonwebtoken:jjwt-jackson:0.12.3'

// Generate token
String token = Jwts.builder()
    .subject(username)
    .claim("role", role)
    .issuedAt(new Date())
    .expiration(new Date(System.currentTimeMillis() + 3_600_000))
    .signWith(secretKey)
    .compact();

// Validate token
Claims claims = Jwts.parser()
    .verifyWith(secretKey)
    .build()
    .parseSignedClaims(token)
    .getPayload();
String username = claims.getSubject();
```

Filter chain:
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ... {
        String header = req.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String username = jwtService.extractUsername(token);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails user = userDetailsService.loadUserByUsername(username);
                if (jwtService.isValid(token, user)) {
                    var auth = new UsernamePasswordAuthenticationToken(
                            user, null, user.getAuthorities());
                    SecurityContextHolder.getContext().setAuthentication(auth);
                }
            }
        }
        chain.doFilter(req, res);
    }
}
```

---

## Common pitfalls

- **Storing in localStorage** — vulnerable to XSS; prefer httpOnly cookie for refresh token.
- **No expiry** — always set `exp`. A stolen JWT without expiry is valid forever.
- **Sensitive data in payload** — payload is only Base64-encoded, not encrypted. Anyone can decode it.
- **Can't revoke access tokens** — by design. Use short expiry + refresh token rotation as mitigation.

---

## Interview one-liner
> "JWT is a signed, Base64-encoded token containing claims. The server verifies the signature on every request without hitting the database, making it stateless. Short-lived access tokens with longer-lived refresh tokens balance security and UX."
