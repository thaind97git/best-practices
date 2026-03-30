---
name: vitest-environment
description: Vitest environment configuration — 4 rules for per-file environment overrides, setup files, globals config, and mocking missing browser APIs. Apply when configuring vitest.config.ts or the global setup file.
metadata:
  author: vercel
  version: "1.0"
---

# Environment (MEDIUM)

4 rules for configuring the Vitest test environment correctly.

---

### env-per-file-override

**One-liner:** Override the test environment per file using the `@vitest-environment` docblock.

**When to apply:** When most tests run in `jsdom` but specific files need `node` (e.g., pure API integration tests).

```ts
// @vitest-environment node
import { describe, it, expect } from 'vitest'

describe('Node-only logic', () => {
  it('should use Buffer', () => {
    expect(Buffer.from('hello').toString('hex')).toBe('68656c6c6f')
  })
})
```

---

### env-setup-files

**One-liner:** Use `setupFiles` in `vitest.config.ts` for global test configuration — matchers, polyfills, cleanup.

**When to apply:** When the same setup code is needed in every test file.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    setupFiles: ['./src/tests/setup.ts'],
  },
})
```

```ts
// src/tests/setup.ts
import '@testing-library/jest-dom/vitest'
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

afterEach(() => cleanup())
```

---

### env-globals-config

**One-liner:** Configure `globals: true` only if your team prefers implicit globals; otherwise use explicit imports.

**When to apply:** When setting up a new project or migrating from Jest.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    globals: true, // enables describe/it/expect without imports
  },
})
```

**Note:** Explicit imports (`import { describe, it, expect, vi } from 'vitest'`) are preferred by the official Vitest docs — they improve IDE auto-complete and TypeScript inference.

---

### env-browser-api-mocking

**One-liner:** Mock browser APIs unavailable in jsdom/happy-dom in the setup file.

**When to apply:** When tests fail with `ReferenceError: ResizeObserver is not defined` or similar missing browser API errors.

```ts
// src/tests/setup.ts
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query: string) => ({
    matches: false,
    media: query,
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
  })),
})
```
