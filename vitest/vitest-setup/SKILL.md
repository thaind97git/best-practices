---
name: vitest-setup
description: Vitest test setup and isolation — 7 rules for preventing test leakage via cleanup, mock restoration, shared state, and resource management. Apply when writing beforeEach/afterEach hooks or managing test lifecycle.
metadata:
  author: vercel
  version: "1.0"
---

# Test Setup & Isolation (CRITICAL)

7 rules for proper test lifecycle management and preventing inter-test contamination.

---

### setup-beforeeach-cleanup

**One-liner:** Clean up DOM, timers, and subscriptions in `afterEach` to prevent test-to-test leakage.

**When to apply:** Whenever a test renders a component, starts a timer, or subscribes to a store or event emitter.

```ts
// BAD — rendered component leaks into the next test's DOM queries
test('renders title', () => {
  render(<App />)
  expect(screen.getByText('Title')).toBeInTheDocument()
})
// next test may find stale DOM nodes
```

```ts
// GOOD — register cleanup in setup file
// src/tests/setup.ts
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

afterEach(() => cleanup())
```

**Note:** `@testing-library/react` auto-calls `cleanup()` in `afterEach` when the setup file imports it. Add explicitly only if using a custom renderer.

---

### setup-restore-mocks

**One-liner:** Restore all mock implementations after each test to prevent spy state from leaking.

**When to apply:** Whenever `vi.spyOn`, `vi.fn()`, or `vi.mock` with `{ spy: true }` is used.

```ts
// BAD — spy state persists into subsequent tests
const spy = vi.spyOn(console, 'warn')
test('warns', () => {
  myFn()
  expect(spy).toHaveBeenCalled()
})
// console.warn is still spied on in all following tests
```

```ts
// GOOD — restore after each test
afterEach(() => {
  vi.restoreAllMocks()
})
```

Or configure globally in `vitest.config.ts`:
```ts
test: {
  restoreMocks: true,
}
```

---

### setup-avoid-shared-state

**One-liner:** Never share mutable variables between tests; re-initialize in `beforeEach`.

**When to apply:** When a test file declares a `let` variable at module scope that multiple tests mutate.

```ts
// BAD — mutation in one test leaks to the next
let store = createStore()

test('increments', () => {
  store.increment()
  expect(store.count).toBe(1)
})
test('starts at zero', () => {
  expect(store.count).toBe(0) // fails — still 1 from previous test
})
```

```ts
// GOOD
let store: Store

beforeEach(() => {
  store = createStore()
})
```

---

### setup-beforeall-expensive

**One-liner:** Use `beforeAll` for expensive one-time setup that is safe to share across tests.

**When to apply:** When initializing a database connection, starting a server, or loading a large read-only fixture.

```ts
// BAD — re-creates server for every test; slow
beforeEach(async () => {
  server = await createTestServer()
})
```

```ts
// GOOD
let server: TestServer

beforeAll(async () => {
  server = await createTestServer()
})
afterAll(async () => {
  await server.close()
})
```

---

### setup-reset-modules

**One-liner:** Use `vi.resetModules()` when a test needs a fresh module registry.

**When to apply:** When testing module-level side effects (e.g., env var reads at import time, singleton initialization).

```ts
// BAD — module is cached; changing env var has no effect
process.env.FEATURE = 'true'
const { isEnabled } = await import('./feature')
expect(isEnabled).toBe(true)
```

```ts
// GOOD
beforeEach(() => {
  vi.resetModules()
})

test('respects FEATURE env var', async () => {
  process.env.FEATURE = 'true'
  const { isEnabled } = await import('./feature')
  expect(isEnabled).toBe(true)
})
```

**Note:** `vi.resetModules()` clears the module registry. `vi.clearAllMocks()` only resets call history. They are complementary, not interchangeable.

---

### setup-test-factories

**One-liner:** Use factory functions to build complex test data instead of duplicating object literals.

**When to apply:** When 3+ tests use the same data shape with minor variations.

```ts
// BAD — duplicated object literal across tests
test('validates name', () => {
  const user = { id: '1', name: '', email: 'a@b.com', role: 'admin' }
  expect(validate(user)).toBe(false)
})
```

```ts
// GOOD
const makeUser = (overrides?: Partial<User>): User => ({
  id: '1',
  name: 'Alice',
  email: 'alice@example.com',
  role: 'user',
  ...overrides,
})

test('validates name', () => {
  expect(validate(makeUser({ name: '' }))).toBe(false)
})
```

---

### setup-on-test-finished

**One-liner:** Use `onTestFinished()` inside the test body for per-test resource cleanup that runs even on failure.

**When to apply:** When cleanup is specific to one test and should not pollute `afterEach`, or when cleanup must run even if the test assertion throws.

```ts
// BAD — test-specific teardown in shared afterEach
let server: Server | undefined
afterEach(() => {
  if (server) { server.close(); server = undefined }
})

test('opens a connection', () => {
  server = createServer()
  expect(server.isListening).toBe(true)
})
```

```ts
// GOOD
test('opens a connection', ({ onTestFinished }) => {
  const server = createServer()
  onTestFinished(() => server.close())

  expect(server.isListening).toBe(true)
})
```

**Note:** `onTestFinished` is stable in Vitest 2.x. Must be called from test context — do not call from global hooks. Especially useful in `describe.concurrent` blocks where `afterEach` may race between tests.

**Concurrent test caveat:** In `describe.concurrent` blocks, always destructure `onTestFinished` from the **test context parameter** (`({ onTestFinished }) => {}`). Do not use the one imported globally from `vitest` — the global import cannot reliably associate cleanup with the correct concurrent test.
