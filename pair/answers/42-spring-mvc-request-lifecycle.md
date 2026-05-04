# A46: Spring MVC request lifecycle

## The lifecycle

```
HTTP Request
    ↓
1. DispatcherServlet        (Front Controller — single entry point, configured in web.xml or SpringBoot auto-config)
    ↓
2. HandlerMapping           (finds which controller method handles this URL + HTTP method)
    → returns HandlerExecutionChain: handler + list of HandlerInterceptors
    ↓
3. HandlerInterceptor.preHandle()   (runs for each interceptor — auth, logging, etc.)
    ↓
4. HandlerAdapter           (bridges DispatcherServlet → actual controller; default: RequestMappingHandlerAdapter)
    → resolves method arguments (@RequestParam, @PathVariable, @RequestBody, etc.)
    → calls the controller method
    ↓
5. Controller method executes, returns:
    - ModelAndView  (for view rendering)
    - String        (view name)
    - @ResponseBody / ResponseEntity  (skip view resolution entirely)
    ↓
6. HandlerInterceptor.postHandle()  (runs after controller, before view render)
    ↓
7. ViewResolver             (maps view name string → actual View implementation, e.g. InternalResourceViewResolver → JSP, ThymeleafViewResolver → .html)
    ↓
8. View.render()            (merges Model data into template, writes HTML to response)
    ↓
9. HandlerInterceptor.afterCompletion()  (always runs, even on exception — good for cleanup)
    ↓
HTTP Response
```

---

## When @ResponseBody is used

Steps 7 and 8 (ViewResolver + View render) are **skipped entirely**.

Instead, a `HttpMessageConverter` serializes the return value directly into the response body (e.g., `MappingJackson2HttpMessageConverter` → JSON).

```java
@GetMapping("/users/{id}")
@ResponseBody          // or use @RestController at class level
public UserDto getUser(@PathVariable Long id) {
    return userService.findById(id);
    // → Jackson serializes UserDto → JSON → written to response body
}
```

`@RestController` = `@Controller` + `@ResponseBody` on every method.

---

## Key components summary

| Component | Role |
|---|---|
| `DispatcherServlet` | Front controller, orchestrates everything |
| `HandlerMapping` | URL → handler lookup (e.g. `RequestMappingHandlerMapping`) |
| `HandlerInterceptor` | Cross-cutting concerns: auth, logging, rate limiting |
| `HandlerAdapter` | Invokes the controller method, resolves arguments |
| `Controller` | Your business logic |
| `ViewResolver` | View name → View implementation |
| `HttpMessageConverter` | Serialization for `@ResponseBody` |

---

## Comparison: Struts 2 vs Spring MVC lifecycle

| Step | Struts 2 | Spring MVC |
|---|---|---|
| Entry point | `StrutsPrepareAndExecuteFilter` | `DispatcherServlet` |
| Routing | `ActionMapper` + struts.xml | `HandlerMapping` + `@RequestMapping` |
| Pre/post hooks | Interceptor stack | `HandlerInterceptor` |
| Business logic | `Action.execute()` | `@Controller` method |
| View resolution | Result in struts.xml | `ViewResolver` |

---

## Real code — full minimal Spring MVC flow

```java
// 1. Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}

// 2. HandlerInterceptor example (auth check)
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        String token = request.getHeader("Authorization");
        if (token == null || !tokenService.isValid(token)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;  // short-circuit — controller is NOT called
        }
        return true;
    }
}

// 3. Register interceptor
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private AuthInterceptor authInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/api/**");
    }
}
```

---

## Edge cases worth knowing

- **Exception handling**: if the controller throws, `DispatcherServlet` delegates to `HandlerExceptionResolver` (e.g. `@ControllerAdvice` + `@ExceptionHandler`)
- **Filter vs Interceptor**: Servlet `Filter` runs *outside* `DispatcherServlet` (can't access Spring context); `HandlerInterceptor` runs *inside* and has access to the handler method
- **Async**: if controller returns `Callable` or `DeferredResult`, DispatcherServlet releases the thread and resumes when the async computation completes
