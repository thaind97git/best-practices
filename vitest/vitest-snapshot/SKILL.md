---
name: vitest-snapshot
description: Vitest snapshot testing — 5 rules for inline vs file snapshots, avoiding large snapshots, stable serialization, reviewing updates, and descriptive names. Apply when writing or updating snapshot assertions.
metadata:
  author: vercel
  version: "1.0"
---

# Snapshot Testing (MEDIUM)

5 rules for writing maintainable and reliable snapshot tests.

---

### snap-inline-over-file

**One-liner:** Prefer `toMatchInlineSnapshot()` over `toMatchSnapshot()` for small, reviewable snapshots.

**When to apply:** For snapshots under ~20 lines. Use file snapshots only for large outputs (full SVG, large JSON).

```ts
// BAD — snapshot lives in a separate __snapshots__ file; hard to review in PRs
expect(output).toMatchSnapshot()
```

```ts
// GOOD — snapshot content is visible inline next to the assertion
expect(output).toMatchInlineSnapshot(`
  "<div class=\\"container\\">Hello</div>"
`)
```

---

### snap-avoid-large

**One-liner:** Avoid snapshotting large objects or full component trees — snapshot only the meaningful subtree.

**When to apply:** When a snapshot would exceed ~30 lines or captures irrelevant implementation details.

```ts
// BAD — entire page snapshot breaks on any unrelated UI change
expect(container).toMatchSnapshot()
```

```ts
// GOOD — snapshot only the relevant part
expect(screen.getByRole('dialog')).toMatchInlineSnapshot(...)
```

---

### snap-stable-serialization

**One-liner:** Ensure snapshot output is deterministic — avoid dates, random IDs, or non-deterministic ordering.

**When to apply:** When the component or function generates dynamic values (timestamps, UUIDs, random numbers).

```ts
// BAD — Date.now() produces a different snapshot every run
expect({ createdAt: Date.now(), label: 'test' }).toMatchSnapshot()
```

```ts
// GOOD — mock time before snapshotting
vi.setSystemTime(new Date('2024-01-01'))
expect({ createdAt: Date.now(), label: 'test' }).toMatchInlineSnapshot(`
  { "createdAt": 1704067200000, "label": "test" }
`)
```

---

### snap-review-updates

**One-liner:** Always review snapshot diffs in PRs — never blindly run `--update-snapshots`.

**When to apply:** When CI fails with a snapshot mismatch, or after upgrading Vitest or a UI library.

**Guidance:** Run `vitest run --update-snapshots` locally, review each diff carefully, then commit only intentional changes. Configure CI to fail on unexpected snapshot changes rather than auto-updating.

---

### snap-describe-intent

**One-liner:** Give snapshot tests a name that describes what aspect is being captured — pass the hint as the **last** argument to `toMatchInlineSnapshot`.

**When to apply:** When more than one snapshot exists in a test file.

```ts
// BAD — generic name gives no context on failure
expect(component).toMatchSnapshot()
```

```ts
// GOOD — hint (descriptive label) is the LAST argument, not the first
// First arg is the snapshot content; second arg is the hint/name
expect(screen.getByRole('dialog')).toMatchInlineSnapshot(
  `"<div class=\\"dialog\\">Confirm delete?</div>"`,
  'error-state-with-retry-button'
)
```

**Note:** The Vitest v2 signature is `toMatchInlineSnapshot(snapshot?: string, hint?: string)` or `toMatchInlineSnapshot(shape: Partial<T>, snapshot?: string, hint?: string)`. Passing a plain string as the **first** argument is treated as the snapshot content itself — not a name. The descriptive label is always the **last** argument (`hint`).
