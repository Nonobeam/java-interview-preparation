# 67. Global exception handling: `@ControllerAdvice` + `@ExceptionHandler`

## The problem

Without a central handler, each controller has to wrap calls in try/catch, return ad-hoc error shapes, and clients see inconsistent JSON or stack traces. `@ControllerAdvice` solves this by registering one class that intercepts exceptions thrown by **any** controller in the app.

## Minimal setup

```java
@RestControllerAdvice                  // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(EntityNotFoundException e) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ApiError("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiError> handleBadRequest(IllegalArgumentException e) {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ApiError("BAD_REQUEST", e.getMessage()));
    }

    @ExceptionHandler(Exception.class)        // catch-all — keep last
    public ResponseEntity<ApiError> handleAny(Exception e) {
        log.error("Unhandled", e);
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ApiError("INTERNAL_ERROR", "Unexpected error"));
    }
}

public record ApiError(String code, String message) {}
```

`@RestControllerAdvice` adds `@ResponseBody` so return values are serialized to JSON automatically — same relationship as `@RestController` to `@Controller`.

## Validation errors (`@Valid` failures)

Bean-validation failures throw `MethodArgumentNotValidException`. Translate field errors into a clean response:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ValidationError> handleValidation(MethodArgumentNotValidException e) {
    Map<String, String> fields = e.getBindingResult()
        .getFieldErrors()
        .stream()
        .collect(Collectors.toMap(
            FieldError::getField,
            fe -> fe.getDefaultMessage(),
            (a, b) -> a   // dedupe
        ));
    return ResponseEntity.badRequest().body(new ValidationError("VALIDATION_FAILED", fields));
}

public record ValidationError(String code, Map<String, String> fields) {}
```

## Domain exception → HTTP status mapping

A common pattern: throw plain Java exceptions in services; map them in one place.

```java
// Service layer — pure Java, no HTTP coupling
public class TransferService {
    public void transfer(Long from, Long to, BigDecimal amount) {
        Account src = accounts.findById(from)
            .orElseThrow(() -> new EntityNotFoundException("Account " + from));
        if (src.balance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(from, amount);
        }
        ...
    }
}

// Handler — single source of truth for HTTP semantics
@ExceptionHandler(InsufficientFundsException.class)
public ResponseEntity<ApiError> handleFunds(InsufficientFundsException e) {
    return ResponseEntity
        .status(HttpStatus.UNPROCESSABLE_ENTITY)   // 422
        .body(new ApiError("INSUFFICIENT_FUNDS", e.getMessage()));
}
```

Keeps the service layer free of HTTP concerns.

## Resolution order

When an exception is thrown, Spring looks for handlers in this order:

1. `@ExceptionHandler` on the same controller.
2. `@ExceptionHandler` in any matching `@ControllerAdvice`.
3. Built-in defaults (returns 500 with no body, or HTML error page).

Most-specific exception type wins — so `EntityNotFoundException` is preferred over a `RuntimeException` handler.

## Scoping advice

By default `@ControllerAdvice` applies to **all** controllers. Narrow it when needed:

```java
@RestControllerAdvice(basePackages = "com.acme.api.v1")
public class V1ApiAdvice { ... }

@RestControllerAdvice(annotations = RestController.class)   // skip MVC controllers
public class ApiOnlyAdvice { ... }
```

## ProblemDetail (RFC 7807) — modern alternative

Spring 6 / Boot 3 ships `ProblemDetail` for standards-compliant error responses:

```java
@ExceptionHandler(EntityNotFoundException.class)
public ProblemDetail handle(EntityNotFoundException e) {
    ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
    pd.setTitle("Resource not found");
    pd.setDetail(e.getMessage());
    pd.setType(URI.create("https://api.acme.com/errors/not-found"));
    return pd;
}
```

Or extend `ResponseEntityExceptionHandler` to override Spring's default mappings for things like `MethodArgumentNotValidException`.

## Interview one-liner

> "`@RestControllerAdvice` is a global interceptor: methods annotated `@ExceptionHandler(SomeException.class)` catch that exception type from any controller and return a structured response. It centralizes HTTP-status mapping, validation error formatting, and avoids try/catch in every endpoint. Spring 6 ships `ProblemDetail` for RFC 7807 errors."
