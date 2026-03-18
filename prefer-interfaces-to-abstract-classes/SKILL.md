---
name: prefer-interfaces-to-abstract-classes
description: Guide for preferring interfaces over abstract classes in Java, especially when defining types, mixins, and optional skeletal implementations. Use this skill when multiple class hierarchies should share one capability contract without being forced under the same abstract superclass.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - design
tags:
  - interface
  - abstract-class
  - mixin
  - skeletal-implementation
---

# prefer-interfaces-to-abstract-classes

## Purpose

Help redesign Java APIs so that interfaces define the primary type contract, while abstract classes are used only as optional skeletal implementations when they truly reduce boilerplate.
Use it when multiple class hierarchies need to share one capability, or when an abstract base class is constraining type reuse too early.

## Core Capabilities

### 1. Identify Type Contracts That Belong In Interfaces

- Find behaviors that should be available to unrelated class hierarchies
- Detect cases where an abstract superclass is being used just to define a capability contract
- Explain why mixin-style behavior is better modeled as an interface

### 2. Model Optional Capabilities As Interfaces

Provide the standard direction:

```java
public interface Taggable {
    Set<String> tags();

    default boolean hasTag(String tag) {
        return tags().contains(tag);
    }
}
```

- Put the type definition in the interface
- Add small default methods when they are obvious and safe
- Let existing classes adopt the interface without changing their superclass

### 3. Add Skeletal Implementations Only As Helpers

- Use an abstract skeletal implementation only when it significantly reduces repeated code
- Keep the interface as the real public contract
- Ensure classes can still implement the interface directly if they already have another superclass

## How to Use

1. Ask whether the behavior is a type contract or a code-sharing convenience.
2. If it is a type contract, define it as an interface first.
3. Add default methods only for simple derived behavior.
4. Introduce a skeletal abstract class only if it removes meaningful boilerplate.
5. Keep the skeletal class optional, not mandatory for all implementations.

## Key Concepts

### Why Interfaces Make Better Public Types

Any class can implement an interface regardless of where it already sits in the class hierarchy.
That makes interfaces ideal for capabilities that should cross-cut multiple existing types.

### Why Interfaces Are Good Mixins

Interfaces let a class say “I also support this behavior” without giving up its main inheritance path.
That is exactly what mixin-like optional behavior needs.

### Why Skeletal Implementations Still Matter

Abstract skeletal classes can be useful, but only as helpers layered under an interface.
They should reduce boilerplate, not become the only way to participate in the type.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of interfaces, abstract classes, and single inheritance
- JUnit 5 for regression tests

## Important Notes

- Interfaces should define the main type whenever multiple implementations are expected
- Default methods are for simple reusable behavior, not for hidden state
- Abstract classes are helpful for skeletal implementations, but too rigid as the only public type definition

## Best Practices

- Put capability contracts in interfaces
- Use default methods for small obvious behavior
- Keep skeletal implementations optional and lightweight
- Test that unrelated class hierarchies can still share the same interface contract

## Success Criteria

- Unrelated class hierarchies can adopt the same capability contract
- The interface remains usable without inheriting a specific abstract class
- Skeletal implementations reduce boilerplate without becoming mandatory
- Tests prove the design works across multiple existing base classes
