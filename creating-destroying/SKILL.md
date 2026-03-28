---
name: creating-destroying
description: Use when Codex needs to design, review, explain, or refactor Java code around object creation, construction APIs, dependency wiring, object reuse, memory retention, resource cleanup, and lifecycle management. Apply this skill for Effective Java's "Creating and Destroying Objects" topics such as static factory methods vs constructors, builders, singletons, noninstantiable utility classes, dependency injection, avoiding unnecessary objects, eliminating obsolete references, avoiding finalizers or cleaners, and preferring try-with-resources.
---

# Creating and Destroying

## Overview

Use this skill to turn Effective Java's "Creating and Destroying Objects" guidance into concrete coding decisions.
Inspect the actual Java API, class design, and call sites first, then apply the smallest change that materially improves correctness, readability, testability, or lifecycle safety.

## Workflow

1. Identify the dominant problem before changing code.
- construction API design
- too many parameters
- singleton or utility-class design
- hardwired dependencies
- unnecessary allocation or boxing
- memory retention or cache/listener leaks
- cleanup and resource lifetime

2. Read the class and its call sites.
- Decide whether the problem is in the public API, the implementation, or both.
- Preserve existing behavior unless the user explicitly wants an API break.

3. Apply the relevant Effective Java item guidance.
- Prefer focused refactors over broad stylistic rewrites.
- Add or update tests when behavior changes or bug risk is meaningful.

4. Explain the result in Effective Java terms.
- State which item or items drove the change.
- Mention the tradeoff, not just the rule.

## Item Guide

### Item 1: Prefer static factory methods when they improve the API

Use a static factory instead of a public constructor when one or more of these are true:

- the returned object needs a meaningful name
- repeated calls can reuse cached instances
- the method should return an interface or subtype
- implementation classes should stay hidden
- instance creation may vary by input or release

Common names:
- `from`
- `of`
- `valueOf`
- `getInstance`
- `newInstance`
- `getType`
- `newType`

Keep constructors when direct instantiation is clearer and subtype flexibility or caching is unnecessary.

Watch for drawbacks:
- classes without accessible constructors cannot be subclassed
- static factories are harder to discover than constructors

### Item 2: Use a builder for many parameters

Use a builder when constructors or factories would otherwise take many parameters, especially when:

- several parameters are optional
- many parameters share the same type
- readability is poor at the call site
- the built object should be immutable

Expect these traits:
- required parameters in the builder constructor or entry factory
- optional parameters with fluent setter-like methods
- validation in builder methods and `build()`
- immutable target object

For class hierarchies, prefer hierarchical builders with recursive generics when the hierarchy is already established.

Do not introduce a builder when the type has only a few obvious parameters and is unlikely to grow.

### Item 3: Implement singletons deliberately

If a type must have exactly one instance:

- prefer a single-element `enum` when possible
- otherwise use a `public static final` instance or a static factory plus private constructor

Check these edge cases:
- testability: consumers often need an interface or injectable dependency
- serialization: preserve singleton semantics explicitly if not using `enum`
- reflection: private constructors alone are not absolute protection

Prefer not to introduce a singleton when dependency injection would give cleaner tests and configuration.

### Item 4: Make utility classes noninstantiable

For classes that only contain static members:

- add a private constructor
- throw `AssertionError` inside it

Do not make the class `abstract` just to block instantiation. That still allows subclassing and communicates the wrong design intent.

### Item 5: Prefer dependency injection to hardwired resources

When a class depends on dictionaries, repositories, clocks, clients, parsers, configuration, or similar resources:

- inject the dependency through a constructor, static factory, or builder
- inject a `Supplier<? extends T>` when the code needs a factory

Avoid these designs unless the dependency is truly universal and fixed:
- static utility holders of mutable resources
- singleton services with hardcoded collaborators
- constructing dependencies directly inside business logic

Prioritize:
- testability
- configurability
- immutability of the receiving class

### Item 6: Avoid unnecessary objects

Look for avoidable allocation patterns such as:

- `new String("...")`
- boxed primitives in hot loops
- repeated regex compilation
- constructors on immutable value types when `valueOf` or another factory exists

Prefer:
- string literals
- cached immutable instances
- precompiled `Pattern`
- primitives over boxed primitives in computation-heavy paths

Do not add manual object pools for lightweight objects. They usually hurt clarity and often hurt performance.

### Item 7: Eliminate obsolete references

Suspect memory retention when a class manages its own storage or registrations:

- arrays used as manual storage pools
- custom stacks, buffers, and freelists
- caches
- listeners and callbacks

Typical fixes:
- null out slots that leave the active region
- remove stale cache entries
- use weak references only when lifetime semantics truly match
- deregister callbacks or store them weakly when appropriate

Be especially alert when the GC cannot infer which references are obsolete because the class is managing memory itself.

### Item 8: Avoid finalizers and cleaners

Do not rely on finalizers or cleaners for normal resource management.

Prefer:
- `AutoCloseable`
- explicit `close()`
- `try-with-resources`

Treat cleaners only as a best-effort safety net or for noncritical native-peer cleanup.

If you touch cleaner-based code:
- ensure cleanup state does not retain the outer object
- keep cleaner logic minimal
- never depend on prompt or guaranteed execution

### Item 9: Prefer try-with-resources

For closeable resources:

- replace `try-finally` with `try-with-resources`
- use multiple resources in the same `try (...)` header when appropriate
- keep `catch` handling outside the resource declaration when it improves clarity

This improves:
- correctness
- readability
- exception diagnostics through suppressed exceptions

## Review Heuristics

When reviewing Java code with this skill, prioritize findings in this order:

1. correctness and lifecycle bugs
- leaking file handles, sockets, DB connections, threads, or listeners
- retaining obsolete references
- unsafe singleton or cleanup logic

2. API design problems
- ambiguous constructors
- too many positional parameters
- hardwired dependencies

3. performance issues with clear payoff
- needless allocation in hot code
- accidental autoboxing
- repeated creation of expensive immutable helpers

## References

Read [references/items-1-9.md](./references/items-1-9.md) when you need a compact checklist for each item before reviewing or refactoring code.
