---
name: dependency-injection
description: Demonstrates the principle that dependency injection is superior to hardwiring resources, with practical examples of SpellChecker and MosaicFactory implementations.
version: 1.0.0
author: AI Assistant
categories:
  - design-patterns
  - best-practices
tags:
  - dependency-injection
  - solid-principles
  - testability
  - factory-pattern
  - java
---

# Dependency Injection Skill

## Purpose

This skill demonstrates the fundamental principle from Effective Java (Item 5): **"Prefer dependency injection to hardwiring resources"**. It provides a complete working example that shows why dependency injection is superior to static utility classes and singletons when a class depends on one or more underlying resources.

### When to Use This Skill

Use this skill when you need to:
- Understand the benefits of dependency injection over hardwiring
- Design classes that depend on underlying resources
- Improve code flexibility and testability
- Implement the factory method pattern with Supplier<T>
- Teach or demonstrate dependency injection principles

## Core Capabilities

### 1. Demonstration of Anti-Patterns

Shows two common anti-patterns and their limitations:

- **Static Utility Class (`SpellCheckerStatic`)**
  - Hardcodes dictionary dependency
  - Assumes only one dictionary exists
  - Impossible to test with different dictionaries
  - Inflexible for multi-language scenarios

- **Singleton Pattern (`SpellCheckerSingleton`)**
  - Single instance limits flexibility
  - Dictionary fixed at construction time
  - Global state complicates testing
  - Cannot support multiple dictionaries

### 2. Correct Implementation with DI

Provides the proper dependency injection approach (`SpellChecker`):
- Accepts dictionary through constructor
- Supports any Lexicon implementation
- Enables easy testing with mock dictionaries
- Allows multiple independent instances
- Thread-safe with immutable dependencies

### 3. Factory Pattern with Supplier<T>

Demonstrates advanced DI pattern (`MosaicFactory`):
- Uses `Supplier<T>` as factory interface
- Enables lazy resource creation
- Supports dynamic type selection
- Integrates seamlessly with Java 8+ features

### 4. Comprehensive Testing

Shows how DI improves testability:
- Unit tests with mock dictionaries
- Multi-language support tests
- Concurrent access tests
- Custom dictionary implementation tests

## Key Concepts

### Dependency Injection (DI)

The practice of passing resources (dependencies) to a class when creating instances, rather than having the class create them directly.

**Benefits:**
- **Flexibility**: Support different resource implementations
- **Testability**: Easy to inject mock dependencies
- **Reusability**: Same class works in different contexts
- **Thread-safety**: Immutable dependencies are inherently thread-safe

### Hardwiring Resources

The anti-pattern of directly creating or managing dependencies within a class using:
- Static utility classes
- Singleton pattern
- Direct instantiation in constructors

**Problems:**
- Inflexible: Cannot change resources
- Untestable: Cannot substitute mock dependencies
- Tight coupling: Class controls its dependencies

### Factory Method Pattern

A creational pattern where a factory (often `Supplier<T>` in Java 8+) creates instances of a type, allowing:
- Lazy creation (create only when needed)
- Dynamic type selection
- Complex creation logic encapsulation

### Supplier<T> Interface

Java 8 functional interface representing a factory:
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

## How to Use

### Basic Dependency Injection

```java
// Create a dictionary dependency
Lexicon englishDict = Dictionary.english(Set.of("hello", "world", "java"));

// Inject dictionary through constructor
SpellChecker checker = new SpellChecker(englishDict);

// Use the checker
boolean isValid = checker.isValid("hello"); // true
```

### Multi-Language Support

```java
// Create checkers for different languages
Lexicon englishDict = Dictionary.english(Set.of("hello"));
Lexicon chineseDict = Dictionary.chinese(Set.of("你好"));

SpellChecker englishChecker = new SpellChecker(englishDict);
SpellChecker chineseChecker = new SpellChecker(chineseDict);

// Each instance works independently
englishChecker.isValid("hello");    // true
chineseChecker.isValid("你好");      // true
```

### Testing with Mock Dependencies

```java
// Create a mock dictionary for testing
Lexicon mockDict = new Lexicon() {
    @Override
    public boolean contains(String word) {
        return "test".equals(word);
    }
    
    @Override
    public String getLanguage() {
        return "mock";
    }
};

// Inject mock for unit testing
SpellChecker testChecker = new SpellChecker(mockDict);
testChecker.isValid("test"); // true (controlled behavior)
```

### Factory Pattern with Supplier

```java
// Use method reference as factory
MosaicFactory plainFactory = new MosaicFactory(PlainTile::new);

// Use lambda for parameterized creation
MosaicFactory coloredFactory = new MosaicFactory(
    () -> new ColoredTile("blue")
);

// Create mosaics
String mosaic = plainFactory.createMosaic(3, 3);
```

## Prerequisites

### Required Knowledge
- Java programming fundamentals
- Basic understanding of object-oriented design
- Familiarity with interfaces and polymorphism

### Required Tools
- Java 25 (with preview features enabled)
- Gradle 9.2.1 or later
- JUnit 5 for testing

### Project Setup

1. **Clone the project**:
   ```bash
   cd 20260310-dependency-injection/sample/dependency-injection
   ```

2. **Build the project**:
   ```bash
   ./gradlew clean build
   ```

3. **Run tests**:
   ```bash
   ./gradlew test
   ```

## Important Notes

### Design Principles

1. **Prefer Constructor Injection**
   - Make dependencies explicit in the constructor
   - Validate dependencies with `Objects.requireNonNull()`
   - Store dependencies in final fields for immutability

