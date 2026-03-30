---
name: vitest-async
description: Vitest async patterns — 9 rules for correctly handling async assertions, timers, polling, and concurrency. Apply when writing async tests, using fake timers, or debugging flaky async behavior.
metadata:
  author: vercel
  version: "1.0"
---

# Async Patterns (CRITICAL)

9 rules for correctly handling asynchronous code in Vitest tests.

---

### async-await-assertions

**One-liner:** Await async assertions to prevent false positives.

**When to apply:** Any time an assertion runs inside an `async` function or inside `waitFor`, `expect.poll`, or `.resolves`/`.rejects` chains.

```ts
// BAD — unawaited assertion always passes, even if the promise rejects
test('data loads', () => {
  expect(fetchData()).resolves.toBe('ok')
})
```

```ts
// GOOD
test('data loads', async () => {
  await expect(fetchData()).resolves.toBe('ok')
})
```

---

### async-return-promises

**One-liner:** Return promises from test functions or use `async/await` — never fire-and-forget.

**When to apply:** When the test body contains a `.then()` chain or creates a promise it does not await.

```ts
// BAD — promise floats; test ends before assertion runs
test('resolves', () => {
  fetchData().then((data) => {
    expect(data).toBe('ok')
  })
})
```

```ts
// GOOD
test('resolves', async () => {
  const data = await fetchData()
  expect(data).toBe('ok')
})
```

---

### async-fake-timers

**One-liner:** Use `vi.useFakeTimers()` to control time-dependent code deterministically.

**When to apply:** When the code under test uses `setTimeout`, `setInterval`, `Date.now()`, or `debounce`/`throttle` utilities.

```ts
// BAD — real timer makes the test slow and timing-dependent
test('debounced call fires', async () => {
  const fn = vi.fn()
  debounce(fn, 300)()
  await new Promise((r) => setTimeout(r, 400))
  expect(fn).toHaveBeenCalled()
})
```

```ts
// GOOD
beforeEach(() => { vi.useFakeTimers() })
afterEach(() => { vi.useRealTimers() })

test('debounced call fires', () => {
  const fn = vi.fn()
  debounce(fn, 300)()
  vi.advanceTimersByTime(300)
  expect(fn).toHaveBeenCalled()
})
```

---

### async-waitfor-polling

**One-liner:** Use `vi.waitFor` to retry a callback until it succeeds or times out.

**When to apply:** When waiting for a side-effect that isn't directly controlled by the test (DOM mutation, store update, async event handler).

```ts
// BAD — arbitrary sleep is fragile and slow
await new Promise((r) => setTimeout(r, 500))
expect(element.textContent).toBe('loaded')
```

```ts
// GOOD
await vi.waitFor(
  () => expect(element.textContent).toBe('loaded'),
  { timeout: 1000, interval: 50 }
)
```

**Note:** With fake timers active, `vi.waitFor` automatically advances time by `interval` on each retry.

**Note (React component tests):** Prefer `waitFor` from RTL over `vi.waitFor` — RTL's `waitFor` is aware of React's rendering cycle and flushes pending state updates automatically. Use `vi.waitFor` for non-DOM async conditions (store updates, service calls, etc.).

---

### async-concurrent-expect

**One-liner:** In concurrent tests, use the `expect` from the local test context, not the global one.

**When to apply:** Inside any `describe.concurrent` or `it.concurrent` block.

```ts
// BAD — global expect may attribute the assertion to the wrong test
describe.concurrent('suite', () => {
  it('test 1', async () => {
    expect(await getValue()).toBe(1)
  })
})
```

```ts
// GOOD
describe.concurrent('suite', () => {
  it('test 1', async ({ expect }) => {
    expect(await getValue()).toBe(1)
  })
})
```

**Note:** Snapshots inside concurrent tests also require the local `expect` — global snapshots will throw.

---

### async-act-wrapper

**One-liner:** Await `userEvent` calls and wrap manual state triggers in `act()` to avoid act warnings.

**When to apply:** When using `@testing-library/user-event` or manually triggering state updates outside RTL's helpers.

```ts
// BAD — synchronous fireEvent for a state-updating async handler
fireEvent.click(button)
expect(screen.getByText('Done')).toBeInTheDocument()
```

```ts
// GOOD — userEvent.setup() handles act() internally
const user = userEvent.setup()
await user.click(button)
expect(screen.getByText('Done')).toBeInTheDocument()
```

**Note:** Prefer `userEvent.setup()` over the static `userEvent.click()` for correct async handling in Vitest.

---

### async-error-handling

**One-liner:** Test async error handling with `.rejects` or by catching inside the test body with `expect.assertions`.

**When to apply:** When the function under test is expected to reject or throw asynchronously.

```ts
// BAD — no assertion on rejection; test passes even if no error is thrown
test('throws', async () => {
  try {
    await riskyOperation()
  } catch (e) {
    // silently swallows
  }
})
```

```ts
// GOOD
test('throws on invalid input', async () => {
  await expect(riskyOperation()).rejects.toThrow('Invalid input')
})
```

---

### async-run-timers-async

**One-liner:** Use `vi.runAllTimersAsync()` instead of `vi.runAllTimers()` when timer callbacks are async.

**When to apply:** When a `setTimeout`/`setInterval` callback contains `await`, returns a Promise, or schedules microtasks.

```ts
// BAD — synchronous drain skips unresolved microtasks from async callbacks
setTimeout(async () => {
  await fetchSomething() // never awaited
  setState('done')
}, 100)

vi.runAllTimers()
expect(getState()).toBe('done') // flaky — microtask not drained
```

```ts
// GOOD
setTimeout(async () => {
  await fetchSomething()
  setState('done')
}, 100)

await vi.runAllTimersAsync()
expect(getState()).toBe('done')
```

**Note:** `vi.runAllTimersAsync()` is Vitest-specific. Jest's `runAllTimers()` has no async variant. Requires `vi.useFakeTimers()` to be active.

---

### async-expect-poll

**One-liner:** Use `expect.poll()` as a cleaner alternative to nested `waitFor + expect` combos.

**When to apply:** When retrying a value-returning getter until it matches an assertion. Prefer over `vi.waitFor` when the condition is a pure value check.

```ts
// BAD — verbose nested pattern
await waitFor(() => {
  expect(getStatus()).toBe('done')
})
```

```ts
// GOOD
await expect.poll(() => getStatus(), { interval: 50, timeout: 2000 }).toBe('done')
```

**Note:** `expect.poll()` is Vitest-native (stable since Vitest 1.x). Does not work with snapshot matchers, `.resolves`/`.rejects`, or `toThrow`. All `expect.poll` assertions must be awaited.
