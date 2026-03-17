---
name: minimize-access-carefully
description: Guide for minimizing class and member accessibility in Java. Use this skill when designing APIs, deciding between public and package-private types, or preventing mutable state and implementation details from leaking into external callers.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - encapsulation
tags:
  - access-control
  - encapsulation
  - api-design
  - visibility
---

# minimize-access-carefully

## Purpose

Explain how to keep Java classes and members as inaccessible as possible while still meeting functional needs.
Use it for API design, package structure decisions, public constant exposure, and defensive handling of arrays or mutable state.

## Core Capabilities

### 1. Detect Overexposed APIs

- Find classes or members that are public without a real external need
- Flag public mutable fields and exposed arrays
- Highlight package-private or private opportunities for helper types and implementation details

### 2. Recommend Safer Visibility Choices

Provide the standard direction:

- Make top-level types package-private unless they must be part of the public API
- Make members `private` first, then relax only when a real caller requires it
- Prefer methods and views over public mutable fields
- Expose immutable constants only when they are truly part of the abstraction

### 3. Repair Mutable Exposure

- Replace public arrays with private arrays plus safe views or defensive copies
- Use `Collections.unmodifiableList(...)` for read-only exposure
- Return cloned arrays or copies when callers need independent data

## How to Use

1. Decide which types and members are truly part of the external API.
2. Push everything else down to the smallest workable access level.
3. Replace direct field exposure with methods when behavior or invariants matter.
4. Audit public constants to ensure they are immutable.
5. Add tests that prove callers cannot mutate internal state accidentally.

## Key Concepts

### Information Hiding

Good components hide implementation details and communicate through a small, stable API.
That separation makes code easier to change, optimize, test, and reuse without breaking callers.

### Visibility Escalation Is Expensive

Moving from `private` to package-private is local.
Moving to `protected` or `public` turns implementation choices into long-term API commitments.

### Arrays Need Special Care

A `public static final` array is still mutable through its elements.
If callers can reach the array reference, they can often change internal state even though the field itself is final.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of packages, classes, and fields
- JUnit 5 for regression tests

## Important Notes

- Public classes rarely should have public mutable fields
- Package-private is often the right default for implementation-only top-level classes
- `protected` is much more expensive than it looks because it expands the supported API surface
- Test convenience is not a valid reason to make production APIs more public than necessary

## Best Practices

- Start from `private` and loosen only when required
- Prefer immutable views or defensive copies for collections and arrays
- Keep implementation helpers out of the exported API whenever possible
- Treat every public member as a long-term compatibility promise

## Success Criteria

- Internal state is no longer directly mutable by callers
- Public API surface is intentionally small
- Arrays and other mutable structures are exposed safely, if at all
- Tests prove that callers cannot accidentally corrupt internal configuration