2. **Use Interfaces for Dependencies**
   - Abstract dependencies with interfaces (e.g., `Lexicon`)
   - Allows different implementations
   - Enables mock objects for testing

3. **Keep Dependencies Immutable**
   - Use final fields for injected dependencies
   - Ensures thread-safety
   - Prevents accidental modification

4. **Avoid Static State**
   - Don't use static fields for dependencies
   - Static state makes testing difficult
   - Leads to global state problems

### When DI Might Be Overkill

Dependency injection frameworks (like Spring, Guice) may be unnecessary for:
- Simple applications with few dependencies
- Projects without complex testing requirements
- Situations where a single implementation is guaranteed

### Common Mistakes to Avoid

1. **Hardcoding Dependencies in Constructors**
   ```java
   // BAD: Hardcoded dependency
   public SpellChecker() {
       this.dictionary = new Dictionary("en", Set.of("hello"));
   }
   
   // GOOD: Injected dependency
   public SpellChecker(Lexicon dictionary) {
       this.dictionary = Objects.requireNonNull(dictionary);
   }
   ```

2. **Using Mutable Dependencies**
   ```java
   // BAD: Mutable dependency can change unexpectedly
   private Lexicon dictionary; // not final
   
   // GOOD: Immutable dependency
   private final Lexicon dictionary;
   ```

3. **Creating Dependencies in Static Methods**
   ```java
   // BAD: Static factory creates dependency
   public static SpellChecker create() {
       return new SpellChecker(new Dictionary("en", words));
   }
   
   // GOOD: Let client inject dependency
   public SpellChecker(Lexicon dictionary) { ... }
   ```

## Best Practices

### 1. Constructor Injection Pattern

Always use constructor injection for mandatory dependencies:

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
}
```

### 2. Factory Injection for Variable Dependencies

Use `Supplier<T>` for dependencies that:
- Need lazy creation
- May have different implementations
- Require complex creation logic

```java
public class MosaicFactory {
    private final Supplier<? extends Tile> tileFactory;
    
    public MosaicFactory(Supplier<? extends Tile> tileFactory) {
        this.tileFactory = Objects.requireNonNull(tileFactory);
    }
}
```

### 3. Interface-Based Design

Define dependencies as interfaces:

```java
// Interface for the dependency
public interface Lexicon {
    boolean contains(String word);
    String getLanguage();
}

// Implementation can vary
public class Dictionary implements Lexicon { ... }
public class MockDictionary implements Lexicon { ... }
```

### 4. Testability-Driven Design

Design with testing in mind:

```java
@Test
void testWithMockDictionary() {
    // Create controlled mock
    Lexicon mockDict = new MockDictionary("test");
    
    // Inject mock for predictable behavior
    SpellChecker checker = new SpellChecker(mockDict);
    
    // Test with certainty
    assertTrue(checker.isValid("expected-word"));
}
```

### 5. Use Bounded Wildcards with Suppliers

When using `Supplier<T>`, apply bounded wildcards for flexibility:

```java
// Allows any subtype of Tile
public MosaicFactory(Supplier<? extends Tile> tileFactory) {
    this.tileFactory = tileFactory;
}
```

## Success Criteria

A successful implementation should meet these criteria:

### Code Quality
- ✅ All dependencies injected through constructors
- ✅ Dependencies stored in final fields (immutable)
- ✅ Null checks on all injected dependencies
- ✅ Dependencies abstracted as interfaces

### Flexibility
- ✅ Multiple instances with different dependencies can coexist
- ✅ Easy to switch implementations
- ✅ Supports different configurations

### Testability
- ✅ Easy to inject mock dependencies
- ✅ Unit tests don't depend on external resources
- ✅ Tests are fast and reliable

### Build Verification
- ✅ `./gradlew clean build` succeeds
- ✅ All tests pass
- ✅ No compiler warnings

### Documentation
- ✅ Classes have clear Javadoc comments
- ✅ Dependencies documented in constructor parameters
- ✅ Usage examples provided

## Real-World Applications

### Examples of DI in Practice

1. **Database Access**
   - Inject `DataSource` instead of creating connections
   - Easy to switch between production and test databases

2. **Web Services**
   - Inject `HttpClient` for API calls
   - Mock client for testing without network

3. **Configuration**
   - Inject `Configuration` objects
   - Different configs for dev/test/prod

4. **Logging**
   - Inject `Logger` instances
   - Redirect logs for testing

## Troubleshooting

### Problem: NullPointerException in Constructor

**Cause**: Forgot to validate injected dependency

**Solution**:
```java
public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary, "dictionary must not be null");
}
```

### Problem: Tests Fail Due to Shared State

**Cause**: Using singleton or static state

**Solution**: Convert to instance-based design with constructor injection

### Problem: Cannot Use Different Implementations

**Cause**: Dependency is concrete class, not interface

**Solution**: Extract interface and inject it instead:
```java
// Before: Hard to change
public SpellChecker(Dictionary dictionary) { ... }

// After: Flexible
public SpellChecker(Lexicon dictionary) { ... }
```

## References

- **Effective Java, Third Edition** - Item 5: Prefer dependency injection to hardwiring resources
- **Design Patterns** - Factory Method Pattern (Gamma et al.)
- **Java 8 API** - `java.util.function.Supplier<T>`
- **Clean Code** - Robert C. Martin (Dependency Injection chapter)

## License

This skill is part of the Effective Java Skills collection and is available under the MIT License.
