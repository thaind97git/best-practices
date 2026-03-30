---
name: vitest-organization
description: Vitest test organization — 5 rules for file colocation, describe nesting, test naming conventions, skip/only discipline, and describe.concurrent for parallel execution. Apply when structuring test files or naming tests.
metadata:
  author: vercel
  version: "1.0"
---

# Test Organization (LOW)

5 rules for keeping test suites readable and well-structured.

---

### org-file-colocation

**One-liner:** Keep test files in a `src/tests/` mirror of `src/` to make them easy to find.

**When to apply:** When adding a new test file to the project.

This project mirrors structure: `src/modules/Foo/Bar.tsx` → `src/tests/modules/Foo/Bar.test.tsx`.

---

### org-describe-nesting

**One-liner:** Use nested `describe` blocks to group related tests — outer for the unit, inner for a method or scenario.

**When to apply:** When a test file covers multiple methods or multiple behavioral scenarios of the same component.

```ts
describe('useAddressForm', () => {
  describe('validation', () => {
    it('should reject empty street', () => { ... })
    it('should reject invalid postal code', () => { ... })
  })

  describe('submission', () => {
    it('should call API on valid form', async () => { ... })
    it('should show error on API failure', async () => { ... })
  })
})
```

---

### org-test-naming

**One-liner:** Write test names as `it('should <behavior> when <condition>')` — describe behavior, not implementation.

**When to apply:** Every `it()` / `test()` call.

```ts
// BAD — describes the check, not the behavior
it('renders button', () => { ... })
it('calls onClick', () => { ... })
```

```ts
// GOOD
it('should disable the submit button when the form is invalid', () => { ... })
it('should call onSubmit with form values when the form is submitted', async () => { ... })
```

---

### org-test-skip-only

**One-liner:** Use `it.skip` and `it.only` as temporary debugging tools — never commit them.

**When to apply:** During local development only. Add a lint rule or CI check to block committed `.only` tests.

```ts
// BAD — accidentally committed; CI runs only this one test
it.only('my new test', () => { ... })
```

Use the `eslint-plugin-no-only-tests` plugin to fail CI on committed `.only` calls.

---

### test-concurrent-describe

**One-liner:** Use `describe.concurrent` to run independent tests in parallel within a file.

**When to apply:** When a `describe` block contains tests that share no mutable state and each test sets up its own data.

```ts
// BAD — sequential execution even though tests are independent
describe('formatters', () => {
  it('should format currency', () => { ... })
  it('should format date', () => { ... })
  it('should format phone number', () => { ... })
})
```

```ts
// GOOD — all three run in parallel
describe.concurrent('formatters', () => {
  it('should format currency', async ({ expect }) => { ... })
  it('should format date', async ({ expect }) => { ... })
  it('should format phone number', async ({ expect }) => { ... })
})
```

Use `test.sequential` to opt individual tests back out of concurrent execution:
```ts
describe.concurrent('suite', () => {
  it('runs in parallel', async ({ expect }) => { ... })
  it.sequential('must run after the previous test', async ({ expect }) => { ... })
})
```

**Note:** `describe.concurrent` is Vitest-specific (stable since Vitest 1.x). In concurrent tests, always use the test-local `expect` from the function parameter — global `expect` can attribute assertions to the wrong test. Snapshot assertions also require the local `expect`.
