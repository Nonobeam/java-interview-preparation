# 71. OAuth2

## What OAuth2 is — say this first

OAuth2 is an **authorization framework** — a set of rules (a protocol) that defines how a client application can obtain limited access to a resource on behalf of a user, **without the user giving the client their password**.

It is **not a library**, not an implementation, not an authentication protocol.

## What it is NOT

- **Not authentication** — OAuth2 only grants *access*. It doesn't tell you *who the user is*. That's OpenID Connect (OIDC), which is a thin identity layer built on top of OAuth2.
- **Not a specific token format** — OAuth2 says "issue an access token" but doesn't define what the token looks like. You choose: opaque token (random string the server must look up) or JWT (self-contained, signed).

## The four roles

| Role | Who it is in practice |
|------|-----------------------|
| **Resource Owner** | The end user (owns their data) |
| **Client** | Your app that wants access (mobile app, SPA, backend service) |
| **Authorization Server** | The entity that authenticates the user and issues tokens (Google, Okta, Keycloak, your own Auth service) |
| **Resource Server** | The API that holds the protected data (your backend) |

## The four grant types (flows)

### 1. Authorization Code (+ PKCE)
Most common for user-facing apps (SPA, mobile, web). User is redirected to the authorization server, logs in, and gets redirected back with an authorization code. The client exchanges the code for an access token server-side.

PKCE (Proof Key for Code Exchange) is an extension that prevents code interception — mandatory for public clients (SPAs, mobile apps).

```
User → Client → Authorization Server (login page)
                      ↓ redirect with code
Client exchanges code → access token + refresh token
```

### 2. Client Credentials
Server-to-server (no user involved). The client authenticates with its own `client_id` + `client_secret` and gets a token directly.

```
Service A → Authorization Server (client_id + secret)
                   ↓
            access token
```

### 3. Device Code
For input-constrained devices (TV, CLI tools). User enters a code on a different device (phone/browser).

### 4. ~~Implicit~~ (deprecated)
Was used for SPAs before PKCE existed. Now replaced by Authorization Code + PKCE.

## How it works in a typical API call

```
1. Client obtains access token (via any grant type above)
2. Client calls: GET /api/orders
                 Authorization: Bearer <access_token>
3. Resource Server validates the token (introspection or JWT signature check)
4. Resource Server returns data if token is valid and has the right scope
```

## OAuth2 vs OpenID Connect (OIDC)

| | OAuth2 | OIDC |
|---|---|---|
| Purpose | Authorization (can this app do X?) | Authentication (who is this user?) |
| What you get | `access_token` | `access_token` + `id_token` (JWT with user info) |
| Standard endpoint | `/authorize`, `/token` | Adds `/userinfo`, discovery doc |

"Login with Google" uses **OIDC** (built on top of OAuth2). Google is both the Authorization Server and the Identity Provider.

## Spring Security implementation

For a **resource server** (API that validates tokens):

```java
// application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: https://auth-server/.well-known/jwks.json
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        return http.build();
    }
}
```

Spring validates the JWT signature against the public keys from `jwk-set-uri` automatically.

For scopes and roles:

```java
@PreAuthorize("hasAuthority('SCOPE_orders:read')")
public List<Order> getOrders() { ... }
```

## Interview one-liner

> "OAuth2 is an authorization framework — a protocol, not a library. It defines how a client can request limited access to a resource on behalf of a user without that user sharing credentials. The four roles are: resource owner, client, authorization server, and resource server. The most common flow is Authorization Code + PKCE for user-facing apps, and Client Credentials for service-to-service. OAuth2 only handles authorization; if you need 'who is this user', you layer OpenID Connect on top of it."
