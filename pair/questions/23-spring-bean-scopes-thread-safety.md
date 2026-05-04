# Q27 — Spring Bean Scopes & Thread Safety

Classic Spring (Framework, XML-configured), not Spring Boot.

```xml
<bean id="userService" class="com.acme.UserService"/>
<bean id="requestTracker" class="com.acme.RequestTracker"/>
<bean id="userController" class="com.acme.UserController">
    <property name="userService" ref="userService"/>
    <property name="tracker" ref="requestTracker"/>
</bean>
```

```java
public class RequestTracker {
    private String currentUserId;
    private long requestStartTime;

    public void start(String userId) {
        this.currentUserId = userId;
        this.requestStartTime = System.currentTimeMillis();
    }

    public long elapsed() { return System.currentTimeMillis() - requestStartTime; }
    public String getUserId() { return currentUserId; }
}

public class UserController {
    private UserService userService;
    private RequestTracker tracker;

    public ModelAndView handle(HttpServletRequest req) {
        tracker.start(req.getParameter("uid"));
        User u = userService.find(tracker.getUserId());
        // ... 50ms of work ...
        log.info("user {} took {}ms", tracker.getUserId(), tracker.elapsed());
        return new ModelAndView("profile", "user", u);
    }
}
```

## Questions

1. What is the default scope of a Spring bean, and what bug does `RequestTracker` have in production under concurrent traffic? Give a concrete example of what a user could see in the logs.
2. Name the five standard bean scopes in Spring (web-aware included). Which one would you change `RequestTracker` to, and why?
3. If you just change `RequestTracker` to `scope="request"`, the app will fail to start (or fail on first request). Why? And what are the two ways to fix it — one XML-based, one annotation-based?
4. Would making `RequestTracker`'s fields `ThreadLocal` be a valid alternative? What's the downside compared to using request scope?
