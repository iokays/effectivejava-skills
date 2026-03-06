---
name: spring-ai-recursive-advisors
description: Demonstrates recursive advisor pattern in Spring AI for iterative LLM interactions
version: 1.0.0
author: AI Assistant
categories:
  - spring-ai
  - llm
  - advisors
tags:
  - spring-ai
  - recursive-advisors
  - chat-model
  - quality-check
  - tool-calling
  - structured-output
---

# Purpose

This skill provides a comprehensive demonstration of the Recursive Advisor pattern introduced in Spring AI 1.1. It shows how to break the traditional single-pass advisor model to enable iterative retry behaviors, such as quality checks, tool calling loops, and structured output validation.

Use this skill when you need to:
- Implement iterative LLM interactions with automatic retry logic
- Add quality checks that provide feedback and retry until conditions are met
- Handle multiple tool calls in sequence with full observability
- Validate structured outputs against Java record schemas
- Build custom recursive advisors for specific use cases

# Core Capabilities

This skill demonstrates three key patterns:

1. **ToolCallAdvisor**: Moves tool calling loop into the advisor chain, giving other advisors visibility into each tool call round-trip. Handles multiple sequential tool calls automatically.

2. **StructuredOutputValidationAdvisor**: Ensures LLM's JSON output matches a schema derived from a Java record. Retries with error details if validation fails.

3. **Custom QualityCheckAdvisor**: Evaluates LLM's answer length and retries with feedback if it falls below a configurable threshold. Demonstrates the pattern for building custom recursive advisors.

Additionally, this skill includes:
- Complete unit tests demonstrating advisor behavior
- Example configurations for each advisor type
- Best practices for preventing infinite retry loops
- Performance considerations and ordering strategies

# How to Use

## Step 1: Add Dependencies

Add Spring AI dependencies to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation(platform("org.springframework.ai:spring-ai-bom:1.1.2"))
    implementation("org.springframework.ai:spring-ai-starter-model-openai")
}
```

## Step 2: Create Custom Advisor

Implement the `CallAdvisor` interface:

```java
public class QualityCheckAdvisor implements CallAdvisor {

    private static final int MAX_RETRIES = 3;
    private static final int MIN_LENGTH_THRESHOLD = 200;

    @Override
    public ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain) {
        ChatClientResponse response = chain.nextCall(request);
        int attempts = 0;

        while (attempts < MAX_RETRIES && !isHighQuality(response)) {
            String feedback = "Your previous answer was incomplete. "
                + "Please provide a more thorough response.";

            var augmentedPrompt = request
                .prompt()
                .augmentUserMessage(
                    userMessage -> userMessage.mutate()
                        .text(userMessage.getText() + System.lineSeparator() + feedback)
                        .build()
                );

            var augmentedRequest = request
                .mutate()
                .prompt(augmentedPrompt)
                .build();

            response = chain
                .copy(this)
                .nextCall(augmentedRequest);

            attempts++;
        }

        return response;
    }

    private boolean isHighQuality(ChatClientResponse response) {
        String content = response
            .chatResponse()
            .getResult()
            .getOutput()
            .getText();

        return content != null && content.length() >= MIN_LENGTH_THRESHOLD;
    }
}
```

## Step 3: Configure ChatClient with Advisor

```java
var chatClient = ChatClient
    .builder(chatModel)
    .defaultAdvisors(new QualityCheckAdvisor())
    .build();
```

## Step 4: Use the ChatClient

```java
String result = chatClient
    .prompt()
    .user("Explain the SOLID principles in detail.")
    .call()
    .content();
```

The advisor will automatically:
1. Call the LLM with the original prompt
2. Check if the response meets the quality threshold
3. If not, add feedback and retry
4. Repeat up to 3 times until quality check passes

# Key Concepts

## Recursive Advisor Pattern

A regular advisor processes a request exactly once: request → chain → response. A recursive advisor can evaluate the response and loop back through the downstream part of the chain for another attempt.

**Two key properties:**
1. Only advisors downstream of the recursive advisor re-execute on each loop
2. Downstream advisors still observe every iteration

## Termination Conditions

Every recursive advisor must define a termination condition to prevent infinite loops:
- Maximum retry limit (e.g., `MAX_RETRIES = 3`)
- Explicit exit criteria (e.g., quality check passes)
- Both combined for safety

## Chain Ordering

Advisor ordering determines which advisors observe each iteration:
- **ToolCallAdvisor**: Near `HIGHEST_PRECEDENCE` so inner advisors observe tool execution
- **StructuredOutputValidationAdvisor**: Near `LOWEST_PRECEDENCE`, runs last before model call
- **Custom advisors**: Position based on observability needs

## Streaming Limitations

Recursive advisors currently support only non-streaming mode. The advisor needs the complete response before deciding whether to retry.

# Prerequisites

## Required Dependencies

- **Java**: 25 or higher (for record support and modern features)
- **Spring Boot**: 3.4 or 3.5
- **Spring AI**: 1.1.2 or higher
- **Build Tool**: Gradle 9.2.1+ with Kotlin DSL

## Configuration

Add the following to your `build.gradle.kts`:

```kotlin
plugins {
    java
}

