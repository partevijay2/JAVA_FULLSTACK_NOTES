# Singleton Pattern

## Intent
Ensure a class has only one instance and provide a global point of access to it.

## When to use
- Shared resources: logger, configuration, thread-pool manager, connection manager, caching manager.
- When exactly one object is needed to coordinate actions across the system.

## Key Variants (pros / cons)
- Eager initialization
  - Pros: Simple, thread-safe due to classloader semantics.
  - Cons: Instance created even if never used.

- Static-holder (Initialization-on-demand holder idiom)
  - Pros: Lazy, thread-safe, simple. Recommended for most cases where a class must be used.
  - Cons: Slightly less explicit than eager initialization.

- Double-checked locking (with volatile)
  - Pros: Lazy and efficient.
  - Cons: More complex; must use `volatile` and correct synchronization.

- Enum singleton
  - Pros: Best safety for serialization and against reflection; concise.
  - Cons: Less flexible if you need to extend a class or lazily initialize complex state.

## Common Pitfalls and Mitigations
- Thread-safety: use static-holder, enum, or proper synchronization with `volatile`.
- Serialization: prefer enum singletons or implement `readResolve`.
- Reflection: enum resists reflective construction; otherwise defend in constructor.
- Testing: singletons introduce global state — prefer dependency injection or allow controlled reset in tests.
- Overuse: don't use singletons where a well-scoped dependency would be better.

## Examples

### Static-holder (recommended class-based lazy)
```java
// SingletonStaticHolder.java
public class SingletonStaticHolder {
    private SingletonStaticHolder() {
        // prevent instantiation
    }

    private static class Holder {
        static final SingletonStaticHolder INSTANCE = new SingletonStaticHolder();
    }

    public static SingletonStaticHolder getInstance() {
        return Holder.INSTANCE;
    }

    public void doSomething() {
        System.out.println("SingletonStaticHolder doing something");
    }
}
```

### Enum singleton (best-practice for most simple singletons)
```java
// SingletonEnum.java
public enum SingletonEnum {
    INSTANCE;

    public void doSomething() {
        System.out.println("SingletonEnum doing something");
    }
}
```

### Double-checked locking (Java 5+)
```java
// SingletonDoubleChecked.java
public class SingletonDoubleChecked {
    private static volatile SingletonDoubleChecked instance;

    private SingletonDoubleChecked() {
        if (instance != null) {
            throw new IllegalStateException("Already initialized");
        }
    }

    public static SingletonDoubleChecked getInstance() {
        if (instance == null) {
            synchronized (SingletonDoubleChecked.class) {
                if (instance == null) {
                    instance = new SingletonDoubleChecked();
                }
            }
        }
        return instance;
    }

    public void doSomething() {
        System.out.println("SingletonDoubleChecked doing something");
    }
}
```

## Exercises
1. Implement a Logger singleton (lazy) using the static-holder idiom and use it from multiple threads.
2. Implement a Settings/Configuration singleton that reads from a file once (lazy) and test concurrent access.
3. Convert the Logger to an enum singleton and verify serialization/deserialization preserves identity.
4. Write a unit test that ensures only one instance is created under heavy multi-threaded getInstance() calls.

## Production Checklist
- Prefer enum singleton if simple singleton behavior is required.
- If lazy initialization is required and you need a class (not enum), use the static-holder idiom.
- Avoid global mutable state; prefer dependency injection when possible.
- Add readResolve or other protections if you use classic singletons and serialization is involved.
- Consider lifecycle and cleanup: singletons live for JVM lifetime unless designed otherwise.

---

Created by GitHub Copilot assistant — concise notes and examples for the Singleton pattern.
