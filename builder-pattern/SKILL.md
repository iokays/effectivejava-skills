---
name: builder-pattern
description: Demonstrate the Builder pattern for handling constructors with many parameters, including telescoping constructor, JavaBeans, and Builder pattern variations
version: 1.0.0
author: iokays
categories:
  - design-patterns
  - java
  - best-practices
tags:
  - builder-pattern
  - effective-java
  - constructor
  - immutable-object
  - fluent-api
---

# Builder Pattern Skill

## Overview

This skill demonstrates the Builder pattern, one of the most important creational design patterns in Java. It addresses the problem of creating objects when constructors have many parameters, especially optional ones.

The implementation showcases three approaches:
1. **Telescoping Constructor Pattern** - Traditional approach with multiple constructors
2. **JavaBeans Pattern** - Using setters for configuration
3. **Builder Pattern** - The recommended approach combining safety and readability

## Core Capabilities

### 1. Nutritional Facts Example
- Compare three constructor patterns side by side
- Demonstrate the problems with telescoping constructors
- Show the thread-safety issues with JavaBeans pattern
- Illustrate the benefits of Builder pattern

### 2. Class Hierarchy Support
- Abstract base class with generic Builder
- Concrete subclasses (NyPizza, Calzone)
- Recursive type parameters for fluent API
- Covariant return types

### 3. Immutability Support
- Create immutable objects easily
- Validate parameters before construction
- Defensive copying for collections

## Usage

### Basic Builder Pattern

```java
// Create a NutritionFacts instance with Builder
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

### Class Hierarchy Builder

```java
// Create a New York style pizza
NyPizza pizza = new NyPizza.Builder(SMALL)
    .addTopping(SAUSAGE)
    .addTopping(ONION)
    .build();

// Create a Calzone with sauce inside
Calzone calzone = new Calzone.Builder()
    .addTopping(HAM)
    .sauceInside()
    .build();
```

### Comparing Patterns

```java
// Telescoping Constructor - hard to read
NutritionFactsTelescoping t1 = new NutritionFactsTelescoping(240, 8, 100, 0, 35, 27);

// JavaBeans - mutable, not thread-safe
NutritionFactsJavaBeans j1 = new NutritionFactsJavaBeans();
j1.setServingSize(240);
j1.setServings(8);
j1.setCalories(100);

// Builder - clean and safe
NutritionFacts b1 = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .build();
```

## Project Structure

```
builder-pattern-demo/
├── src/main/java/com/iokays/sample/builder/
│   ├── telescoping/
│   │   └── NutritionFactsTelescoping.java
│   ├── javabeans/
│   │   └── NutritionFactsJavaBeans.java
│   ├── builder/
│   │   └── NutritionFacts.java
│   └── hierarchy/
│       ├── Pizza.java
│       ├── NyPizza.java
│       └── Calzone.java
└── src/test/java/com/iokays/sample/builder/
    ├── telescoping/NutritionFactsTelescopingTest.java
    ├── javabeans/NutritionFactsJavaBeansTest.java
    ├── builder/NutritionFactsTest.java
    └── hierarchy/PizzaTest.java
```

## Prerequisites

- **Java 25** or later
- **Gradle 9.2.1** with Kotlin DSL
- **JUnit 5** for testing

## When to Use Builder Pattern

Use the Builder pattern when:
1. **Many constructor parameters** - More than 4-5 parameters, especially if optional
2. **Many optional parameters** - Some or most parameters are optional
3. **Immutable objects needed** - You need thread-safe immutable objects
4. **Readability matters** - Client code should be self-documenting
5. **Validation required** - Need to validate parameter combinations

## Advantages

### vs Telescoping Constructor
- ✅ More readable - named parameters
- ✅ No parameter ordering issues
- ✅ Easy to add new optional parameters
- ❌ Slightly more verbose

### vs JavaBeans
- ✅ Immutability - objects are thread-safe
- ✅ Consistency - no partially constructed state
- ✅ Validation - can validate before construction
- ❌ Requires creating Builder object

## Key Concepts

### 1. Fluent API
Builder methods return `this` to enable method chaining:

```java
public Builder calories(int val) {
    calories = val;
    return this;  // Enables chaining
}
```

### 2. Recursive Type Parameters
Allows subclasses to return their own type:

```java
abstract static class Builder<T extends Builder<T>> {
    protected abstract T self();
}
```

### 3. Covariant Return Types
Subclass builder returns subclass type:

```java
@Override
public NyPizza build() {
    return new NyPizza(this);
}
```

### 4. Defensive Copying
Protect internal collections from modification:

```java
Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();
}
```

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

All tests should pass, demonstrating:
- Correct construction of objects
- Proper default values
- Immutability guarantees
- Thread-safety considerations

## Best Practices

1. **Always call build()** - Don't forget the final build step
2. **Validate in build()** - Check invariants before creating object
3. **Make objects immutable** - All fields should be final
4. **Use defensive copy** - Copy mutable parameters
5. **Consider validation** - Throw IllegalArgumentException for invalid states

```java
public NutritionFacts build() {
    if (servingSize <= 0) {
        throw new IllegalArgumentException("servingSize must be positive");
    }
    return new NutritionFacts(this);
}
```

## Common Mistakes to Avoid

1. ❌ **Forgetting to call build()** - Creates a Builder, not the object
2. ❌ **Reusing Builder without care** - Builder state persists
3. ❌ **Not making fields final** - Loses immutability
4. ❌ **Skipping validation** - Allows invalid states
5. ❌ **Making Builder mutable** - Should be stateless between builds

## Related Patterns

- **Factory Pattern** - For creating objects without specifying exact class
- **Prototype Pattern** - For cloning existing objects
- **Composite Pattern** - Often used with Builders for complex structures

## References

- *Effective Java* by Joshua Bloch, Item 2
- [Builder Pattern - Wikipedia](https://en.wikipedia.org/wiki/Builder_pattern)
- [Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/)

## Notes

- The Builder pattern is one of the most commonly used patterns in modern Java development
- It's particularly useful in libraries and APIs where client code readability is important
- Consider using IDE plugins or annotation processors (like Lombok's `@Builder`) to reduce boilerplate
- The pattern works well with method references and streams

## Performance Considerations

- Builder object creation has minimal overhead
- Memory impact is negligible for most applications
- Garbage collection of Builders is straightforward
- No performance penalty compared to other patterns
