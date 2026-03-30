---
name: react-custom-hooks
description: React custom hooks rules — 5 rules for designing, naming, and composing custom hooks. Apply when extracting reusable logic, naming utility functions, or reviewing custom hook APIs.
metadata:
  source: https://react.dev/learn/reusing-logic-with-custom-hooks
  version: "1.0"
---

# Custom Hooks (LOW-MEDIUM)

5 rules for authoring custom Hooks. Custom Hooks are how React encourages sharing stateful logic — not UI, not state instances, just the wiring.

---

### hook-use-prefix

**One-liner:** Custom hook names must start with `use` followed by a capital letter (e.g., `useOnlineStatus`, `useChatRoom`, `useFormInput`).

**When to apply:** Any time you write a function that calls one or more React Hooks.

```tsx
// BAD — looks like a plain function; linter won't enforce Rules of Hooks inside it
function onlineStatus() {
  const [isOnline, setIsOnline] = useState(true); // linter won't check this
  // ...
}
```

```tsx
// GOOD — clear intent; linter enforces Hook rules inside
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

---

### hook-no-use-for-non-hooks

**One-liner:** Do not add the `use` prefix to a function that calls no Hooks — name it as a plain function.

**When to apply:** When extracting a utility function from a component and deciding on a name.

```tsx
// BAD — use prefix implies Hooks are called inside; confuses readers and the linter
function useSortedItems(items: Item[]) {
  return items.slice().sort((a, b) => a.name.localeCompare(b.name));
  // no Hooks called — should not be named useXxx
}
```

```tsx
// GOOD — plain function name for plain logic
function getSortedItems(items: Item[]) {
  return items.slice().sort((a, b) => a.name.localeCompare(b.name));
}
```

**Why:** The `use` prefix is a contract — it signals to React, the ESLint plugin, and human readers that the function follows the Rules of Hooks.

---

### hook-share-logic-not-state

**One-liner:** Each call to a custom Hook creates completely independent state — custom Hooks share the logic (wiring), not the state instance.

**When to apply:** When designing a custom Hook and considering whether two consumers will share or isolate their state.

```tsx
// Both components call useOnlineStatus() — they each get their own isOnline state
// They both subscribe to the same browser events, but independently
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? 'Online' : 'Disconnected'}</p>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  return <button disabled={!isOnline}>Save</button>;
}
// StatusBar and SaveButton do NOT share a single isOnline variable
```

**Implication:** To share state between components, lift it up to a common ancestor and pass it down — don't rely on a custom Hook to provide shared state.

---

### hook-extract-when-repeated

**One-liner:** Extract a custom Hook when you find yourself duplicating the same `useState + useEffect` pattern across multiple components.

**When to apply:** When two or more components have nearly identical Hook combinations.

```tsx
// BEFORE — duplicated in StatusBar and SaveButton
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return <p>{isOnline ? '✅ Online' : '❌ Disconnected'}</p>;
}
```

```tsx
// AFTER — shared logic in a custom Hook
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? '✅ Online' : '❌ Disconnected'}</p>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  return <button disabled={!isOnline}>Save progress</button>;
}
```

**Additional benefit:** You can later swap the implementation (e.g., replace `useEffect` + event listeners with `useSyncExternalStore`) without touching the consuming components.

---

### hook-no-lifecycle-wrapper

**One-liner:** Avoid creating generic lifecycle wrappers like `useMount`, `useUnmount`, or `useUpdateEffect` — they obscure dependencies from the linter and the reader.

**When to apply:** When you are tempted to abstract the lifecycle pattern rather than the domain logic.

```tsx
// BAD — hides what fn reads; linter cannot check deps inside fn
function useMount(fn: () => void) {
  useEffect(() => {
    fn();
  }, []); // fn's dependencies are invisible to exhaustive-deps
}
```

```tsx
// BAD — usage looks clean but silently misses deps
function Component({ userId }: { userId: string }) {
  useMount(() => {
    fetchUser(userId); // userId is not tracked as a dep
  });
}
```

```tsx
// GOOD — explicit Effect with proper deps
function Component({ userId }: { userId: string }) {
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);
}
```

**Good pattern instead:** Extract hooks around domain logic, not lifecycle:

```tsx
// GOOD — hides implementation, exposes intent, deps are traceable
function useData(url: string) {
  const [data, setData] = useState<unknown>(null);
  useEffect(() => {
    if (!url) return;
    let ignore = false;
    fetch(url)
      .then(res => res.json())
      .then(json => { if (!ignore) setData(json); });
    return () => { ignore = true; };
  }, [url]);
  return data;
}
```
