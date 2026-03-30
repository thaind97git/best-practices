---
name: react
description: Comprehensive React best practices guide sourced from react.dev official documentation. Covers 9 categories with rules for component design, hooks, state, effects, refs, and custom hooks. Apply when writing, reviewing, or refactoring React code.
user-invocable: true
metadata:
  source: https://react.dev
  version: "1.0"
---

# React Best Practices

Comprehensive best practices sourced from the official React documentation (react.dev). Contains rules across 9 categories, organized by impact to guide component authoring, state design, and effect usage.

---

## When to Apply

Reference these guidelines when:

- Writing new React components or hooks
- Reviewing component design and state structure
- Debugging stale closures, infinite loops, or stale state bugs
- Refactoring class components or effect-heavy code
- Deciding whether to use state, a ref, or derived values
- Designing custom hooks for shared logic

---

## Rule Categories by Priority

| Priority | Category                   | Impact   | Prefix    |
| -------- | -------------------------- | -------- | --------- |
| 1        | Rules of Hooks             | CRITICAL | hooks-    |
| 2        | Component Purity           | CRITICAL | purity-   |
| 3        | Effects — When to Use      | HIGH     | effect-   |
| 4        | Effect Dependencies        | HIGH     | deps-     |
| 5        | State Structure            | HIGH     | state-    |
| 6        | State Preservation & Reset | MEDIUM   | preserve- |
| 7        | Events vs Effects          | MEDIUM   | evfx-     |
| 8        | Refs                       | MEDIUM   | ref-      |
| 9        | Custom Hooks               | LOW-MED  | hook-     |

---

## Quick Reference

### 1. Rules of Hooks (CRITICAL)

- hooks-top-level — Only call Hooks at the top level, never inside conditions or loops
- hooks-react-only — Only call Hooks from React functions or custom Hooks
- hooks-no-direct-call — Never call component functions directly; always use JSX
- hooks-no-pass — Never pass Hooks as values or arguments
- hooks-immutable-returns — Never mutate props, state, or Hook return values

---

### 2. Component Purity (CRITICAL)

- purity-same-output — Components must return the same JSX for the same inputs
- purity-no-render-side-effects — Never produce side effects during render
- purity-no-prop-mutation — Never mutate props or objects received from outside
- purity-local-mutation-ok — Local mutation (objects created during render) is acceptable
- purity-no-direct-dom — Never access or mutate the DOM directly during render

---

### 3. Effects — When to Use (HIGH)

- effect-external-only — Use Effects only for synchronizing with external systems
- effect-cleanup — Always implement cleanup in Effects that start a subscription or connection
- effect-no-derived-state — Do not use Effects to compute derived state
- effect-no-event-post — Do not use Effects to handle user event side effects; use event handlers
- effect-fetch-ignore — Always use an `ignore` flag or AbortController to prevent race conditions in fetching Effects
- effect-no-chain — Do not chain Effects that set state; consolidate into event handlers
- effect-app-init — Use module-level code (or an `if (!initialized)` guard) for one-time app initialization

---

### 4. Effect Dependencies (HIGH)

- deps-exhaustive — All reactive values read inside an Effect must be listed as dependencies
- deps-no-suppress — Never suppress the exhaustive-deps ESLint rule
- deps-change-code — To remove a dependency, change the code — not the dep list
- deps-updater-fn — Use updater functions (`setState(prev => ...)`) to avoid reading state in Effects
- deps-extract-primitives — Destructure object props to primitives before using as deps
- deps-objects-inside — Create objects/functions inside the Effect instead of outside to stabilize deps
- deps-split-effects — Split one Effect into multiple if unrelated concerns share the same Effect

---

### 5. State Structure (HIGH)

- state-group-related — Group state that always changes together into one object
- state-no-contradiction — Replace multiple booleans with a single status string to prevent impossible states
- state-no-redundant — Never store values that can be derived from existing state or props
- state-no-duplication — Store IDs instead of full objects to avoid stale duplicates
- state-no-deep-nesting — Normalize/flatten deeply nested state structures
- state-no-mirror-props — Do not mirror props in state; use the `initial` naming convention to be explicit

---

### 6. State Preservation & Reset (MEDIUM)

- preserve-position — State is tied to position in the render tree, not JSX tags
- preserve-no-nested-defs — Never define components inside another component
- preserve-stable-keys — Always use stable, unique IDs as `key` — never array indices
- preserve-key-reset — Use `key` prop to intentionally reset a component's state
- preserve-consistent-tree — Keep the component tree shape consistent across conditional branches

---

### 7. Events vs Effects (MEDIUM)

- evfx-event-handler-first — Put user-interaction side effects in event handlers, not Effects
- evfx-effect-reactive — Effects re-run when any dependency changes; event handlers do not
- evfx-use-effect-event — Use `useEffectEvent` to read latest reactive values non-reactively inside Effects
- evfx-effect-event-rules — Only call Effect Events from inside Effects; never pass them to other components

---

### 8. Refs (MEDIUM)

- ref-non-rendering — Use refs for data that does not affect the rendered output
- ref-state-for-ui — Use state (not refs) for any value shown in the UI
- ref-no-render-read — Do not read or write `ref.current` during rendering
- ref-dom-access — Use refs to access DOM nodes imperatively (focus, scroll, measure)

---

### 9. Custom Hooks (LOW-MEDIUM)

- hook-use-prefix — Custom hook names must start with `use` followed by a capital letter
- hook-no-use-for-non-hooks — Do not add `use` prefix to functions that call no Hooks
- hook-share-logic-not-state — Each call to a custom Hook creates independent state; Hooks share logic only
- hook-extract-when-repeated — Extract a custom Hook when `useState + useEffect` patterns repeat across components
- hook-no-lifecycle-wrapper — Avoid generic lifecycle wrappers like `useMount` that obscure dependencies

---

## Category Files

Each category has its own `SKILL.md` with full rule details, anti-patterns, and code examples:

| Category                   | File                                                                       | Priority |
| -------------------------- | -------------------------------------------------------------------------- | -------- |
| Rules of Hooks             | [react-hooks-rules/SKILL.md](./react-hooks-rules/SKILL.md)                 | CRITICAL |
| Component Purity           | [react-component-purity/SKILL.md](./react-component-purity/SKILL.md)       | CRITICAL |
| Effects — When to Use      | [react-effects/SKILL.md](./react-effects/SKILL.md)                         | HIGH     |
| Effect Dependencies        | [react-effect-dependencies/SKILL.md](./react-effect-dependencies/SKILL.md) | HIGH     |
| State Structure            | [react-state-structure/SKILL.md](./react-state-structure/SKILL.md)         | HIGH     |
| State Preservation & Reset | [react-state-preservation/SKILL.md](./react-state-preservation/SKILL.md)   | MEDIUM   |
| Events vs Effects          | [react-events-vs-effects/SKILL.md](react-events-vs-effects/SKILL.md)       | MEDIUM   |
| Refs                       | [react-refs/SKILL.md](react-refs/SKILL.md)                                 | MEDIUM   |
| Custom Hooks               | [react-custom-hooks/SKILL.md](react-custom-hooks/SKILL.md)                 | LOW-MED  |

---

## Related Commands

- `/review-code` — Review source code quality against these rules
- `/write-test` — Generate tests for a React component or hook
