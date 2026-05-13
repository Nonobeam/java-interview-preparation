# Q71 — What is OAuth2?

Key constraint: OAuth2 is an **authorization framework / protocol** (a set of rules), not a library or tool. The answer must make this clear.

## Questions

1. Define OAuth2. Why is it called an authorization framework, not an authentication protocol?
2. Name the four OAuth2 grant types (flows). Which one is most common for server-to-server? Which for a browser SPA?
3. Who are the four roles in OAuth2? (resource owner, client, authorization server, resource server)
4. What does OAuth2 actually hand out? (access token) What does it NOT define? (what the token looks like — that's JWT or opaque token, your choice)
5. How does OAuth2 differ from OpenID Connect (OIDC)? If you want "login with Google", which one are you actually using?
6. How does Spring Security implement OAuth2? What annotations/configs are involved in a resource server setup?
