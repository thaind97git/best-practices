---
name: vitest-mocking
description: Vitest mocking patterns — 9 rules for vi.mock hoisting, spyOn vs mock, MSW network interception, type safety, and vi.hoisted. Apply when writing or reviewing any test that mocks modules, functions, or network requests.
metadata:
  author: vercel
  version: "1.0"
---

# Mocking Patterns (HIGH)

9 rules for correct and maintainable mocking in Vitest.

---

### mock-vi-mock-hoisting

**One-liner:** `vi.mock()` is hoisted to the top of the file — place it at module scope, not inside `describe` or `it`.

**When to apply:** When mocking a module that is imported at the top of the file.

```ts
// BAD — vi.mock inside describe is not reliably hoisted
describe('MyComponent', () => {
  vi.mock('./api', () => ({ getData: vi.fn() }))
})
```

```ts
// GOOD — module-scope, hoisted automatically before imports
vi.mock('./api', () => ({ getData: vi.fn() }))

describe('MyComponent', () => {
  // ...
})
```

**Note:** If the mock factory references a variable, that variable must be declared with `vi.hoisted()` — see `mock-vi-hoisted`.

---

### mock-spyon-vs-mock

**One-liner:** Use `vi.spyOn` to track/replace a single method; use `vi.mock` to replace an entire module.

**When to apply:** `vi.spyOn` when you need the real implementation by default or want to override one method. `vi.mock` when all exports of a module must be replaced.

```ts
// BAD — full module mock just to replace one method; all other exports are lost
vi.mock('./utils', () => ({ formatDate: vi.fn(() => '01/01/2024') }))
```

```ts
// GOOD — spy preserves the rest of the module
import * as utils from './utils'
vi.spyOn(utils, 'formatDate').mockReturnValue('01/01/2024')
```

---

### mock-implementation-not-value

**One-liner:** Use `mockReturnValue` / `mockImplementation` for mock behavior; avoid reassigning the function reference directly.

**When to apply:** When a mock must return different values per call, or when behavior depends on arguments.

```ts
// BAD — direct reassignment loses Vitest's spy tracking
myMock = () => 'hardcoded'
```

```ts
// GOOD
const mockFn = vi.fn()
mockFn.mockReturnValueOnce('first').mockReturnValue('default')

// argument-dependent behavior
mockFn.mockImplementation((x: number) => x * 2)
```

---

### mock-msw-network

**One-liner:** Use MSW (Mock Service Worker) to intercept HTTP requests at the network level.

**When to apply:** When the code under test makes `fetch` or `axios` calls and you want to test the full request/response cycle without coupling to a mock function.

```ts
// BAD — mocking the entire axios module loses headers, status handling, interceptors
vi.mock('axios')
```

```ts
// GOOD — MSW intercepts at network level; real axios/fetch code runs
import { http, HttpResponse } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer(
  http.get('/api/users', () => HttpResponse.json([{ id: 1 }]))
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

---

### mock-avoid-overmocking

**One-liner:** Mock only what is necessary; avoid mocking implementation details or pure functions.

**When to apply:** When reviewing a test that mocks every dependency — ask if the test would still be valuable with fewer mocks.

```ts
// BAD — mocking a pure utility that has no side effects
vi.mock('./formatDate', () => ({ formatDate: vi.fn(() => '2024-01-01') }))
```

```ts
// GOOD — pure functions can be used directly; no mocking needed
import { formatDate } from './formatDate'
// use formatDate as-is in the test
```

---

### mock-type-safety

**One-liner:** Use `vi.mocked()` to preserve TypeScript types on mocked functions.

**When to apply:** When accessing `.mock.calls`, `.mockReturnValue`, or other mock properties on an imported function.

```ts
// BAD — TypeScript doesn't know the function is a mock
import { getData } from './api'
vi.mock('./api')
getData.mockReturnValue('test') // TS error: Property 'mockReturnValue' does not exist
```

```ts
// GOOD
import { getData } from './api'
vi.mock('./api')
vi.mocked(getData).mockReturnValue('test') // fully typed
```

---

### mock-clear-between-tests

**One-liner:** Clear mock state between tests to prevent call counts and return values from leaking.

**When to apply:** Whenever mocks are declared at module scope and shared across multiple tests.

```ts
// BAD — mock call count accumulates across tests
const mockFn = vi.fn()
test('called once', () => { myFn(mockFn); expect(mockFn).toHaveBeenCalledTimes(1) })
test('called once again', () => { myFn(mockFn); expect(mockFn).toHaveBeenCalledTimes(1) }) // actually 2
```

```ts
// GOOD
const mockFn = vi.fn()
beforeEach(() => { vi.clearAllMocks() })
```

Or globally via config:
```ts
test: { clearMocks: true }
```

---

### mock-vi-hoisted

**One-liner:** Use `vi.hoisted()` to declare variables referenced inside `vi.mock()` factory functions.

**When to apply:** When a `vi.mock()` factory needs to reference a `vi.fn()` or any variable that must also be accessible in test bodies.

```ts
// BAD — mockFn is declared after vi.mock is hoisted; it is undefined inside the factory
const mockFn = vi.fn()
vi.mock('./api', () => ({ getData: mockFn })) // mockFn is undefined at factory execution time
```

```ts
// GOOD — vi.hoisted runs before module evaluation
const { mockGetData } = vi.hoisted(() => ({
  mockGetData: vi.fn(),
}))

vi.mock('./api', () => ({ getData: mockGetData }))

test('calls getData', () => {
  mockGetData.mockResolvedValue({ id: 1 })
  // ...
})
```

**Note:** `vi.hoisted()` is Vitest-specific with no Jest equivalent. Supports async factories: `await vi.hoisted(async () => { ... })`. Cannot access imported module values inside `vi.hoisted` — they have not been evaluated yet.

---

### mock-jest-dom-vitest-import

**One-liner:** Import `@testing-library/jest-dom/vitest` (not `/jest`) to correctly register DOM matchers on Vitest's `expect`.

**When to apply:** In the Vitest setup file (`src/tests/setup.ts`) when using `@testing-library/jest-dom` matchers such as `toBeInTheDocument`.

```ts
// BAD — wrong entry point; silently fails to extend Vitest's expect
import '@testing-library/jest-dom'
// or
import '@testing-library/jest-dom/jest'
```

```ts
// GOOD — src/tests/setup.ts
import '@testing-library/jest-dom/vitest'
```

**Note:** The wrong import produces no error at setup time but causes `TypeError: expect(...).toBeInTheDocument is not a function` at assertion time — a common gotcha when migrating from Jest to Vitest.

**Note:** Requires `@testing-library/jest-dom` **v6.0.0+**. The `/vitest` entry point does not exist in v5.x — if you are on v5, upgrade to v6 first.
