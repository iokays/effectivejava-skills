---
name: try-with-resources
description: Java resource management best practice using try-with-resources to replace try-finally
version: 1.0.0
author: AI Assistant
categories:
  - java
  - best-practices
  - resource-management
tags:
  - java
  - try-with-resources
  - autocloseable
  - exception-handling
  - resource-management
---

# try-with-resources Skill

## Purpose

This skill provides comprehensive guidance and code examples for implementing proper resource management in Java using the try-with-resources statement, which was introduced in Java 7 as a superior alternative to the traditional try-finally pattern.

### When to Use This Skill

- Implementing resource management in Java applications
- Refactoring legacy try-finally code to modern patterns
- Creating custom AutoCloseable implementations
- Understanding exception suppression mechanisms
- Managing multiple resources simultaneously

## Core Capabilities

- **Resource Auto-Management**: Automatically closes resources when they are no longer needed
- **Exception Safety**: Preserves the primary exception while suppressing secondary exceptions from close operations
- **Code Simplification**: Reduces boilerplate code compared to try-finally
- **Multiple Resource Support**: Manages multiple resources in a single try statement
- **Custom Resource Integration**: Supports any AutoCloseable implementation

## Key Concepts

### AutoCloseable Interface

The `AutoCloseable` interface is the foundation of try-with-resources:

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

Any class that needs automatic resource management should implement this interface.

### Suppressed Exceptions

When both the try block and close() method throw exceptions:
- **Primary exception**: The exception from the try block is preserved
- **Suppressed exception**: The exception from close() is suppressed and attached to the primary exception
- **Access**: Use `Throwable.getSuppressed()` to retrieve suppressed exceptions

### Resource Closing Order

Resources are closed in the **reverse order** of their declaration:
```java
try (Resource1 r1 = new Resource1();
     Resource2 r2 = new Resource2()) {
    // Resources used here
} // r2.close() called first, then r1.close()
```

## How to Use

### Basic Usage

#### Single Resource Management

```java
// Modern approach (Java 7+)
public static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

#### Multiple Resources

```java
// Clean and readable
public static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
}
```

### With Catch Clause

```java
// Graceful error handling
public static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

### Custom AutoCloseable Implementation

```java
public class CustomResource implements AutoCloseable {
    private final String name;
    private boolean closed = false;
    
    public CustomResource(String name) {
        this.name = name;
        System.out.println("[" + name + "] Resource opened");
    }
    
    public void doSomething() {
        if (closed) {
            throw new IllegalStateException("Resource is already closed");
        }
        System.out.println("[" + name + "] Doing something...");
    }
    
    @Override
    public void close() throws Exception {
        if (!closed) {
            closed = true;
            System.out.println("[" + name + "] Resource closed");
        }
    }
}

// Usage
try (CustomResource resource = new CustomResource("MyResource")) {
    resource.doSomething();
}
```

### Exception Suppression Demonstration

```java
public class ExceptionalResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new Exception("Close exception");
    }
}

// Demonstration
try (ExceptionalResource resource = new ExceptionalResource()) {
    throw new Exception("Primary exception");
} catch (Exception primary) {
    System.out.println("Primary: " + primary.getMessage());
    // Primary: Primary exception
    
    for (Throwable suppressed : primary.getSuppressed()) {
        System.out.println("Suppressed: " + suppressed.getMessage());
        // Suppressed: Close exception
    }
}
```

## Best Practices

### 1. Always Use try-with-resources for AutoCloseable

**❌ Avoid:**
```java
InputStream in = new FileInputStream(path);
try {
    // use resource
} finally {
    in.close();
}
```

**✅ Prefer:**
```java
try (InputStream in = new FileInputStream(path)) {
    // use resource
}
```

### 2. Declare Resources in Order of Dependency

```java
// Good: Connection created first, then Statement, then ResultSet
try (Connection conn = DriverManager.getConnection(url);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sql)) {
    // Process results
}
```

### 3. Make close() Idempotent

```java
@Override
public void close() throws Exception {
    if (!closed) {
        closed = true;
        // Perform cleanup
    }
}
```

### 4. Handle Exceptions Appropriately

```java
try (Resource resource = new Resource()) {
    resource.doSomething();
} catch (SpecificException e) {
    // Handle specific exception
    logger.error("Operation failed", e);
} catch (Exception e) {
    // Handle general exceptions
    logger.error("Unexpected error", e);
}
```

### 5. Document Close Behavior

```java
/**
 * Closes this resource. This method is idempotent and can be called
 * multiple times without side effects.
 * 
 * @throws IOException if an I/O error occurs during close
 */
@Override
public void close() throws IOException {
    // Implementation
}
```

## Prerequisites

- **Java Version**: Java 7 or later (Java 25 recommended)
- **Knowledge**: Basic understanding of Java exception handling
- **IDE Support**: Modern IDEs provide automatic resource management suggestions
- **Testing Framework**: JUnit 5 for testing resource management code

## Important Notes

### Resource Lifecycle

1. **Declaration**: Resources must be declared within the try parentheses
2. **Scope**: Resources are only accessible within the try block
3. **Closing**: Resources are automatically closed when the try block exits (normally or exceptionally)

### Exception Handling

- Primary exceptions from the try block take precedence
- Exceptions from close() are suppressed but not lost
- Both exceptions are logged in the stack trace
- Use `getSuppressed()` to access suppressed exceptions programmatically

### Migration from try-finally

When refactoring legacy code:
1. Identify all try-finally blocks that manage resources
2. Verify the resource implements AutoCloseable (most standard resources do)
3. Convert to try-with-resources syntax
4. Test to ensure behavior is preserved
5. Remove any redundant null checks or manual close calls

### Common Pitfalls

❌ **Declaring resources outside the try statement:**
```java
Resource resource = new Resource();
try (resource) { // Error: must declare within try
    // ...
}
```

✅ **Correct approach:**
```java
try (Resource resource = new Resource()) {
    // ...
}
```

### Performance Considerations

- Minimal overhead compared to manual close() calls
- JVM optimizes the generated bytecode
- The benefit of cleaner code and better exception handling outweighs any minor performance cost

## Success Criteria

A resource management implementation using this skill should meet:

- ✅ All resources implement AutoCloseable
- ✅ Resources are declared within try-with-resources statements
- ✅ Multiple resources are managed in a single try block when appropriate
- ✅ No manual close() calls in finally blocks
- ✅ Exception handling preserves primary exceptions
- ✅ Code is readable and maintainable
- ✅ All tests pass including resource cleanup verification
- ✅ No resource leaks detected by static analysis tools

## Example Project Structure

```
try-with-resources-demo/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/iokays/trywithresources/
│   │           ├── classic/              # Traditional try-finally examples
│   │           │   └── ClassicResourceManagement.java
│   │           ├── modern/               # Modern try-with-resources examples
│   │           │   └── ModernResourceManagement.java
│   │           ├── custom/               # Custom AutoCloseable implementations
│   │           │   └── CustomResource.java
│   │           └── ResourceManagementDemo.java
│   └── test/
│       └── java/
│           └── com/iokays/trywithresources/
│               ├── classic/
│               │   └── ClassicResourceManagementTest.java
│               ├── modern/
│               │   └── ModernResourceManagementTest.java
│               └── custom/
│                   └── CustomResourceTest.java
├── build.gradle.kts
└── gradlew
```

## Further Reading

- Java Language Specification: Section 14.20.3 (try-with-resources)
- Effective Java, Third Edition, Item 9: "Prefer try-with-resources to try-finally"
- Oracle Java Documentation: [The try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
