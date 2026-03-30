---
name: vitest-performance
description: Vitest performance optimization — 8 rules for worker pool selection, isolation, happy-dom, sharding, CI run mode, bail, coverage provider, and blob reporter. Apply when tuning CI speed or configuring vitest.config.ts.
metadata:
  author: vercel
  version: "1.0"
---

# Performance (HIGH)

8 rules for optimizing Vitest test suite speed in local development and CI.

---

### perf-pool-selection

**One-liner:** Choose the right worker pool (`forks`, `threads`, `vmThreads`) based on your suite's isolation needs.

**When to apply:** When tuning CI performance or experiencing SharedArrayBuffer / DOM isolation issues.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    pool: 'threads', // faster on large suites; use 'forks' for better DOM isolation
  },
})
```

| Pool | Isolation | Speed | Use when |
|---|---|---|---|
| `forks` (default) | High | Moderate | DOM-heavy tests, SharedArrayBuffer issues |
| `threads` | Medium | Fast | Large non-DOM suites |
| `vmThreads` | VM context | Fast | Need VM isolation; cannot disable it |
| `vmForks` | VM + process | Slower | Need VM isolation + process-level memory separation |

**Breaking change from v1:** The default pool changed from `threads` → `forks` in Vitest v2. Existing configs that relied on the implicit default may behave differently after upgrading.

---

### perf-disable-isolation

**One-liner:** Disable per-file module isolation when tests have no module-level side effects.

**When to apply:** In suites of pure utility or business logic tests where each file does not mutate module state.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    isolate: false,
  },
})
```

**Note:** Do not disable isolation in DOM-heavy React component test suites — shared module state will cause flaky tests.

---

### perf-happy-dom

**One-liner:** Prefer `happy-dom` over `jsdom` for faster DOM simulation in React component tests.

**When to apply:** When the test suite uses a DOM environment and does not require jsdom-specific edge-case APIs.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    environment: 'happy-dom',
  },
})
```

**Note:** `happy-dom` is generally 2–4x faster than `jsdom`. If tests rely on obscure browser APIs not yet implemented in happy-dom, fall back to `jsdom`.

---

### perf-sharding

**One-liner:** Use `--shard` to distribute test files across parallel CI machines.

**When to apply:** When CI test run time exceeds 2 minutes and multiple parallel runners are available.

```bash
# Machine 1
vitest run --shard=1/3

# Machine 2
vitest run --shard=2/3

# Machine 3
vitest run --shard=3/3
```

**Note:** Sharding splits test *files*, not individual test cases. Balance shard count with the number of available CI runners.

---

### perf-run-mode-ci

**One-liner:** Use `vitest run` (not `vitest`) in CI to disable watch mode.

**When to apply:** In all CI pipeline scripts.

```yaml
# BAD — starts in watch mode, hangs CI indefinitely
- run: pnpm vitest

# GOOD
- run: pnpm vitest run
```

---

### perf-bail-fast-fail

**One-liner:** Use `bail: 1` to stop the test run immediately after the first failure in CI.

**When to apply:** In CI pipelines where you want immediate feedback rather than running the full suite to completion.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    bail: 1,
  },
})
```

---

### perf-coverage-v8

**One-liner:** Use `coverage.provider: 'v8'` over `'istanbul'` for faster CI coverage collection.

**When to apply:** As the default in all projects. Only switch to `istanbul` when branch-level accuracy on non-V8 runtimes (e.g., Bun) is required.

```ts
// BAD — istanbul instruments source code before execution; adds overhead
export default defineConfig({
  test: {
    coverage: { provider: 'istanbul' },
  },
})
```

```ts
// GOOD — v8 uses the runtime's native coverage; no instrumentation step
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['src/tests/**'],
    },
  },
})
```

Install: `npm i -D @vitest/coverage-v8`

**Note:** On v2.x, V8 coverage is faster but may show minor branch-coverage differences vs Istanbul. Full accuracy parity via AST remapping arrives in Vitest 3.2.0+.

---

### perf-blob-reporter-merge

**One-liner:** In sharded CI, use `--reporter=blob` per shard and `--merge-reports` to produce a unified report.

**When to apply:** When using `--shard` across multiple CI machines and a single HTML/JSON report is needed.

```bash
# Each shard outputs a blob file
vitest run --reporter=blob --shard=1/3
vitest run --reporter=blob --shard=2/3
vitest run --reporter=blob --shard=3/3

# Final aggregation job
vitest --merge-reports=./blob-reports --reporter=html --reporter=json
```

**Note:** Available in Vitest 2.x. The `blob` reporter writes compact binary files to `.vitest-reports/` by default. The merge step does not run tests — it only reads blob files.
