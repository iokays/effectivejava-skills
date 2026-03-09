# Singleton Pattern Skill

## Overview

This Skill demonstrates the Singleton pattern, one of the simplest yet most important creational design patterns in Java. A singleton is a class that is instantiated exactly once.

The implementation showcases three approaches:
1. **Public final field** - Simple and direct
2. **Static factory method** - Flexible and testable
3. **Enum singleton** - Recommended approach with built-in serialization support

## Core Capabilities

### 1. Three Implementation Approaches
- Side-by-side comparison of three singleton implementations
- Demonstration of pros and cons of each approach
- Handling of serialization and reflection attacks

### 2. Thread Safety
- All three approaches are thread-safe by design
- Enum singleton is inherently thread-safe
- Eager initialization guarantees a single instance

### 3. Serialization Support
- Enum singleton handles serialization automatically
- Static factory method requires `readResolve()` method
- Public final field needs defensive measures

## Usage

### Public Final Field Approach

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}

// Usage
Elvis elvis = Elvis.INSTANCE;
```

### Static Factory Method Approach

```java
public class Elvis implements Serializable {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { ... }
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
    
    // Prevent deserialization from creating new instances
    private Object readResolve() {
        return INSTANCE;
    }
}

// Usage
Elvis elvis = Elvis.getInstance();
```

### Enum Singleton (Recommended)

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}

// Usage
Elvis elvis = Elvis.INSTANCE;
```

## Project Structure

```
singleton-pattern-demo/
├── src/main/java/com/iokays/sample/singleton/
│   ├── field/
│   │   └── Elvis.java
│   ├── factory/
│   │   └── Elvis.java
│   └── enum_singleton/
│       └── Elvis.java
└── src/test/java/com/iokays/sample/singleton/
    ├── field/ElvisTest.java
    ├── factory/ElvisTest.java
    └── enum_singleton/ElvisTest.java
```

## Prerequisites

- **Java 25** or higher
- **Gradle 9.2.1** with Kotlin DSL
- **JUnit 5** for testing

## When to Use Singleton Pattern

Use Singleton pattern when:
1. **Exactly one instance needed** - Class must have exactly one instance
2. **Global access required** - Instance must be accessible from a well-known access point
3. **Stateless objects** - Object typically represents a stateless component
4. **System components** - Represents a unique system component

## Advantages of Each Approach

### Public Final Field
- ✅ Simple and clear API
- ✅ Clearly shows singleton nature
- ✅ Easy to understand
- ❌ No flexibility for future changes
- ❌ Requires extra code for serialization

### Static Factory Method
- ✅ Flexibility to change implementation
- ✅ Can return different instances (e.g., per-thread)
- ✅ Can use method reference as Supplier
- ✅ Supports generic singleton factory
- ❌ Requires readResolve() for serialization

### Enum Singleton (Recommended)
- ✅ Most concise implementation
- ✅ Automatic serialization support
- ✅ Guaranteed single instance even with reflection
- ✅ Thread-safe by design
- ❌ Cannot inherit from other classes

## Key Concepts

### 1. Eager Initialization
All three approaches use eager initialization - instance is created when class is loaded:

```java
private static final Elvis INSTANCE = new Elvis();
```

This is thread-safe because JVM guarantees class initialization is synchronized.

### 2. Private Constructor
Constructor must be private to prevent external instantiation:

```java
private Elvis() {
    // Defense against reflection attacks
    if (INSTANCE != null) {
        throw new IllegalStateException("Singleton already initialized");
    }
}
```

### 3. Serialization Safety
For serializable singletons, must prevent deserialization from creating new instances:

```java
private Object readResolve() {
    return INSTANCE;
}
```

Enum singleton handles this automatically.

### 4. Reflection Defense
To prevent reflection attacks, check if instance exists in constructor:

```java
private Elvis() {
    if (INSTANCE != null) {
        throw new IllegalStateException("Already initialized");
    }
}
```

Enum singleton is immune to reflection attacks.

## Testing

Run all tests:
```bash
./gradlew clean test
```

Expected output:
```
> Task :test
BUILD SUCCESSFUL
```

All tests should pass, verifying:
- Singleton guarantee - same instance returned every time
- Correct behavior - methods work as expected
- Enum properties - single enum value

## Best Practices

1. **Prefer enum singleton** - Unless you need to inherit from another class
2. **Make constructor private** - Always, no exceptions
3. **Handle serialization** - If not using enum, implement readResolve()
4. **Defend against reflection** - If security matters, add checks in constructor
5. **Consider testability** - Static factory method is most testable

## Common Mistakes

1. ❌ **Lazy initialization without synchronization** - Not thread-safe
2. ❌ **Double-checked locking without volatile** - Broken pattern
3. ❌ **Forgetting readResolve()** - Serialization breaks singleton
4. ❌ **Not handling reflection** - Attacker can create multiple instances
5. ❌ **Using singleton for wrong reasons** - Don't use as global variables

## When Not to Use Singleton

1. **Testing difficulty** - Hard to mock singleton classes
2. **Hidden dependencies** - Dependencies are not explicit
3. **Global state** - Can cause subtle bugs in concurrent code
4. **Overuse** - Not every class needs to be singleton
5. **Premature optimization** - Start simple, optimize later

## Related Patterns

- **Factory Pattern** - Can return singleton instance
- **Builder Pattern** - Can build singleton object
- **Dependency Injection** - Often a better alternative to singleton

## References

- *Effective Java* by Joshua Bloch, Item 3
- [Singleton Pattern - Wikipedia](https://en.wikipedia.org/wiki/Singleton_pattern)
- [Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/)

## Notes

- Singleton is one of the most overused and abused patterns
- Consider dependency injection frameworks as alternatives
- Enum singleton is the best approach for most cases
- Testing singletons can be challenging - consider testability

## Performance Considerations

- Eager initialization happens at class load time
- No runtime overhead for instance creation
- No synchronization needed with eager initialization
- Minimal memory footprint (single instance)
