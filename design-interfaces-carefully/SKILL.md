---
name: design-interfaces-carefully
description: Guide for designing Java interfaces carefully, especially before adding default methods or evolving widely implemented contracts. Use this skill when an interface may gain new methods, when default methods could violate existing implementation invariants, or when you need to test an interface across multiple implementations before publishing it.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - api-design
tags:
  - interface
  - default-method
  - evolution
  - compatibility
---

# design-interfaces-carefully

## Purpose

Help evolve Java interfaces safely by treating published interface contracts as long-term compatibility commitments.
Use it when you are designing a new interface, considering default methods, or evaluating whether a new method could silently break existing implementations.

## Core Capabilities

### 1. Detect Interface Evolution Risk

- Find default methods that assume too much about existing implementations
- Identify wrappers or synchronized/delegating implementations whose invariants a default method may bypass
- Explain when a default method is only safe for some implementations, not all

### 2. Design Interfaces For Future Implementations

Provide the standard direction:

```java
public interface FilterableSequence<E> extends Iterable<E> {
    Iterator<E> iterator();

    default boolean removeIf(Predicate<? super E> filter) {
        // only safe if every implementation can support iterator.remove()
    }
}
```

- Treat every new interface method as a compatibility decision
- Add default methods only when the implementation assumptions are genuinely universal
- Expect wrappers, synchronized adapters, and custom iterators to expose hidden edge cases

### 3. Validate Interfaces Against Multiple Implementations

- Test at least several distinct implementations before treating the interface as stable
- Include plain implementations, wrappers, and constrained variants
- Prefer explicit overrides when a wrapper must preserve extra invariants such as locking, auditing, or validation

## How to Use

1. List the existing or expected implementations of the interface.
2. Ask whether the new method preserves all their invariants.
3. If not, avoid the default method or require explicit overrides.
4. Test the interface against multiple real implementations before publishing it.
5. Document assumptions that implementers must preserve.

## Key Concepts

### Why Default Methods Are Risky On Existing Interfaces

A default method is injected into existing implementations without their author revisiting the code.
If the default implementation assumes too much about iteration, mutation, synchronization, or internal invariants, the interface may compile but fail at runtime.

### Why New Interfaces Still Need Careful Design

Even on brand-new interfaces, a casual default method can lock in assumptions that make future implementations awkward or unsafe.
The earlier you discover those assumptions, the cheaper they are to fix.

### Why Multiple Implementations Are Essential Tests

An interface is only as good as the range of implementations it can safely support.
Testing only the simplest implementation gives a false sense of safety.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of interfaces, iterators, and default methods
- JUnit 5 for regression tests

## Important Notes

- Default methods are not automatically safe just because they compile
- Existing implementations may rely on invariants the interface cannot see
- Interface evolution is much harder after release than before release

## Best Practices

- Design interfaces against multiple implementation styles, not just one concrete class
- Use default methods only for behavior whose assumptions are broadly valid
- Override default methods in wrappers when they must preserve extra constraints
- Treat published interfaces as long-lived compatibility promises

## Success Criteria

- New interface methods do not silently violate existing implementation invariants
- Default methods are only used where their assumptions are broadly safe
- Multiple implementations validate the interface design before release
- Wrapper implementations explicitly override risky inherited defaults when needed
