---
name: vitest-assertions
description: Vitest assertion best practices — 5 rules for using specific matchers, covering edge cases, one concept per test, expect.assertions for async, and toBe vs toEqual. Apply when writing or reviewing any expect() call.
metadata:
  author: vercel
  version: "1.0"
---

# Assertions (LOW-MEDIUM)

5 rules for writing clear, reliable, and precise assertions.

---

### assert-specific-matchers

**One-liner:** Use the most specific matcher available — prefer `toBeInTheDocument` over `toBeTruthy`, `toHaveBeenCalledWith` over `toHaveBeenCalled`.

**When to apply:** In every assertion — always ask if a more specific matcher exists.

```ts
// BAD — passes for any truthy value; failure message is unhelpful
expect(screen.queryByText('Error')).toBeTruthy()
```

```ts
// GOOD — specific DOM assertion with a clear failure message
expect(screen.getByText('Error')).toBeInTheDocument()
```

---

### assert-edge-cases

**One-liner:** Cover edge cases: empty arrays, null/undefined inputs, boundary values, and error states.

**When to apply:** When writing tests for a function that accepts user input or external data.

```ts
describe('formatCurrency', () => {
  it('should format positive amounts', () => { ... })
  it('should format zero', () => { ... })
  it('should format negative amounts', () => { ... })
  it('should handle null input', () => { ... })
})
```

---

### assert-one-assertion-concept

**One-liner:** Each test should verify one concept — split tests that assert multiple unrelated behaviors.

**When to apply:** When a test has more than 2–3 `expect()` calls asserting different concerns.

```ts
// BAD — mixing render, interaction, and navigation assertions
test('form works', async () => {
  render(<Form />)
  expect(screen.getByRole('button')).toBeDisabled()
  await user.type(screen.getByRole('textbox'), 'hello')
  expect(screen.getByRole('button')).toBeEnabled()
  await user.click(screen.getByRole('button'))
  expect(mockNavigate).toHaveBeenCalledWith('/success')
})
```

```ts
// GOOD — separate concerns into focused tests
it('should disable submit button when input is empty', () => { ... })
it('should enable submit button when input is filled', async () => { ... })
it('should navigate to success page on submit', async () => { ... })
```

---

### assert-expect-assertions

**One-liner:** Use `expect.assertions(n)` to verify that async assertions actually ran.

**When to apply:** In async tests where an assertion could be skipped if a promise resolves early or a catch block is empty.

```ts
// BAD — if fetchData() resolves without throwing, catch is skipped and the test passes vacuously
test('handles error', async () => {
  try {
    await fetchData()
  } catch (e) {
    expect(e.message).toBe('Network error')
  }
})
```

```ts
// GOOD — guarantees the assertion inside catch ran exactly once
test('handles error', async () => {
  expect.assertions(1)
  try {
    await fetchData()
  } catch (e) {
    expect(e.message).toBe('Network error')
  }
})
```

---

### assert-toequal-vs-tobe

**One-liner:** Use `toBe` for primitives and reference equality; use `toEqual` for deep structural equality.

**When to apply:** When asserting on object or array values.

```ts
// BAD — toBe on objects compares references, not structure; always fails
expect(getUser()).toBe({ id: 1, name: 'Alice' })
```

```ts
// GOOD — deep equality for objects
expect(getUser()).toEqual({ id: 1, name: 'Alice' })

// primitives — toBe is correct
expect(getCount()).toBe(42)
```
