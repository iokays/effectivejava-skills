---
name: equals-hashcode-contract
description: Guide for implementing equals and hashCode together in Java value objects. Use this skill when a class overrides equals, needs to work correctly in HashMap or HashSet, or shows lookup bugs caused by mismatched equality and hash codes.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - collections
tags:
  - equals
  - hashcode
  - hashmap
  - hashset
---

# equals-hashcode-contract

## Purpose

Explain and apply the rule that overriding `equals` requires overriding `hashCode`.
Use it to diagnose collection bugs, implement value objects, and generate tests that prove the contract is respected.

## Core Capabilities

### 1. Detect Contract Violations

- Find classes that override `equals` but inherit `Object.hashCode`
- Explain why `HashMap#get` and `HashSet#contains` fail for logically equal values
- Highlight which fields must participate in both methods

### 2. Generate Correct Implementations

Provide a standard recipe:

```java
@Override
public boolean equals(Object other) {
    if (other == this) {
        return true;
    }
    if (!(other instanceof PhoneNumber that)) {
        return false;
    }
    return areaCode == that.areaCode
        && prefix == that.prefix
        && lineNum == that.lineNum;
}

@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

### 3. Produce Verification Tests

- Test equal objects share the same hash code
- Test `HashMap#get` works with a fresh but equal key
- Test `HashSet#contains` works with a fresh but equal value
- Test invalid constructor inputs still fail fast

## How to Use

1. Identify whether the class is a value object.
2. List the fields that define logical equality.
3. Implement `equals` using only those fields.
4. Implement `hashCode` with the same fields and stable deterministic logic.
5. Verify behavior with real `HashMap` and `HashSet` examples.

## Key Concepts

### Why Collections Break

`HashMap` and `HashSet` choose a bucket from `hashCode` before they evaluate `equals`.
If two equal objects return different hash codes, they land in different buckets and never meet.

### Good hashCode Characteristics

- Equal objects always return the same hash code
- Unequal objects are distributed reasonably well
- The implementation depends only on the fields used by `equals`
- The implementation stays stable while the object stays unchanged

## Prerequisites

- Java 25 or compatible modern JDK
- Basic understanding of `Object`, `HashMap`, and `HashSet`
- JUnit 5 for regression tests

## Important Notes

- Never include fields in `hashCode` that are excluded from `equals`
- Never keep the default `hashCode` after overriding `equals`
- Prefer immutable value objects for safer contract behavior
- `Objects.hash(...)` is acceptable for clarity, but the `31 * result + fieldHash` recipe is easier to reason about in teaching material

## Best Practices

- Keep `equals` and `hashCode` next to each other in the class
- Add one failing collection example before the fix and one passing example after the fix
- Use value-object examples such as phone numbers, coordinates, or identifiers
- Add tests that document the bug, not just the final happy path

## Success Criteria

- A logically equal fresh key retrieves the stored `HashMap` value
- A logically equal fresh value is found by `HashSet#contains`
- Tests prove equal objects share the same hash code
- Documentation explains the failure mode, not only the final code
