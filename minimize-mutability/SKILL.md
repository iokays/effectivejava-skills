---
name: minimize-mutability
description: Guide for redesigning Java classes toward immutability, functional update methods, and limited mutable companions. Use this skill when a value object exposes setters, drifts into invalid state, or should become easier to share safely across threads and collections.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - immutability
tags:
  - immutable
  - final
  - value-object
  - api-design
---

# minimize-mutability

## Purpose

Help redesign Java classes so that mutation is reduced to the minimum necessary.
Use it when a value object should stop exposing setters, preserve invariants more reliably, or move complex multi-step edits into a dedicated mutable companion.

## Core Capabilities

### 1. Identify Unnecessary Mutability

- Find public APIs that expose setters without strong business need
- Detect value objects that can drift into invalid state after construction
- Distinguish true mutable workflows from data that should be modeled as snapshots

### 2. Convert Mutable APIs Into Immutable Value Objects

Provide the standard direction:

```java
public final class QuotaSnapshot {
    private final int limit;
    private final int used;

    public QuotaSnapshot reserve(int units) {
        return new QuotaSnapshot(limit, used + units);
    }
}
```

- Remove mutators from the core value object
- Keep fields `private final`
- Make update methods return new instances instead of modifying the current one
- Validate invariants at construction time so every visible instance is always valid

### 3. Introduce a Mutable Companion Only When Needed

- Keep the main class immutable for sharing, reasoning, and collection usage
- Use a mutable companion for multi-step editing or batch construction
- Convert back to an immutable snapshot before handing results to callers

## How to Use

1. Decide whether the class is really a mutable workflow object or just a value object with accidental setters.
2. If it is a value object, remove mutators and make state `private final`.
3. Replace in-place updates with functional methods that return new instances.
4. If batch editing is genuinely needed, add a separate mutable companion class.
5. Add tests that prove invalid intermediate state is no longer visible from the immutable API.

## Key Concepts

### Why Immutability Simplifies Design

An immutable object has one visible state for its whole lifetime.
Once construction succeeds, clients no longer need to wonder which setter ran earlier or whether the object is temporarily half-updated.

### Why Minimal Mutability Improves Safety

Fewer mutation paths mean fewer invalid states, easier thread-safety, more stable equality/hash behavior, and simpler reasoning when objects are reused.

### Why a Mutable Companion Can Still Be Useful

Immutability is not a ban on all mutation everywhere.
When a sequence of edits is expensive or cumbersome, a separate mutable companion can absorb those edits locally, then produce one final immutable result.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of constructors, `final`, and value objects
- JUnit 5 for regression tests

## Important Notes

- Do not add setter pairs by default just because a class has fields
- Prefer immutable value objects for small and medium-sized domain data
- A mutable companion is a deliberate exception, not the default design
- Construction should establish all invariants up front

## Best Practices

- Default to `private final` fields
- Prefer functional update methods such as `plus`, `reserve`, `expand`, `withX`
- Make invalid state impossible to observe from the immutable API
- Keep mutable editing scoped to builders, editors, or companion classes

## Success Criteria

- The main API object exposes no unnecessary mutators
- All visible instances remain valid after construction
- Updates produce new objects instead of corrupting existing ones
- Tests prove the difference between the old mutable design and the new immutable one
