---
name: unit-test
description: "Use when writing or editing unit tests."
---

# unit-test

## Overview

A test is a claim about what the code **does** for whoever uses it. It is never a claim about **how** the code does it.

Every test must pass two checks:

1. **Refactor check:** If someone rewrote the internals without changing behavior, would this test still pass? It must.
2. **Break check:** If the behavior this test names stopped working, would this test fail? It must.

A test that fails the refactor check is brittle. A test that fails the break check provides insufficent protection.

Rewrite tests that protect meaningful contracts.
Delete tests only when their behavior is already covered or is not a requirement.

## When to use

- Adding a new logic or behavior test
- Refactoring existing tests
- Adding edge case handling tests

Do not change expected behavior merely to accommodate implementation changes. Rewrite tests that depend on internals.

## Arrange Act Assert

**Unbreakable rules:**
- **Arrange:** Set up only what the behavior needs. Keep relevant inputs visible and control time, randomness, and other unpredictable inputs.
- **Act:** Exercise one distinct behavioral contract. Multiple calls are fine when testing a sequence. Keep independently meaningful behaviors in separate tests, even when they involve the same feature.
- **Assert:** Check the observable outcome promised by the test name. Multiple assertions are fine when they establish that outcome. Expected results must come from the requirement, not from the implementation's logic.
- Separate these phases with comments: // Arrange, // Act, // Assert

## Test behavior over implementation

```java
// Bad test
@Test
void save_callsFileWriterOnce() throws Exception {
    // Act
    storage.save(clip);

    // Assert
    verify(writerMock, times(1)).write(any(byte[].class));
}

// Good test
@Test
void save_persistsScreenshot() throws Exception {
    // Act
    Path path = storage.save(clip);

    // Assert
    assertTrue(Files.exists(path));
    assertArrayEquals(expectedBytes, Files.readAllBytes(path));
}
```

## DAMP Over DRY in Tests

In production code, DRY (Don't Repeat Yourself) is usually right. In tests, **DAMP (Descriptive And Meaningful Phrases)** is better. A test should read like a specification — each test should tell a complete story without requiring the reader to trace through shared helpers.

```java
// DAMP: Each test is self-contained and readable
@Test
void rejectsTasksWithEmptyTitles() {
    // Arrange
    var input = new TaskInput("", "user-1");

    // Act & Assert
    var error = assertThrows(IllegalArgumentException.class, () -> createTask(input));
    assertEquals("Title is required", error.getMessage());
}

@Test
void trimsWhitespaceFromTitles() {
    // Arrange
    var input = new TaskInput("  Buy groceries  ", "user-1");

    // Act
    Task task = createTask(input);

    // Assert
    assertEquals("Buy groceries", task.title());
}

// Over-DRY: Shared setup obscures what each test actually verifies
// (Don't do this just to avoid repeating the input shape)
```

Duplication in tests is acceptable when it makes each test independently understandable.

## Real tests over mock tests

Prefer real dependencies when they are fast, deterministic, and safe to run.

Use test doubles when real dependencies require external services, cause unwanted side effects, or make failure cases difficult to reproduce.

## How many tests

Choose tests based on distint behavioral contracts, not a fixed checklist or target count.

Prioritize:
- Core success scenarios.
- Expected failure paths.
- One test for a general failure path.
- Boundaries where expected behavior changes.
- Known regressions.

Minimize redundant coverage, not test count.

Two tests are redundant only when they protect the same behavior under equivalent conditions.

Tests using the same methods are not necessarily redundant.
Distinct states, transitions, and failure conditions may require separate tests.
