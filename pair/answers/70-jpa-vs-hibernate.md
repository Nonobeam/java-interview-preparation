# 70. JPA vs Hibernate — specification vs implementation

## Core distinction

| | JPA | Hibernate |
|---|---|---|
| What it is | **Specification** (a set of interfaces/annotations) | **Implementation** of that specification |
| Package | `jakarta.persistence.*` | `org.hibernate.*` |
| Who defines it | Jakarta EE community (formerly Sun/Oracle) | Red Hat / Hibernate team |
| Can you run it alone? | No — needs an implementation | Yes — it implements JPA and can run standalone |

JPA is a contract. Hibernate is a library that fulfils that contract.

## Analogy

JPA is like the JDBC API: it defines `DataSource`, `Connection`, `PreparedStatement`. Hibernate is like a JDBC driver — it actually does the work. You code to the JPA interface; Hibernate executes the SQL.

## What JPA gives you

- Annotations: `@Entity`, `@Table`, `@Id`, `@OneToMany`, `@ManyToOne`, `@Version`, etc.
- `EntityManager` / `EntityManagerFactory` API
- JPQL (Java Persistence Query Language)
- Transaction integration via `@Transactional`

All of these are standard JPA — you can swap Hibernate for EclipseLink or OpenJPA and the code still compiles.

## What Hibernate adds on top

Hibernate-specific features not in JPA:
- `@NaturalId` — secondary unique lookup
- `@BatchSize` — controls batch loading for collections
- `@Fetch(FetchMode.SUBSELECT)` — alternative fetch strategy
- `@CreationTimestamp` / `@UpdateTimestamp` — automatic audit timestamps
- `SessionFactory` / `Session` — Hibernate's own API (superset of `EntityManagerFactory` / `EntityManager`)
- Native `Criteria` API (older Hibernate before JPA's `CriteriaBuilder`)
- Envers — auditing / history
- Second-level cache integration (Ehcache, Caffeine, etc.)

You reach for Hibernate-specific annotations when JPA's standard options aren't powerful enough.

## Can you swap Hibernate for EclipseLink?

Yes, but in practice nobody does. In a Spring Boot project:
1. Replace `spring-boot-starter-data-jpa` (which pulls Hibernate) with EclipseLink dependency
2. Set `spring.jpa.properties.eclipselink.*` instead of `spring.jpa.properties.hibernate.*`
3. Remove any `@org.hibernate.*` annotations (replace with JPA equivalents or remove)

The JPA-standard code compiles unchanged; Hibernate-specific annotations must be removed or replaced.

## Spring Data JPA — one more layer

```
Your code
   ↓
Spring Data JPA (repositories, query derivation, pagination)
   ↓
JPA (EntityManager, JPQL, annotations)
   ↓
Hibernate (actual SQL generation, caching, session management)
   ↓
JDBC / DataSource
   ↓
Database
```

When someone says "we use JPA" they usually mean the whole stack. When they say "we use Hibernate" they mean the same thing, just emphasising the implementation. **Both are correct for the same project.**

## Which annotations to prefer

- Default to **JPA annotations** (`jakarta.persistence.*`) for portability.
- Use **Hibernate annotations** only when JPA doesn't have an equivalent, e.g., `@NaturalId`, `@BatchSize`, `@CreationTimestamp`.
- Keep Hibernate-specific annotations isolated (in entities or a config layer) so they are easy to find if you ever need to migrate.

## Interview one-liner

> "JPA is a specification — a set of standard interfaces and annotations in `jakarta.persistence`. Hibernate is an implementation of that specification that adds extra features on top. Spring Data JPA sits above both, giving you repository abstractions and query derivation. When you annotate an entity with `@Entity`, that's JPA; when Hibernate generates the SQL and manages the session cache, that's Hibernate. You can swap Hibernate for EclipseLink without changing JPA-standard code, but nobody does in practice."
