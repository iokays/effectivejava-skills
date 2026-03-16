---
name: override-clone-carefully
description: Guide for implementing clone safely in Java. Use this skill when a class implements Cloneable, needs a public clone method, contains mutable internal state, or should be replaced with a copy constructor or copy factory.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - object-copying
tags:
  - clone
  - cloneable
  - copy-constructor
  - mutable-state
---

# override-clone-carefully

## Purpose

Explain when Java `clone` is dangerous, how to implement it correctly when you have no better choice, and when to prefer copy constructors or copy factories.
Use it for legacy `Cloneable` code, classes with mutable internal state, and teaching material about object copying.

## Core Capabilities

### 1. Diagnose Broken clone Implementations

- Detect classes that rely on `super.clone()` but still share mutable internal state
- Explain why shallow field-by-field copying breaks invariants
- Highlight common red flags such as arrays, linked structures, or mutable references

### 2. Produce Safer clone Implementations

Provide the standard recipe:

```java
@Override
public DeepCloneReportStack clone() {
    try {
        DeepCloneReportStack result = (DeepCloneReportStack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(e);
    }
}
```

- Call `super.clone()` first
- Repair mutable state before returning
- Use covariant return types
- Hide `CloneNotSupportedException` from callers in public clone methods

### 3. Recommend Better Alternatives

- Suggest copy constructors for new code
- Suggest copy factories such as `copyOf(...)`
- Preserve programmatic access without exposing callers to the fragile `Cloneable` contract

## How to Use

1. Identify whether the class already inherits or exposes `Cloneable`.
2. List all mutable fields and internal data structures.
3. Decide whether `clone` must be supported for compatibility, or whether a copy constructor / factory is better.
4. If `clone` is required, call `super.clone()` and repair every mutable field.
5. Add tests that prove the original and copy no longer share mutable state.

## Key Concepts

### Why clone Is Fragile

`Cloneable` changes the behavior of `Object.clone()` without defining a public method in the interface.
That means callers cannot rely on a clean, explicit protocol, and implementers must follow conventions that the type system does not enforce.

### Shallow Copy vs Deep Repair

The object returned by `super.clone()` starts as a field-by-field copy.
If a field points to mutable data such as an array or linked structure, the clone still points at the same data until you replace or rebuild it.

### Why Copy Constructors Often Win

Copy constructors and copy factories are clearer, easier to evolve, work better with `final` fields, and avoid the brittle `Cloneable` protocol.
For new code, they should usually be the default choice.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of object state and mutability
- JUnit 5 for regression tests

## Important Notes

- New interfaces should not extend `Cloneable`
- New extensible classes generally should not implement `Cloneable`
- Immutable classes rarely need `clone`
- Arrays are the strongest remaining use case for the built-in clone mechanism

## Best Practices

- Prefer copy constructors or `copyOf(...)` for new code
- If you expose `clone`, return the concrete type instead of `Object`
- Test for shared mutable state explicitly
- Treat `clone` as another constructor that must preserve invariants

## Success Criteria

- Broken shallow clone behavior is reproduced and understood
- Fixed clone behavior proves state independence
- Tests cover both the failure mode and the repaired implementation
- Alternatives such as copy constructors are documented when they are the better design
