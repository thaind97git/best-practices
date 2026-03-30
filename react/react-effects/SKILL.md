---
name: react-effects
description: React Effects — 7 rules for knowing when to use and when not to use useEffect. Apply when writing data-fetching logic, subscriptions, or any code that runs after render.
metadata:
  source: https://react.dev/learn/synchronizing-with-effects
  version: "1.0"
---

# Effects — When to Use (HIGH)

7 rules for correctly scoping `useEffect`. The most common React performance and correctness bugs stem from Effects used where event handlers or derived values should be used instead.

---

### effect-external-only

**One-liner:** Use Effects only to synchronize React components with external systems (browser APIs, WebSockets, third-party libraries, network).

**When to apply:** Before writing any `useEffect`, ask: "Is this synchronizing with something outside React?" If not, it likely belongs elsewhere.

| Scenario | Wrong | Correct |
|---|---|---|
| Derive value from state/props | `useEffect(() => setFull(...))` | Compute inline during render |
| Handle user button click | `useEffect(() => if (clicked) post()` | Move to event handler |
| Subscribe to browser events | — | `useEffect` with cleanup ✓ |
| Connect to WebSocket | — | `useEffect` with cleanup ✓ |
| Fetch data from an API | — | `useEffect` with ignore flag ✓ |

---

### effect-cleanup

**One-liner:** Always return a cleanup function from Effects that start a connection, subscription, or timer.

**When to apply:** Any Effect that calls `addEventListener`, `connect()`, `subscribe()`, `setInterval()`, or starts a fetch.

```tsx
// BAD — no cleanup; listeners stack up on every re-render
useEffect(() => {
  window.addEventListener('resize', handleResize);
}, []);
```

```tsx
// GOOD — cleanup removes the listener
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

```tsx
// GOOD — WebSocket with cleanup
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
```

**Why:** React intentionally mounts → unmounts → remounts components in Strict Mode to surface missing cleanups. In production, cleanups prevent memory leaks when components unmount or dependencies change.

---

### effect-no-derived-state

**One-liner:** Do not use an Effect to compute a derived value and `setState` with it — compute it inline during render.

**When to apply:** When you see a pattern of `useEffect(() => setState(transform(prop)), [prop])`.

```tsx
// BAD — roundtrip: render → Effect → re-render
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

```tsx
// GOOD — computed directly, no extra render
const fullName = `${firstName} ${lastName}`;
```

```tsx
// BAD — expensive filter in Effect
const [visibleTodos, setVisibleTodos] = useState(todos);
useEffect(() => {
  setVisibleTodos(todos.filter(t => t.text.includes(filter)));
}, [todos, filter]);
```

```tsx
// GOOD — useMemo for expensive derivations
const visibleTodos = useMemo(
  () => todos.filter(t => t.text.includes(filter)),
  [todos, filter]
);
```

---

### effect-no-event-post

**One-liner:** Do not use an Effect to react to a user action — handle it directly in the event handler.

**When to apply:** When you see `useEffect` guarded by a flag set from a user action (e.g., `submitted`, `clicked`).

```tsx
// BAD — Effect watches a submitted flag to call an API
const [submitted, setSubmitted] = useState(false);
useEffect(() => {
  if (submitted) {
    post('/api/register', formData);
    showNotification('Registered!');
  }
}, [submitted]);
```

```tsx
// GOOD — API call belongs in the event handler
function handleSubmit() {
  post('/api/register', formData);
  showNotification('Registered!');
}
```

**Why:** Event handlers run once per user action. Effects run on every relevant dependency change — a prop or state changing later could re-trigger the "user action" unintentionally.

---

### effect-fetch-ignore

**One-liner:** Always guard `setState` calls in fetch Effects with an `ignore` flag or `AbortController` to prevent race conditions.

**When to apply:** Any `useEffect` that calls `fetch` or another async data source.

```tsx
// BAD — if userId changes quickly, an old response can overwrite the new one
useEffect(() => {
  fetchUser(userId).then(user => setUser(user));
}, [userId]);
```

```tsx
// GOOD — ignore flag prevents stale responses
useEffect(() => {
  let ignore = false;
  fetchUser(userId).then(user => {
    if (!ignore) setUser(user);
  });
  return () => { ignore = true; };
}, [userId]);
```

```tsx
// GOOD — AbortController alternative
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setUser)
    .catch(() => {}); // ignore AbortError
  return () => controller.abort();
}, [userId]);
```

---

### effect-no-chain

**One-liner:** Do not chain multiple Effects that set state based on each other — consolidate the logic into a single event handler or derived computation.

**When to apply:** When you see two or more Effects where one reacts to state set by another.

```tsx
// BAD — three renders, messy flow, hard to debug
const [card, setCard] = useState(null);
const [goldCard, setGoldCard] = useState(null);
const [round, setRound] = useState(1);

useEffect(() => { setGoldCard(card); }, [card]);
useEffect(() => { setRound(r => r + 1); }, [card]);
```

```tsx
// GOOD — compute everything in the event handler
function handleCardClick(nextCard: Card) {
  setCard(nextCard);
  setGoldCard(nextCard);
  setRound(r => r + 1);
}
```

---

### effect-app-init

**One-liner:** Use module-level code or a one-time guard (`if (!initialized)`) for logic that must run exactly once when the app starts.

**When to apply:** When you need to initialize analytics, feature flags, or auth at app boot.

```tsx
// BAD — runs twice in Strict Mode; hard to control
useEffect(() => {
  loadDatastore();
  loadAuthUser();
}, []);
```

```tsx
// GOOD — module-level: runs once when the module is first imported
if (typeof window !== 'undefined') {
  loadDatastore();
}

// OR: component-level guard
let didInit = false;
function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      loadDatastore();
      loadAuthUser();
    }
  }, []);
}
```
