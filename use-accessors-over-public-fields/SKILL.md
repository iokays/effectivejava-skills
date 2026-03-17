---
name: use-accessors-over-public-fields
description: Guide for replacing public fields in Java public classes with accessor methods. Use this skill when a class exposes mutable state directly, needs validation or invariants, or should preserve flexibility to evolve its internal representation.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - encapsulation
tags:
  - getters
  - setters
  - public-fields
  - api-design
---

# use-accessors-over-public-fields

## Purpose

Explain why public classes should usually expose behavior through accessor methods instead of public fields.
Use it when designing Java APIs, protecting invariants, adding validation, or preserving freedom to change internal representations later.

## Core Capabilities

### 1. Detect Degenerate Public Data Classes

- Find public classes that expose mutable fields directly
- Explain which invariants can be broken after construction
- Distinguish between public classes and package-private/private helpers where direct fields may be acceptable

### 2. Replace Fields With Accessors and Mutators

Provide the standard direction:

```java
private int startHour;
private int endHour;

public int getStartHour() { return startHour; }
public int getEndHour() { return endHour; }

public void setStartHour(int startHour) {
    validateRange(startHour, this.endHour);
    this.startHour = startHour;
}
```

- Keep fields private
- Validate input at construction and update time
- Reserve room for derived behavior, logging, normalization, and representation changes

### 3. Explain the Important Nuance

- Public mutable fields are the dangerous case
- Public final immutable fields are less harmful but still lock in representation choices
- Package-private or private nested classes can sometimes expose fields when they remain pure implementation details

## How to Use

1. Decide whether the class is part of the public API.
2. If it is public, hide mutable fields behind methods.
3. Move validation and invariant enforcement into constructors and mutators.
4. Keep direct field exposure only for internal helper types when the tradeoff is truly local.
5. Add tests that prove invalid state can no longer be created through the API.

## Key Concepts

### Why Accessors Matter

Accessors are not only about style.
They preserve the ability to add validation, derived behavior, caching, normalization, logging, or a new internal representation later without breaking callers.

### Why Public Fields Are Costly

A public field gives callers direct write access today and locks in your representation for tomorrow.
Once client code depends on those fields, changing the implementation becomes much more expensive.

### The Exception Case

For package-private classes or private nested classes, exposing fields can be acceptable when the scope is tightly controlled and the class exists only as an implementation detail.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of classes, fields, and method visibility
- JUnit 5 for regression tests

## Important Notes

- Public classes should not expose mutable fields directly
- Public final fields are less dangerous, but still reduce future design flexibility
- Validation should not rely on callers being disciplined after construction
- Internal helper classes may choose simpler field exposure when the scope is strictly local

## Best Practices

- Prefer private fields plus getters and setters in public mutable classes
- Put invariant checks in both constructors and mutators
- Keep the public API small and intention-revealing
- Mention explicitly when an exposed-field class is only package-private implementation detail

## Success Criteria

- Invalid state can no longer be introduced through public API calls
- The class keeps freedom to evolve its internal representation
- Tests prove that setters preserve invariants
- The article or documentation explains the difference between public API classes and local helper classes
