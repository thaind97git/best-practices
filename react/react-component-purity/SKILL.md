---
name: react-component-purity
description: React component purity rules — 5 rules ensuring components are pure functions that return predictable output. Apply when writing or reviewing any React component render logic.
metadata:
  source: https://react.dev/learn/keeping-components-pure
  version: "1.0"
---

# Component Purity (CRITICAL)

5 rules that ensure React components behave as pure functions. Required for SSR, concurrent rendering, and React's optimization passes (memoization, double-invoke in Strict Mode) to work correctly.

---

### purity-same-output

**One-liner:** Components must return the same JSX for the same props, state, and context.

**When to apply:** When reviewing component render logic — if the output varies based on anything other than props/state/context, it is impure.

```tsx
// BAD — output depends on external mutable variable
let guestCount = 0;
function GuestBadge() {
  guestCount++; // impure: mutates outer scope
  return <p>Guest #{guestCount}</p>;
}
```

```tsx
// GOOD — output depends only on props
function GuestBadge({ guestNumber }: { guestNumber: number }) {
  return <p>Guest #{guestNumber}</p>;
}
```

**Why:** React may render a component multiple times (Strict Mode, concurrent features). Impure components produce different output on each call, causing visual glitches and broken memoization.

---

### purity-no-render-side-effects

**One-liner:** Never produce side effects (network requests, timers, DOM mutations, logging to external systems) directly in the component body.

**When to apply:** Any time code that "does something" (not just computes a value) appears at the top level of a component function.

```tsx
// BAD — side effect during render
function ProductPage({ productId }: { productId: string }) {
  fetch(`/api/products/${productId}`); // fires on every render — WRONG
  return <div>...</div>;
}
```

```tsx
// GOOD — side effects belong in event handlers or Effects
function ProductPage({ productId }: { productId: string }) {
  const { data } = useFetch(`/api/products/${productId}`); // handled by hook
  return <div>{data?.name}</div>;
}
```

---

### purity-no-prop-mutation

**One-liner:** Never mutate props, arrays, or objects passed in from outside the component.

**When to apply:** When modifying a list or object received as a prop before rendering it.

```tsx
// BAD — mutates the prop array in-place
function StoryList({ stories }: { stories: Story[] }) {
  stories.push({ id: 'new', label: 'Create Story' }); // mutates caller's data
  return stories.map(s => <Story key={s.id} story={s} />);
}
```

```tsx
// GOOD — copy first, then modify
function StoryList({ stories }: { stories: Story[] }) {
  const displayStories = [...stories, { id: 'new', label: 'Create Story' }];
  return displayStories.map(s => <Story key={s.id} story={s} />);
}
```

---

### purity-local-mutation-ok

**One-liner:** Local mutation — modifying objects or arrays created during the same render — is acceptable.

**When to apply:** When building a result array or object inside the render function before returning it.

```tsx
// GOOD — localItems is created during this render; mutating it is fine
function FilteredList({ items, filter }: { items: Item[]; filter: string }) {
  const localItems: Item[] = [];
  for (const item of items) {
    if (item.name.includes(filter)) {
      localItems.push(item); // local mutation — acceptable
    }
  }
  return <ul>{localItems.map(i => <li key={i.id}>{i.name}</li>)}</ul>;
}
```

**Why:** This local array never escapes the render function, so no external code can observe the mutation. It is functionally equivalent to a pure transformation.

---

### purity-no-direct-dom

**One-liner:** Never read from or write to the DOM directly inside the component render body.

**When to apply:** When you see `document.getElementById`, `document.querySelector`, or direct style/className assignments in the render function body.

```tsx
// BAD — DOM manipulation during render
function Clock({ time }: { time: Date }) {
  document.getElementById('clock')!.className = 'night'; // WRONG
  return <h1 id="clock">{time.toLocaleTimeString()}</h1>;
}
```

```tsx
// GOOD — derive the value and pass it as a prop
function Clock({ time }: { time: Date }) {
  const hours = time.getHours();
  const className = hours >= 0 && hours <= 6 ? 'night' : 'day';
  return <h1 className={className}>{time.toLocaleTimeString()}</h1>;
}
```

**Why:** DOM access during render is a side effect. It breaks SSR (no DOM on the server), concurrent rendering (render may be discarded), and Strict Mode double-invocation.
