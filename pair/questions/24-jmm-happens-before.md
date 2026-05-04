# Q28 — Java Memory Model: happens-before, volatile, synchronized

```java
public class ConfigCache {
    private Map<String, String> config;   // not volatile, not final
    private boolean ready = false;

    // called once at startup by a loader thread
    public void load() {
        Map<String, String> m = new HashMap<>();
        m.put("region", "eu-west-1");
        m.put("timeout", "30");
        this.config = m;
        this.ready = true;
    }

    // called by request threads
    public String get(String key) {
        if (!ready) throw new IllegalStateException("not ready");
        return config.get(key);
    }
}
```

## Questions

1. Under the Java Memory Model, list two concrete bugs a request thread could observe when calling `get("region")` after the loader thread has finished `load()`. Be specific about *what value* the request thread sees, not just "it's broken".
2. Define "happens-before". Which JMM rules establish happens-before between two threads? Name at least four.
3. Fix `ConfigCache` three different ways — `volatile`, `synchronized`, and `final` + safe publication. What are the trade-offs of each?
4. `AtomicReference` vs `volatile` — when does `volatile` suffice and when do you need CAS?
5. Is a `HashMap` published via a `volatile` reference safe to read concurrently once published? What about `ConcurrentHashMap`?
