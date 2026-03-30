---
name: react-hooks-rules
description: React Rules of Hooks — 5 mandatory rules that govern where and how Hooks can be called. Breaking any of these causes subtle, hard-to-diagnose bugs. Apply when writing or reviewing any component or custom hook.
metadata:
  source: https://react.dev/reference/rules/rules-of-hooks
  version: "1.0"
---

# Rules of Hooks (CRITICAL)

5 mandatory rules enforced by the `react-hooks` ESLint plugin. Violating them causes incorrect hook call order, stale closures, and runtime errors.

---

### hooks-top-level

**One-liner:** Only call Hooks at the top level — never inside loops, conditions, or nested functions.

**When to apply:** Any time you write or review a `useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, `useContext`, or custom Hook call.

```tsx
// BAD — conditional Hook call breaks call order across renders
function MyComponent({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // WRONG
  }
}
```

```tsx
// GOOD — always called, same position every render
function MyComponent({ condition }) {
  const [state, setState] = useState(0);
  // use condition inside the body, not to gate the hook itself
  const value = condition ? state : 0;
}
```

**Why:** React tracks Hook order to pair each Hook call with its stored state. Any variation in call order between renders corrupts the mapping.

---

### hooks-react-only

**One-liner:** Only call Hooks from React function components or custom Hooks — never from plain JS functions, class components, or event callbacks.

**When to apply:** When extracting reusable logic — decide early whether the function is a Hook (needs `use` prefix + Hook calls) or a plain utility.

```ts
// BAD — Hook called from a regular function
function getFormattedData(id: string) {
  const data = useFetch(`/api/${id}`); // WRONG
  return data;
}
```

```tsx
// GOOD — called from a component or custom Hook
function useFormattedData(id: string) {
  const data = useFetch(`/api/${id}`);
  return data;
}
```

---

### hooks-no-direct-call

**One-liner:** Never call component functions directly as functions — always render them via JSX.

**When to apply:** When you are tempted to call `MyComponent({ prop })` instead of `<MyComponent prop={...} />`.

```tsx
// BAD — bypasses React's rendering system; Hooks inside won't work
const result = MyComponent({ userId: '123' });
```

```tsx
// GOOD
const result = <MyComponent userId="123" />;
```

**Why:** Calling a component directly means React never registers it in the component tree. Its Hooks run in the parent component's context, breaking Hook order and state isolation.

---

### hooks-no-pass

**One-liner:** Never pass a Hook as a value, prop, or argument — call it directly in the component.

**When to apply:** When you want to make a Hook "configurable" by passing it as a parameter.

```tsx
// BAD — Hook identity changes every render; cannot be tracked
function useCustomBehavior(hookFn) {
  const value = hookFn(); // WRONG
}
```

```tsx
// GOOD — accept configuration, not the hook itself
function useCustomBehavior(config: { url: string }) {
  const data = useFetch(config.url);
  return data;
}
```

---

### hooks-immutable-returns

**One-liner:** Never mutate props, state, or the values returned by Hooks — treat them all as read-only.

**When to apply:** Any time you receive state, context, or Hook return values and are about to modify them in-place.

```tsx
// BAD — mutating state directly skips React's re-render cycle
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // WRONG — React doesn't see this change

// BAD — mutating a prop object
function Component({ config }) {
  config.theme = 'dark'; // WRONG — mutates parent's data
}
```

```tsx
// GOOD — create new values, use the setter
const [items, setItems] = useState([1, 2, 3]);
setItems([...items, 4]); // creates a new array

// GOOD — derive new object from prop
function Component({ config }) {
  const localConfig = { ...config, theme: 'dark' };
}
```
