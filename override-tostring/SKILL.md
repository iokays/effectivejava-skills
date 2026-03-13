---
name: override-tostring
description: Guide for designing useful Java toString implementations. Use this skill when a class still inherits Object.toString(), logs are unreadable, collection diagnostics are hard to interpret, or a value object needs a documented string form.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - diagnostics
tags:
  - tostring
  - logging
  - debugging
  - value-object
---

# override-tostring

## Purpose

Explain when and how to override `toString` in Java so objects become readable in logs, assertions, and collections.
Use it for value objects, diagnostic-heavy domain classes, and any class whose inherited `Object.toString()` output is not useful to humans.

## Core Capabilities

### 1. Identify Missing or Weak String Representations

- Detect classes that still inherit `Object.toString()`
- Explain why `ClassName@hash` is weak in logs and test failures
- Distinguish between value objects that deserve a documented format and rich domain objects that only need a concise summary

### 2. Design a Useful toString Strategy

Provide two common patterns:

```java
@Override
public String toString() {
    return "%03d-%03d-%04d".formatted(areaCode, prefix, lineNum);
}
```

```java
@Override
public String toString() {
    return "[Potion #%d: type=%s, smell=%s, look=%s]"
            .formatted(id, type, smell, look);
}
```

- Use a stable documented format for value objects when round-trip parsing matters
- Use a concise summary for larger domain objects
- Keep programmatic accessors even when the string form is readable

### 3. Generate Verification Tests

- Test that `toString()` exposes the fields humans actually care about
- Test that value objects preserve a documented format
- Test round-trip parsing when the format is intentionally specified
- Test that diagnostic output in collections becomes readable

## How to Use

1. Decide whether the class is a value object or a richer domain object.
2. List the state a human needs in logs or failure messages.
3. Choose whether the string format is specified and stable, or intentionally left unspecified.
4. Implement `toString()` around that decision.
5. Keep accessors for the same state instead of forcing callers to parse the string.
6. Add tests that prove the output is useful in real diagnostics.

## Key Concepts

### Why Override toString

`println`, string concatenation, assertions, log frameworks, and debuggers all end up calling `toString()`.
If the class inherits `Object.toString()`, the output usually exposes only class name and identity hash, which helps humans far less than the object's actual meaning.

### When to Specify the Format

Specify the format when the object has a natural external representation, such as a phone number.
Leave the exact format unspecified when the output is mainly a debugging summary and may evolve later.

### Keep Accessors Even With a Good toString

A readable string is for humans.
Fields still need programmatic access through accessors or other APIs so callers do not depend on parsing the string representation.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of Java object methods
- JUnit 5 for regression tests

## Important Notes

- Override `toString()` in instantiable classes unless a superclass already provides a useful representation
- Do not force callers to parse `toString()` for data they need programmatically
- Document the format if callers may rely on it
- Auto-generated `toString()` is usually better than `Object.toString()`, but still may not express the domain's meaning well enough

## Best Practices

- Prefer concise, information-dense output over decorative prose
- Include the fields that explain the object's meaning to humans
- For value objects, consider a static parser or factory if the string format is specified
- Verify the real log or collection output, not only the isolated `toString()` method

## Success Criteria

- Default `Object.toString()` output is replaced where it hurts diagnostics
- Value objects with documented formats support stable human-readable output
- Tests prove the string form is useful in logs, collections, or round-trip parsing
- The implementation improves readability without replacing the actual object API
