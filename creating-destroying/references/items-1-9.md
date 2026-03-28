# Effective Java Chapter 2 Items 1-9 Checklist

Use this reference as a quick decision sheet while reviewing or changing Java code.

## Item 1: Static Factory Methods

- Prefer named factories when constructors are ambiguous or semantically weak.
- Prefer factories when instance reuse, subtype return, or hidden implementations matter.
- Keep constructors when direct construction is the clearest API.
- Common names: `from`, `of`, `valueOf`, `getInstance`, `newInstance`, `getType`, `newType`.

## Item 2: Builder

- Use a builder when a class has many parameters, many optional parameters, or many same-typed parameters.
- Put required parameters up front in the builder entry point.
- Validate early for single-field constraints and at `build()` for cross-field invariants.
- Prefer immutable built objects.

## Item 3: Singleton

- Prefer enum singletons unless inheritance rules block it.
- Otherwise use private constructor plus static instance access.
- Consider serialization, reflection, and testability.
- Do not default to singleton when dependency injection is the better seam.

## Item 4: Noninstantiable Utility Classes

- Add a private constructor.
- Throw `AssertionError` defensively inside the constructor.
- Do not use `abstract` as a substitute.

## Item 5: Dependency Injection

- Inject resources instead of hardcoding them.
- Inject factories with `Supplier<? extends T>` when lazy or repeated creation is needed.
- Prefer constructor injection for required dependencies.
- Avoid static utilities and singletons for stateful or environment-specific collaborators.

## Item 6: Avoid Unnecessary Objects

- Reuse immutable objects freely.
- Prefer factories like `valueOf` over constructors on boxed or immutable values.
- Cache expensive immutable helpers such as compiled regex `Pattern`.
- Prefer primitives to boxed primitives in numeric loops and hot paths.
- Avoid lightweight object pools.

## Item 7: Obsolete References

- Manually clear references when your class manages its own memory.
- Audit caches, listeners, and callback registries for retention.
- Use `WeakHashMap` only when entry lifetime should track key reachability.
- Prefer letting variables fall out of scope over aggressive manual nulling everywhere.

## Item 8: Finalizers and Cleaners

- Do not use them for ordinary cleanup.
- Use `AutoCloseable` and explicit `close()`.
- Cleaner safety nets are optional and best-effort only.
- Never depend on prompt execution or execution at all.

## Item 9: Try-With-Resources

- Prefer `try-with-resources` for anything closeable.
- Put multiple closeable resources in the same header when possible.
- Rely on suppressed exceptions for better diagnostics.
- Reserve `try-finally` for cases that are not resource-lifetime problems.
