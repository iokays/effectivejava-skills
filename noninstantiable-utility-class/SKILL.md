---
name: noninstantiable-utility-class
description: Demonstrates how to enforce noninstantiability using private constructors in utility classes (Effective Java Item 4)
version: 1.0.0
author: AI Assistant
categories:
  - design-patterns
  - effective-java
  - best-practices
tags:
  - utility-class
  - private-constructor
  - noninstantiable
  - effective-java
  - item-4
---

# Noninstantiable Utility Class Skill

## Purpose

This skill demonstrates how to properly implement utility classes that cannot be instantiated, following Item 4 from Effective Java. Utility classes group related static methods and static fields, similar to `java.lang.Math` or `java.util.Arrays`.

## Core Capabilities

### 1. Enforcing Noninstantiability
- Private constructor with AssertionError
- Prevents accidental instantiation from within the class
- Makes the intent explicit through code

### 2. Utility Method Patterns
- Mathematical operations (MathUtils)
- String validation (StringValidator)
- General utilities (UtilityClass)

### 3. Best Practices
- `final` class modifier
- Clear documentation
- Meaningful error messages

## How to Use

### Basic Pattern

```java
// Noninstantiable utility class
public final class UtilityClass {

    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError("Utility class should not be instantiated");
    }

    // Static utility methods
    public static int max(int a, int b) {
        return a > b ? a : b;
    }
}

// Usage
int result = UtilityClass.max(3, 5);  // Returns 5
```

### Real-World Examples

**MathUtils.java** - Mathematical utilities:
```java
public final class MathUtils {
    private MathUtils() {
        throw new AssertionError("MathUtils should not be instantiated");
    }

    public static boolean isPrime(int n) { ... }
    public static long factorial(int n) { ... }
    public static int gcd(int a, int b) { ... }
}
```

**StringValidator.java** - String validation:
```java
public final class StringValidator {
    private StringValidator() {
        throw new AssertionError("StringValidator should not be instantiated");
    }

    public static boolean isValidEmail(String email) { ... }
    public static boolean isNumeric(String str) { ... }
}
```

## Key Concepts

### Why Private Constructor?

1. **Compiler generates default constructor** if no explicit constructor exists
2. **Users might accidentally instantiate** without realizing it's not intended
3. **Abstract class doesn't work** - can be subclassed and instantiated

### Why AssertionError?

```java
private UtilityClass() {
    throw new AssertionError("Utility class should not be instantiated");
}
```

- **Not strictly required** but prevents accidental calls from within the class
- **Makes intent crystal clear** - the constructor is designed not to be called
- **Fails fast** with a meaningful error message

### Why `final` Class?

- **Prevents subclassing** - all constructors must call superclass constructor
- **Documents intent** - clearly shows the class is not designed for inheritance
- **Side effect** - the private constructor also prevents subclassing

## Project Structure

```
noninstantiable-utility-class-demo/
├── src/main/java/com/iokays/sample/utility/
│   ├── UtilityClass.java       # General utility methods
│   ├── MathUtils.java          # Mathematical utilities
│   └── StringValidator.java    # String validation utilities
└── src/test/java/com/iokays/sample/utility/
    ├── UtilityClassTest.java
    ├── MathUtilsTest.java
    └── StringValidatorTest.java
```

## Prerequisites

- **Java 25** or higher
- **Gradle 9.2.1** with Kotlin DSL
- **JUnit 5** for testing

## Important Notes

### What Not To Do

❌ **Don't use abstract class**:
```java
// WRONG - can be subclassed and instantiated
public abstract class UtilityClass {
    public static int max(int a, int b) { ... }
}
```

❌ **Don't forget the private constructor**:
```java
// WRONG - compiler generates public default constructor
public class UtilityClass {
    public static int max(int a, int b) { ... }
}
```

### Testing Noninstantiability

```java
@Test
void privateConstructorShouldThrowAssertionError() throws Exception {
    Constructor<UtilityClass> constructor = UtilityClass.class.getDeclaredConstructor();
    constructor.setAccessible(true);

    InvocationTargetException exception = assertThrows(
        InvocationTargetException.class,
        constructor::newInstance
    );

    assertTrue(exception.getCause() instanceof AssertionError);
}
```

## Best Practices

1. **Always use private constructor** with AssertionError
2. **Make the class final** to prevent subclassing
3. **Add clear documentation** explaining why it cannot be instantiated
4. **Group related methods** logically within the class
5. **Keep methods static and stateless**

## Common Use Cases

### When to Use Utility Classes

1. **Grouping related static methods** - like `java.lang.Math`
2. **Operating on primitive types or arrays** - like `java.util.Arrays`
3. **Factory methods for interfaces** - like `java.util.Collections`
4. **Constants and configuration** - grouping related constants

### Examples from JDK

- `java.lang.Math` - mathematical functions
- `java.util.Arrays` - array manipulation
- `java.util.Collections` - collection utilities
- `java.nio.file.Files` - file operations

## Related Patterns

- **Factory Pattern** - utility classes often contain factory methods
- **Strategy Pattern** - utility methods can implement strategies
- **Template Method** - static methods can provide template implementations

## References

- *Effective Java* by Joshua Bloch, Item 4: "Enforce noninstantiability with a private constructor"
- [java.lang.Math](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html)
- [java.util.Arrays](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)

## Success Criteria

- ✅ Private constructor throws AssertionError when accessed via reflection
- ✅ Class is declared final
- ✅ All methods are static
- ✅ No instance state exists
- ✅ Clear documentation explains noninstantiability
- ✅ All tests pass
