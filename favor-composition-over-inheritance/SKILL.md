---
name: favor-composition-over-inheritance
description: Guide for replacing fragile implementation inheritance with composition and forwarding in Java. Use this skill when a subclass depends on superclass internals, double-counts behavior, leaks parent API flaws, or should wrap an interface instead of extending a concrete class.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - design
tags:
  - composition
  - inheritance
  - decorator
  - forwarding
---

# favor-composition-over-inheritance

## Purpose

Help redesign Java APIs so they use composition and forwarding instead of brittle implementation inheritance.
Use it when a subclass relies on superclass self-use details, inherits methods it does not truly want, or should decorate behavior around an interface.

## Core Capabilities

### 1. Detect Fragile Inheritance

- Find subclasses that extend concrete classes only to tweak one or two behaviors
- Explain how superclass self-use can break overridden methods
- Identify cases where inherited API surface is larger or stranger than the real domain model needs

### 2. Replace Inheritance With Composition

Provide the standard direction:

```java
public final class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount;

    public InstrumentedSet(Set<E> delegate) {
        super(delegate);
    }
}
```

- Hold the real implementation in a private field or forwarding base class
- Forward normal behavior to the wrapped object
- Add only the extra behavior your type actually owns

### 3. Preserve Flexibility Through Interfaces

- Wrap an interface such as `Set`, not a single concrete implementation such as `HashSet`
- Allow the same decorator to work with `HashSet`, `TreeSet`, or any other compatible implementation
- Keep superclass implementation changes from silently changing subclass behavior

## How to Use

1. Ask whether the new type is really a subtype of the parent class.
2. If the answer is not a clean “yes”, stop extending the concrete class.
3. Introduce a wrapped field or forwarding base class around the relevant interface.
4. Re-add only the extra behavior that belongs to the wrapper.
5. Add tests that prove the wrapper behaves correctly with multiple delegate implementations.

## Key Concepts

### Why Concrete-Class Inheritance Is Fragile

A subclass can depend on implementation details the parent never promised to keep stable.
Once the parent changes how methods call each other, the subclass may break without changing a single line.

### Why Composition Is More Stable

With composition, the wrapped object is just an implementation detail.
Your wrapper chooses what to expose and what extra behavior to add, without binding correctness to the parent's internal call graph.

### Why Forwarding Is Worth the Boilerplate

Forwarding methods take a little setup, but they buy back control.
They also let the same wrapper decorate many implementations instead of coupling the design to one concrete class.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of inheritance, interfaces, and collection types
- JUnit 5 for regression tests

## Important Notes

- Extending a concrete class across package boundaries is risky unless the class was explicitly designed for inheritance
- Wrappers are often a better fit when the goal is instrumentation, validation, logging, counting, caching, or access control
- Composition lets you hide parent API flaws instead of inheriting them forever

## Best Practices

- Prefer interfaces plus wrapped delegates
- Use forwarding classes when many methods must be passed through unchanged
- Keep wrapper state local to the wrapper instead of mixing it into inherited behavior
- Test the wrapper with more than one delegate implementation

## Success Criteria

- The redesigned type no longer depends on superclass self-use details
- Extra behavior works correctly for bulk operations such as `addAll`
- The wrapper can decorate multiple implementations through a shared interface
- Tests prove composition avoids the regression that inheritance introduced
