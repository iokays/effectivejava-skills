---
name: implement-comparable-carefully
description: Guide for implementing Comparable correctly in Java value classes. Use this skill when a class has a natural order, needs to work in TreeSet or TreeMap, or risks compareTo being inconsistent with equals.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - ordering
tags:
  - comparable
  - compareto
  - treeset
  - treemap
---

# implement-comparable-carefully

## Purpose

Explain when Java classes should implement `Comparable`, how to write a correct `compareTo`, and what goes wrong when natural ordering disagrees with `equals`.
Use it for value classes, ordered collections, and teaching material about ordering contracts.

## Core Capabilities

### 1. Identify Classes That Deserve a Natural Order

- Detect value classes with an obvious business order
- Explain why ordered collections and sorting APIs benefit from `Comparable`
- Distinguish natural ordering from one-off external comparators

### 2. Generate Correct compareTo Implementations

Provide a standard pattern:

```java
private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.prefix)
                .thenComparingInt(pn -> pn.lineNum);

@Override
public int compareTo(PhoneNumber other) {
    return COMPARATOR.compare(this, Objects.requireNonNull(other, "other"));
}
```

- Compare the most important field first
- Continue field by field until the order is decided
- Reject null naturally
- Avoid subtraction-based comparisons

### 3. Expose Ordering Pitfalls

- Show how `TreeSet` and `TreeMap` depend on `compareTo`
- Explain why `compareTo == 0` acts like equality inside sorted collections
- Demonstrate mismatch cases similar to `BigDecimal`, where `equals` and `compareTo` disagree

## How to Use

1. Decide whether the class has a stable natural order.
2. List the fields that should determine that order.
3. Implement `compareTo` in significance order.
4. Check whether `compareTo == 0` aligns with `equals`.
5. Add tests around sorting and sorted collections.

## Key Concepts

### Why Comparable Matters

Implementing `Comparable` lets instances participate naturally in sorting, `TreeSet`, `TreeMap`, binary search, and min/max operations.
Without it, every caller needs an external comparator or loses ordered-collection support.

### Consistency With equals

The contract strongly recommends that `compareTo == 0` match `equals`.
If they disagree, the class may still work, but sorted collections can behave differently from hash-based collections.

### Avoid Difference-Based Comparisons

Never implement `compareTo` by subtracting numeric fields.
That style risks overflow and produces brittle code.
Use wrapper `compare` methods or comparator construction APIs instead.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of value objects and collection semantics
- JUnit 5 for regression tests

## Important Notes

- Prefer `Comparator.comparing...` or wrapper `compare` methods over `<`, `>`, and subtraction tricks
- A class with no obvious natural order usually should not implement `Comparable`
- If `compareTo` is inconsistent with `equals`, document the fact clearly
- Ordered collections use `compareTo`, not `equals`, to decide uniqueness

## Best Practices

- Start comparison with the most important field
- Keep `compareTo` and `equals` close together in the class
- Add one sorted-collection example to prove the natural ordering is useful
- Add one mismatch example to teach the cost of inconsistency

## Success Criteria

- Sorting APIs work directly on the class
- `TreeSet` / `TreeMap` behavior is explained and verified
- Tests prove the intended natural order
- Any `compareTo` vs `equals` mismatch is explicit, not accidental
