---
name: avoid-finalizer-cleaner
description: Demonstrate why Finalizer and Cleaner mechanisms should be avoided, and show correct resource management practices using AutoCloseable
version: 1.0.0
author: AI Assistant
categories:
  - java
  - best-practices
  - resource-management
tags:
  - finalizer
  - cleaner
  - autocloseable
  - try-with-resources
  - effective-java
  - memory-management
---

# Avoid Finalizer and Cleaner

## Purpose

This Skill demonstrates the problems with Java's Finalizer and Cleaner mechanisms, and teaches the correct approach to resource management using `AutoCloseable` and `try-with-resources`.

Based on **Effective Java Item 8**: "Avoid finalizers and cleaners"

### When to Use

Use this Skill when you need to:
- Understand why `finalize()` is deprecated and dangerous
- Learn why `Cleaner` is unreliable despite being safer
- Implement proper resource cleanup with `AutoCloseable`
- Design classes that manage native resources

## Core Capabilities

### Demonstrates Anti-Patterns
- **Finalizer problems**: Unpredictable execution, performance degradation, security vulnerabilities
- **Cleaner misuse**: Still unpredictable, performance cost, not a replacement for explicit cleanup

### Shows Correct Patterns
- **AutoCloseable implementation**: Predictable, performant, safe
- **try-with-resources**: Automatic resource management
- **Cleaner as safety net**: Legitimate use case for Cleaner

### Educational Components
- Comparative examples showing wrong vs. right approaches
- Performance metrics and trade-offs
- Real-world scenarios and best practices

## How to Use

### Running the Demo

```bash
cd sample/finalizer-cleaner-demo
./gradlew clean build
java --enable-preview -cp build/classes/java/main com.iokays.cleaner.demo.CleanerDemo
```

### Examining the Code

#### Wrong: Using Finalizer

```java
@Deprecated(since = "9")
public class FinalizerDemo {
    @Override
    protected void finalize() throws Throwable {
        try {
            // Cleanup logic
        } finally {
            super.finalize();
        }
    }
}
```

**Problems**:
- ❌ No guarantee of execution
- ❌ No guarantee of timely execution
- ❌ Uncaught exceptions are silently ignored
- ❌ Severe performance penalty (50x slower)
- ❌ Security vulnerability (finalizer attacks)

#### Correct: Using AutoCloseable

```java
public class Resource implements AutoCloseable {
    private boolean closed = false;
    
    @Override
    public void close() {
        if (closed) return;  // Idempotent
        closed = true;
        // Cleanup logic
    }
    
    public void doSomething() {
        if (closed) {
            throw new IllegalStateException("Resource is closed");
        }
        // Business logic
    }
}

// Usage with try-with-resources
try (Resource r = new Resource()) {
    r.doSomething();
}  // Automatically closed
```

**Advantages**:
- ✅ Predictable execution time
- ✅ Best performance (no overhead)
- ✅ Clear exception handling
- ✅ Idempotent close method

#### Cleaner as Safety Net (Legitimate Use)

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    // State must NOT reference Room!
    private static class State implements Runnable {
        int numJunkPiles;
        
        @Override
        public void run() {
            // Cleanup logic
            numJunkPiles = 0;
        }
    }
    
    private final State state;
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
        this.state = new State(numJunkPiles);
        this.cleanable = cleaner.register(this, state);
    }
    
    @Override
    public void close() {
        cleanable.clean();  // Explicit cleanup
    }
}
```

**Key Points**:
- Main cleanup via `close()`, not Cleaner
- Cleaner is only a safety net
- `State` class must be static (no reference to Room)
- Don't rely on Cleaner for critical resources

## Key Concepts

### Finalizer Mechanism (Deprecated in Java 9)

**Definition**: A special method `finalize()` called by garbage collector before reclaiming an object's memory.

**Problems**:

1. **Unpredictable Execution**
   - No guarantee when (or if) `finalize()` runs
   - May never run before program exits
   - Time between object becoming unreachable and finalization is arbitrary

2. **Performance Penalty**
   - Creating/destroying objects with finalizers is ~50x slower
   - Hinders garbage collection efficiency
   - Memory pressure from finalization queue

3. **Security Vulnerabilities**
   - Finalizer attacks can resurrect objects from constructors
   - Malicious subclasses can exploit partial construction
   - Final classes are immune (no subclasses)

4. **Silent Exception Handling**
   - Uncaught exceptions in `finalize()` are ignored
   - No stack trace, no warning
   - Object remains in corrupt state

### Cleaner Mechanism (Java 9+)

**Definition**: A replacement for finalizers using a background thread and lambda-based cleanup actions.

**Improvements over Finalizer**:
- Better control over cleanup thread
- No silent exception swallowing
- Cleaner API

**Remaining Problems**:
- Still unpredictable execution timing
- Performance cost (5x slower than explicit cleanup)
- No guarantee of execution before JVM exit

### AutoCloseable Interface

**Definition**: Interface for resources that must be closed explicitly.

**Best Practice**:
```java
public class MyResource implements AutoCloseable {
    private boolean closed = false;
    
    public void doSomething() {
        if (closed) throw new IllegalStateException("Closed");
        // Business logic
    }
    
    @Override
    public void close() {
        if (closed) return;  // Idempotent
        closed = true;
        // Release resources
    }
}
```

**Usage with try-with-resources**:
```java
try (MyResource r = new MyResource()) {
    r.doSomething();
}  // r.close() called automatically
```

### Safety Net Pattern

When Cleaner is legitimately used as a backup:

```java
public class FileHandle implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    private static class State implements Runnable {
        private final String filePath;
        
