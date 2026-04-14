# 37. Spring Bean Lifecycle

From "the container decides to create this bean" to "the container shuts it down." Knowing the order is a classic interview question because it explains where to plug custom logic (init, proxying, cleanup).

---

## The phases (in order)

```
 1. Instantiation                 → constructor called (or factory method)
 2. Populate properties           → dependency injection (@Autowired, setters)
 3. Aware callbacks               → BeanNameAware, BeanFactoryAware, ApplicationContextAware
 4. BeanPostProcessor.postProcessBeforeInitialization
 5. Initialization
       a. @PostConstruct
       b. InitializingBean.afterPropertiesSet()
       c. custom init-method (@Bean(initMethod="..."))
 6. BeanPostProcessor.postProcessAfterInitialization     ← proxies created here (AOP, @Transactional)
 7. Bean is ready — used by the application
 ---------- context shutdown ----------
 8. Destruction (singletons only)
       a. @PreDestroy
       b. DisposableBean.destroy()
       c. custom destroy-method (@Bean(destroyMethod="..."))
```

---

## Code example covering every hook

```java
@Component
public class DemoBean
        implements BeanNameAware, InitializingBean, DisposableBean {

    private Dependency dep;

    public DemoBean() {                             // 1. instantiation
        System.out.println("constructor");
    }

    @Autowired
    public void setDep(Dependency dep) {            // 2. DI
        this.dep = dep;
    }

    @Override
    public void setBeanName(String name) {          // 3. aware
        System.out.println("beanName=" + name);
    }

    @PostConstruct
    public void postConstruct() {                   // 5a
        System.out.println("@PostConstruct");
    }

    @Override
    public void afterPropertiesSet() {              // 5b
        System.out.println("afterPropertiesSet");
    }

    public void customInit() {                      // 5c
        System.out.println("custom init");
    }

    @PreDestroy
    public void preDestroy() {                      // 8a
        System.out.println("@PreDestroy");
    }

    @Override
    public void destroy() {                         // 8b
        System.out.println("destroy");
    }

    public void customDestroy() {                   // 8c
        System.out.println("custom destroy");
    }
}

@Configuration
class Cfg {
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    DemoBean demoBean() { return new DemoBean(); }
}
```

Output (startup):
```
constructor
beanName=demoBean
@PostConstruct
afterPropertiesSet
custom init
```

---

## Where `BeanPostProcessor` fits
`BeanPostProcessor` runs around the init phase — **for every bean**. This is how Spring implements:
- **AOP proxies** (`@Transactional`, `@Async`) — wrapped in `postProcessAfterInitialization`.
- `@Autowired` resolution — handled by `AutowiredAnnotationBeanPostProcessor`.
- `@PostConstruct` / `@PreDestroy` — handled by `CommonAnnotationBeanPostProcessor`.

> Takeaway: if you see `this.someInternalCall()` **not** being intercepted by `@Transactional`, it's because the proxy is added *after* init, and internal self-calls bypass the proxy. (See Q29.)

---

## Scope matters for destruction
- **Singleton** → Spring manages full lifecycle, including destruction on context close.
- **Prototype** → Spring creates + initializes, then **hands it over**. It will **not** call destroy callbacks — you must clean up yourself.

---

## Preferred modern style
Use **JSR-250 annotations** (`@PostConstruct`, `@PreDestroy`) — portable, no Spring interface coupling, clearest intent. Avoid implementing `InitializingBean`/`DisposableBean` in new code.

---

## Interview one-liner
> "Spring instantiates the bean, injects dependencies, calls `Aware` interfaces, runs `BeanPostProcessor` before-init, runs init callbacks (`@PostConstruct` → `afterPropertiesSet` → custom init), runs `BeanPostProcessor` after-init — which is where AOP proxies get applied — then the bean serves requests. On shutdown, singletons get `@PreDestroy` → `destroy()` → custom destroy."
