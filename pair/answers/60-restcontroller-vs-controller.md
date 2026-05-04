# 65. `@RestController` vs `@Controller`

## Short answer

`@RestController` = `@Controller` + `@ResponseBody` applied to every method.

- `@Controller` returns a **view name** that a `ViewResolver` renders (Thymeleaf, JSP, ...). Use for server-rendered HTML.
- `@RestController` returns the **method's return value as the HTTP body** (serialized to JSON via Jackson by default). Use for REST APIs.

## Side by side

```java
// Server-rendered HTML page
@Controller
public class UserPageController {
    @GetMapping("/users/{id}")
    public String show(@PathVariable Long id, Model model) {
        model.addAttribute("user", service.find(id));
        return "user";   // → resolves to /templates/user.html
    }
}
```

```java
// JSON REST API
@RestController
@RequestMapping("/api/users")
public class UserApiController {
    @GetMapping("/{id}")
    public User get(@PathVariable Long id) {
        return service.find(id);   // serialized to JSON automatically
    }

    @PostMapping
    public ResponseEntity<User> create(@RequestBody @Valid CreateUser cmd) {
        User u = service.create(cmd);
        return ResponseEntity
            .created(URI.create("/api/users/" + u.id()))
            .body(u);
    }
}
```

## What `@ResponseBody` does

Without it, the return value is interpreted as a **view name**. With it, an `HttpMessageConverter` (default: `MappingJackson2HttpMessageConverter` for JSON) writes the object directly to the response body.

```java
// Equivalent of @RestController, the long way
@Controller
public class UserApiController {

    @GetMapping("/api/users/{id}")
    @ResponseBody                      // <-- without this, "user" would be a view name
    public User get(@PathVariable Long id) { ... }
}
```

## Mixing the two on one class — usually wrong

`@RestController` cannot return a view; if you need both HTML and JSON endpoints in the same place, either:
- Split into two controllers (clean), or
- Use `@Controller` + add `@ResponseBody` per JSON method (verbose).

## When you still need `@Controller` for an API

Returning `ModelAndView`, redirect view, or rendering server-side templates. For a pure JSON microservice, always `@RestController`.

## `ResponseEntity` — when you need control over status / headers

Both `@Controller` (with `@ResponseBody`) and `@RestController` can return `ResponseEntity<T>` to set status code, headers, and body explicitly:

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    service.delete(id);
    return ResponseEntity.noContent().build();   // 204
}
```

Plain return (e.g. `return user;`) defaults to `200 OK`.

## Interview one-liner

> "`@RestController` is `@Controller` + `@ResponseBody` on every method — return values get serialized to JSON via Jackson and sent as the HTTP body. `@Controller` returns a view name that a `ViewResolver` renders into HTML. Use `@RestController` for REST APIs, `@Controller` for server-rendered pages."