        State(String filePath) {
            this.filePath = filePath;
        }
        
        @Override
        public void run() {
            System.out.println("Safety net: closing " + filePath);
            // Attempt to close file
        }
    }
    
    private final Cleaner.Cleanable cleanable;
    private boolean closed = false;
    
    public FileHandle(String path) {
        this.cleanable = cleaner.register(this, new State(path));
    }
    
    @Override
    public void close() {
        if (closed) return;
        closed = true;
        // Close file properly
        cleanable.clean();  // Unregister from cleaner
    }
}
```

## Prerequisites

### Required Knowledge
- Java basics (classes, interfaces, exceptions)
- Understanding of garbage collection
- Try-catch blocks and exception handling

### Required Environment
- Java 25+ (for preview features)
- Gradle 9.2.1+
- JUnit 5 (for tests)

### Running the Project

```bash
# Build the project
cd sample/finalizer-cleaner-demo
./gradlew clean build

# Run the demo
java --enable-preview -cp build/classes/java/main \
  com.iokays.cleaner.demo.CleanerDemo

# Run tests
./gradlew test
```

## Important Notes

### When NOT to Use Finalizer/Cleaner

❌ **Never for time-sensitive operations**
- Closing file descriptors
- Releasing locks
- Releasing database connections

❌ **Never for persistent state updates**
- Updating configuration files
- Writing to databases
- Releasing distributed locks

❌ **Never as primary cleanup mechanism**
- Always provide explicit `close()` method
- Cleaner should only be a safety net

### When Cleaner is Acceptable

✅ **As a safety net**
- Client forgot to call `close()`
- Better late cleanup than never
- Document that Cleaner is not primary method

✅ **For native peers**
- Non-Java resources
- No critical resources
- Performance is acceptable

### Critical Implementation Rules

**For AutoCloseable**:
1. Track closed state with a boolean field
2. Make `close()` method idempotent
3. Throw `IllegalStateException` if used after close
4. Document resource lifecycle

**For Cleaner as Safety Net**:
1. State class MUST be static (no reference to outer class)
2. Never use lambdas (they capture outer instance)
3. Unregister from Cleaner in `close()` method
4. Don't rely on Cleaner for critical resources

### Performance Comparison

| Approach | Relative Performance | Predictability | Safety |
|----------|---------------------|----------------|--------|
| Finalizer | 50x slower | ❌ None | ❌ Dangerous |
| Cleaner | 5x slower | ❌ Low | ⚠️ Safe but unreliable |
| AutoCloseable | Baseline | ✅ High | ✅ Safe and reliable |

## Best Practices

### Resource Management Checklist

- [ ] Implement `AutoCloseable` interface
- [ ] Provide `close()` method
- [ ] Track closed state
- [ ] Make `close()` idempotent
- [ ] Document resource lifecycle
- [ ] Use `try-with-resources` in client code
- [ ] (Optional) Add Cleaner as safety net

### Code Review Guidelines

**Reject code that**:
- Uses `finalize()` method
- Relies on Cleaner for primary cleanup
- Cleaner State class is non-static inner class
- Uses lambda for Cleaner action

**Accept code that**:
- Implements `AutoCloseable`
- Uses `try-with-resources`
- Documents resource lifecycle
- (Optional) Uses Cleaner as safety net

### Common Mistakes

#### 1. Non-static State Class

```java
// ❌ WRONG - captures outer instance
public class BadRoom {
    private static class State implements Runnable {
        @Override
        public void run() {
            // Can access BadRoom.this - prevents GC!
        }
    }
}

// ✅ CORRECT - static nested class
public class GoodRoom {
    private static class State implements Runnable {
        // No reference to GoodRoom
        @Override
        public void run() {
            // Safe cleanup
        }
    }
}
```

#### 2. Lambda as Cleaner Action

```java
// ❌ WRONG - lambda captures 'this'
cleaner.register(this, () -> {
    // Can access 'this' - prevents GC!
});

// ✅ CORRECT - use static nested class
private static class State implements Runnable {
    @Override
    public void run() {
        // No capture of outer instance
    }
}
```

#### 3. Forgetting to Unregister

```java
// ❌ WRONG - Cleaner runs even after close
@Override
public void close() {
    // Cleanup resources
    // But don't call cleanable.clean()!
}

// ✅ CORRECT - unregister from Cleaner
@Override
public void close() {
    cleanable.clean();  // Runs cleanup and unregisters
}
```

## Success Criteria

A successful implementation should:

- [ ] All tests pass (`./gradlew clean build`)
- [ ] No compiler warnings (except deprecation warnings for finalize)
- [ ] Resources closed within try-with-resources blocks
- [ ] Cleaner State classes are static
- [ ] close() methods are idempotent
- [ ] IllegalStateException thrown after close
- [ ] Documentation explains resource lifecycle

### Verification Commands

```bash
# Run all tests
./gradlew test

# Run demo
java --enable-preview -cp build/classes/java/main \
  com.iokays.cleaner.demo.CleanerDemo

# Check for warnings
./gradlew clean build -Xlint:all
```

## References

- **Effective Java, 3rd Edition** - Item 8: Avoid finalizers and cleaners
- **Java 9 Documentation** - `java.lang.ref.Cleaner`
- **Java Language Specification** - Section 12.6 (Finalization)
- **Item 9** - Prefer try-with-resources to try-finally