group = "com.iokays"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_25
    targetCompatibility = JavaVersion.VERSION_25
}

dependencies {
    implementation(platform("org.springframework.boot:spring-boot-dependencies:3.5.0"))
    implementation(platform("org.springframework.ai:spring-ai-bom:1.1.2"))

    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.ai:spring-ai-starter-model-openai")
    implementation("org.springframework.boot:spring-boot-starter-test")

    testImplementation("org.junit.jupiter:junit-jupiter:6.0.1")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher:6.0.1")
}
```

## API Key Configuration

Set the `SPRING_AI_OPENAI_API_KEY` environment variable or add to `application.properties`:

```properties
spring.ai.openai.api-key=your-api-key-here
```

# Important Notes

## Retry Limits

Always set a maximum retry limit to prevent:
- Infinite loops consuming tokens
- Excessive API costs
- Long wait times for users

**Recommended limits:**
- Quality checks: 3-5 retries
- Validation retries: 3-5 retries (default in `StructuredOutputValidationAdvisor`)
- Custom advisors: Based on expected success probability

## Performance Considerations

Each iteration increases:
- API cost (additional LLM calls)
- Latency (multiple round-trips)
- Token usage (full prompts re-sent each time)

**Mitigation strategies:**
- Set strict retry limits
- Cache intermediate results where possible
- Keep augmented prompts concise
- Place advisors strategically in the chain

## Testing Strategy

When testing recursive advisors:
1. Mock the `ChatModel` to control responses
2. Verify retry behavior by checking call counts
3. Test edge cases (null content, too-short, too-long responses)
4. Verify termination conditions work correctly

See `QualityCheckAdvisorTest` for a complete example.

## Observability

Recursive advisors preserve full observability because:
- Downstream advisors observe every iteration
- Logging advisors placed after recursive ones capture each retry
- Metrics can track retry counts and success rates

# Best Practices

## When to Use Recursive Advisors

**Good use cases:**
- Quality checks requiring feedback
- Tool calling with multiple steps
- Structured output validation
- Complex multi-turn interactions

**Avoid when:**
- Simple one-shot requests
- Streaming responses required
- Latency is critical

## Design Guidelines

1. **Always define termination conditions**
   - Maximum retry count
   - Success criteria
   - Timeout limits

2. **Keep feedback concise**
   - Add only necessary information
   - Avoid growing prompts exponentially

3. **Test retry logic thoroughly**
   - Unit tests with mocked responses
   - Integration tests with real API
   - Edge case coverage

4. **Monitor performance**
   - Track retry rates
   - Measure latency impact
   - Monitor token costs

## Common Patterns

### Quality Check Pattern

```java
while (attempts < MAX_RETRIES && !meetsQuality(response)) {
    augmentWithFeedback(request, getFeedback(response));
    response = retry(request);
    attempts++;
}
```

### Validation Pattern

```java
while (attempts < MAX_RETRIES && !isValid(response)) {
    augmentWithError(request, getValidationError(response));
    response = retry(request);
    attempts++;
}
```

### Tool Calling Pattern

```java
while (response.containsToolCalls()) {
    executeToolCalls(response);
    feedResultsBack(request);
    response = retry(request);
}
```

# Success Criteria

A recursive advisor implementation is considered successful when:

1. ✅ Implements `CallAdvisor` interface
2. ✅ Defines clear termination conditions (max retries + exit criteria)
3. ✅ Uses `chain.copy(this).nextCall()` for retries
4. ✅ Provides meaningful feedback in retry loops
5. ✅ Has comprehensive unit tests
6. ✅ Handles edge cases (null content, empty responses)
7. ✅ Documents retry strategy and behavior
8. ✅ Follows ordering best practices
9. ✅ Preserves observability for downstream advisors
10. ✅ All tests pass with high coverage
