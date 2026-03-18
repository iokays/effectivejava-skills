---
name: design-or-prohibit-inheritance
description: Guide for Java classes that must either be explicitly designed and documented for inheritance or must forbid subclassing. Use this skill when a superclass calls overridable methods, lacks subclassing contracts, or should be made final or factory-based instead.
version: 1.0.0
author: AI Assistant
categories:
  - java
  - effective-java
  - design
tags:
  - inheritance
  - final
  - template-method
  - api-design
---

# design-or-prohibit-inheritance

## Purpose

Help redesign Java classes so they either support inheritance deliberately or forbid it explicitly.
Use it when a class was casually left non-final, when constructors invoke overridable methods, or when subclass authors would otherwise depend on undocumented superclass behavior.

## Core Capabilities

### 1. Detect Unsafe Inheritance Points

- Find constructors that directly or indirectly call overridable methods
- Identify concrete classes that are subclassable without documented extension contracts
- Spot APIs whose behavior would change if a subclass overrides one method and accidentally affects another

### 2. Design Safe Hook-Based Inheritance

Provide the standard direction:

```java
public abstract class SafeTemplate {
    protected SafeTemplate() {
    }

    public final String preview() {
        return render();
    }

    protected abstract String render();
}
```

- Do not call overridable methods from constructors, `clone`, or deserialization hooks
- Expose only a small number of carefully chosen protected hooks
- Keep externally visible orchestration methods `final` when they define critical sequencing
- Document subclass obligations and self-use clearly

### 3. Prohibit Inheritance When It Is Not Needed

- Make the class `final`, or
- Hide constructors behind private/package-private access plus static factories
- Keep implementation details from leaking into a fragile subclass contract

## How to Use

1. Decide whether third-party subclassing is a real requirement.
2. If it is not, prohibit inheritance immediately.
3. If it is required, define a minimal hook surface and document it.
4. Ensure constructors and similar lifecycle methods never invoke overridable methods.
5. Add tests that prove subclass state is not observed before initialization.

## Key Concepts

### Why Casual Subclassing Is Dangerous

A class that merely happens to be non-final is not automatically safe to extend.
Without explicit design and documentation, subclasses may rely on internal sequencing the superclass never promised to keep stable.

### Why Constructor Self-Use Is Especially Dangerous

Superclass constructors run before subclass constructors.
If a superclass constructor calls an overridable method, the subclass override can observe half-initialized state.

### Why Final Is Often the Right Answer

If a class does not truly need subclassing, forbidding inheritance is usually cheaper and safer than inventing a fragile extension story.

## Prerequisites

- Java 25 or a compatible modern JDK
- Basic understanding of inheritance, constructors, and method overriding
- JUnit 5 for regression tests

## Important Notes

- Constructors must not call overridable methods
- `clone` and deserialization hooks have similar risks to constructors
- Supporting inheritance is a long-term API commitment, not a default convenience
- Protected hooks should be rare, intentional, and documented

## Best Practices

- Default to `final` for concrete classes unless subclassing is truly needed
- Use `final` template methods plus protected hooks when safe extension is required
- Keep extension points narrow and explicit
- Test with real subclasses before claiming a class is inheritance-safe

## Success Criteria

- No constructor or similar lifecycle method calls an overridable method
- Classes that do not need extension are explicitly non-subclassable
- Classes that support inheritance expose a minimal documented hook surface
- Tests prove subclass state is not observed before initialization
