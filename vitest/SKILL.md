---
name: vitest
description: Comprehensive performance optimization and best practices guide for Vitest testing framework. Includes 52 rules across 8 categories, prioritized by impact for writing, reviewing, and optimizing tests.
user-invocable: true
---

# Vitest Best Practices

Comprehensive performance optimization and best practices guide for Vitest testing framework. Contains 52 rules across 8 categories, prioritized by impact to guide test writing, refactoring, and code review.

---

## When to Apply

Reference these guidelines when:

- Writing new Vitest tests
- Debugging flaky or slow tests
- Setting up test configuration
- Reviewing test code in pull requests
- Migrating from Jest to Vitest
- Optimizing CI/CD test performance

---

## Rule Categories by Priority

| Priority | Category               | Impact     | Prefix  |
| -------- | ---------------------- | ---------- | ------- |
| 1        | Async Patterns         | CRITICAL   | async-  |
| 2        | Test Setup & Isolation | CRITICAL   | setup-  |
| 3        | Mocking Patterns       | HIGH       | mock-   |
| 4        | Performance            | HIGH       | perf-   |
| 5        | Snapshot Testing       | MEDIUM     | snap-   |
| 6        | Environment            | MEDIUM     | env-    |
| 7        | Assertions             | LOW-MEDIUM | assert- |
| 8        | Test Organization      | LOW        | org-    |

---

## Quick Reference

### 1. Async Patterns (CRITICAL)

- async-await-assertions — Await async assertions to prevent false positives
- async-return-promises — Return promises from test functions
- async-fake-timers — Use fake timers for time-dependent code
- async-waitfor-polling — Use `vi.waitFor` for async conditions
- async-concurrent-expect — Use test context `expect` in concurrent tests
- async-act-wrapper — Await user events to avoid act warnings
- async-error-handling — Test async error handling properly
- async-run-timers-async — Use `vi.runAllTimersAsync()` when timers trigger async callbacks
- async-expect-poll — Use `expect.poll()` as a cleaner alternative to `waitFor + expect` combos

---

### 2. Test Setup & Isolation (CRITICAL)

- setup-beforeeach-cleanup — Clean up state in `afterEach`
- setup-restore-mocks — Restore mocks after each test
- setup-avoid-shared-state — Avoid shared mutable state
- setup-beforeall-expensive — Use `beforeAll` for expensive setup
- setup-reset-modules — Reset modules when needed
- setup-test-factories — Use factories for complex test data
- setup-on-test-finished — Use `onTestFinished()` inside test body for per-test resource cleanup

---

### 3. Mocking Patterns (HIGH)

- mock-vi-mock-hoisting — Understand `vi.mock` hoisting behavior
- mock-spyon-vs-mock — Choose `vi.spyOn` vs `vi.mock`
- mock-implementation-not-value — Use `mockImplementation` for dynamic behavior
- mock-msw-network — Use MSW for API mocking
- mock-avoid-overmocking — Avoid excessive mocking
- mock-type-safety — Maintain type safety in mocks
- mock-clear-between-tests — Clear mocks between tests
- mock-vi-hoisted — Use `vi.hoisted()` to declare variables used inside `vi.mock()` factory functions
- mock-jest-dom-vitest-import — Import `@testing-library/jest-dom/vitest` not `/jest` for DOM matchers

---

### 4. Performance (HIGH)

- perf-pool-selection — Choose optimal test pool (`forks`, `threads`, `vmThreads`, `vmForks`; default changed to `forks` in v2)
- perf-disable-isolation — Disable isolation when safe
- perf-happy-dom — Prefer `happy-dom` over `jsdom`
- perf-sharding — Use sharding for parallel CI
- perf-run-mode-ci — Use run mode in CI
- perf-bail-fast-fail — Fail fast with bail option
- perf-coverage-v8 — Use `coverage.provider: 'v8'` over `'istanbul'` for faster CI coverage runs
- perf-blob-reporter-merge — Use `--reporter=blob` per shard and `--merge-reports` for unified CI reports

---

### 5. Snapshot Testing (MEDIUM)

- snap-inline-over-file — Prefer inline snapshots
- snap-avoid-large — Avoid large snapshots
- snap-stable-serialization — Ensure stable serialization
- snap-review-updates — Review snapshot changes
- snap-describe-intent — Use descriptive snapshot names

---

### 6. Environment (MEDIUM)

- env-per-file-override — Override environment per file
- env-setup-files — Use setup files for global config
- env-globals-config — Configure globals consistently
- env-browser-api-mocking — Mock missing browser APIs

---

### 7. Assertions (LOW-MEDIUM)

- assert-specific-matchers — Use specific matchers
- assert-edge-cases — Cover edge cases
- assert-one-assertion-concept — One concept per test
- assert-expect-assertions — Use `expect.assertions`
- assert-toequal-vs-tobe — Use correct matcher

---

### 8. Test Organization (LOW)

- org-file-colocation — Keep tests near source
- org-describe-nesting — Use logical grouping
- org-test-naming — Write clear test names
- org-test-skip-only — Use skip/only carefully
- test-concurrent-describe — Use `describe.concurrent` to run independent tests in parallel within a file

---

## Category Files

Each category has its own `SKILL.md` with full rule details, anti-patterns, and code examples:

| Category               | File                                                         | Priority   |
| ---------------------- | ------------------------------------------------------------ | ---------- |
| Async Patterns         | [vitest-async/SKILL.md](vitest-async/SKILL.md)               | CRITICAL   |
| Test Setup & Isolation | [vitest-setup/SKILL.md](vitest-setup/SKILL.md)               | CRITICAL   |
| Mocking Patterns       | [vitest-mocking/SKILL.md](vitest-mocking/SKILL.md)           | HIGH       |
| Performance            | [vitest-performance/SKILL.md](vitest-performance/SKILL.md)   | HIGH       |
| Snapshot Testing       | [vitest-snapshot/SKILL.md](vitest-snapshot/SKILL.md)         | MEDIUM     |
| Environment            | [vitest-environment/SKILL.md](vitest-environment/SKILL.md)   | MEDIUM     |
| Assertions             | [vitest-assertions/SKILL.md](vitest-assertions/SKILL.md)     | LOW-MEDIUM |
| Test Organization      | [vitest-organization/SKILL.md](vitest-organization/SKILL.md) | LOW        |
