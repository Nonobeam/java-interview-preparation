# Struts vs Spring MVC — Interview Reference

## What is Apache Struts?

Apache Struts is a Java MVC web framework. Two major versions exist:
- **Struts 1** (legacy, EOL) — very old, tightly coupled, hard to test
- **Struts 2** (modern, still used in enterprise) — complete rewrite based on WebWork, far more flexible

When a company says "Struts" today, they almost always mean **Struts 2**.

---

## Struts 2 Core Concepts

### Action Classes
The heart of Struts 2. Each Action class handles a specific request.

```java
public class LoginAction extends ActionSupport {
    private String username;
    private String password;

    @Override
    public String execute() throws Exception {
        if (authService.login(username, password)) {
            return SUCCESS;  // "success" result
        }
        return ERROR;        // "error" result
    }

    // getters + setters required (OGNL data binding)
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
}
```

Key difference from Spring MVC `@Controller`: **Action classes are prototype-scoped (new instance per request)**, so they're thread-safe by default. Spring MVC controllers are singletons.

### struts.xml — Configuration
```xml
<struts>
    <package name="default" namespace="/" extends="struts-default">
        <action name="login" class="com.example.LoginAction">
            <result name="success">/WEB-INF/views/home.jsp</result>
            <result name="error">/WEB-INF/views/login.jsp</result>
        </action>
    </package>
</struts>
```
Equivalent to Spring MVC's `@RequestMapping`.

### Interceptors
Pre/post processing around Action execution. Similar to Spring MVC `HandlerInterceptor` or Servlet `Filter`.

Built-in interceptors: `params` (binds request params), `validation`, `fileUpload`, `token`, `logger`.

```xml
<interceptors>
    <interceptor name="myAuth" class="com.example.AuthInterceptor"/>
</interceptors>
<action name="dashboard" class="DashboardAction">
    <interceptor-ref name="myAuth"/>
    <interceptor-ref name="defaultStack"/>
    <result>/dashboard.jsp</result>
</action>
```

### ValueStack & OGNL
The ValueStack is a stack of objects accessible from JSP/view layer using OGNL expressions.

```jsp
<!-- OGNL in JSP (Struts 2 tags) -->
<s:property value="username"/>
<s:if test="loggedIn">Welcome!</s:if>
```

Spring MVC equivalent: `${model.username}` in Thymeleaf / `@ModelAttribute`.

---

## Side-by-Side Comparison

| Concept | Struts 2 | Spring MVC |
|---|---|---|
| Entry point | `FilterDispatcher` / `StrutsPrepareAndExecuteFilter` | `DispatcherServlet` |
| Controller | `Action` class (prototype scope) | `@Controller` (singleton) |
| Routing config | `struts.xml` or annotations | `@RequestMapping` |
| Data binding | OGNL + setters on Action | `@RequestParam`, `@ModelAttribute` |
| Pre/post processing | Interceptors | `HandlerInterceptor`, `@Aspect` |
| View resolution | Result types in struts.xml | `ViewResolver` |
| Validation | `validate()` method or XML | `@Valid`, `BindingResult` |
| DI container | Integrates with Spring (optional) | Native Spring IoC |

---

## Request Lifecycle — Struts 2

```
Client Request
    ↓
StrutsPrepareAndExecuteFilter  (servlet filter, entry point)
    ↓
ActionMapper  (maps URL → Action name)
    ↓
ActionProxy  (wraps Action + Interceptor stack)
    ↓
Interceptor Stack (pre-processing: params binding, validation…)
    ↓
Action.execute()  (your business logic)
    ↓
Interceptor Stack (post-processing, reverse order)
    ↓
Result (JSP, redirect, JSON…)
    ↓
Response to Client
```

---

## How to Answer "Have You Used Struts?"

Honest, confident answer:

> "I haven't worked with Struts 2 directly in production — my background is Spring Boot / Spring MVC. But I understand the concepts: Action classes are roughly equivalent to Spring controllers, interceptors map to HandlerInterceptors or filters, and struts.xml serves the same role as @RequestMapping. The MVC lifecycle is structurally similar. I'd expect a learning curve on OGNL and the ValueStack, but the core web MVC pattern is familiar."

---

## Why Companies Still Use Struts

- Large enterprise apps built pre-2010 were written in Struts 1/2
- Migration to Spring MVC is risky and expensive for stable legacy systems
- CMC Global does enterprise/outsourcing work — maintaining legacy systems is common
