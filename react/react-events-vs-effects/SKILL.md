---
name: react-events-vs-effects
description: React events vs effects rules — 4 rules distinguishing when logic belongs in event handlers versus Effects. Apply when deciding where to put side-effect code or when debugging over-triggering Effects.
metadata:
  source: https://react.dev/learn/separating-events-from-effects
  version: "1.0"
---

# Events vs Effects (MEDIUM)

4 rules for separating reactive logic (Effects) from user-interaction logic (event handlers). Mixing these up causes the most common over-triggering and under-triggering bugs in React apps.

---

### evfx-event-handler-first

**One-liner:** Put side effects triggered by a specific user interaction in the event handler — not in an Effect.

**When to apply:** When you are deciding where to place a `fetch`, `navigate`, or mutation call after a button click, form submit, or other explicit user action.

```tsx
// BAD — Effect polls a flag set by the user
const [submitted, setSubmitted] = useState(false);
useEffect(() => {
  if (submitted) {
    postForm(formData);
    showNotification('Submitted!');
  }
}, [submitted, formData]);
```

```tsx
// GOOD — event handler owns the submission side effect
function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
  postForm(formData);
  showNotification('Submitted!');
}
```

**Rule of thumb:** If the question is "why did this run?", the answer for event handlers is always "because the user did X." For Effects it is "because some value changed."

---

### evfx-effect-reactive

**One-liner:** Use Effects for logic that must re-run whenever a reactive value (prop, state) changes — not for one-time user interactions.

**When to apply:** When you need to synchronize something external (connection, subscription, document title) with changing props or state.

```tsx
// GOOD — roomId change is reactive; Effect re-runs when it changes
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]); // reactive to both values
```

```tsx
// BAD — user clicked "connect" once; this should be an event handler, not an Effect
const [shouldConnect, setShouldConnect] = useState(false);
useEffect(() => {
  if (shouldConnect) createConnection(serverUrl, roomId).connect();
}, [shouldConnect, serverUrl, roomId]);
```

---

### evfx-use-effect-event

**One-liner:** Use `useEffectEvent` to read the latest value of a reactive variable inside an Effect without making the Effect re-run when that variable changes.

**When to apply:** When an Effect needs to use a value (like `theme`, a callback, or a flag) that should always be current but should NOT trigger re-synchronization when it changes.

```tsx
// BAD — theme change causes the chat room to disconnect and reconnect
function ChatRoom({ roomId, theme }: { roomId: string; theme: string }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => showNotification('Connected!', theme));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]); // theme in deps → reconnect on theme change
}
```

```tsx
// GOOD — onConnected reads the latest theme without being reactive
function ChatRoom({ roomId, theme }: { roomId: string; theme: string }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme); // always reads current theme
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => onConnected());
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // theme is no longer a dependency
}
```

**Note:** `useEffectEvent` is an experimental API (RFC stage). Check your React version for availability.

---

### evfx-effect-event-rules

**One-liner:** Effect Events must only be called from inside Effects — never stored, passed to children, or called from event handlers.

**When to apply:** When using `useEffectEvent` — these constraints are enforced to preserve the guarantee that the event always reads current values in the right context.

```tsx
// BAD — passed to child component; child calls it outside the Effect
const onConnected = useEffectEvent(() => showNotification(theme));
return <ChatRoom onMessage={onConnected} />; // WRONG
```

```tsx
// BAD — called from an event handler
const onConnected = useEffectEvent(() => showNotification(theme));
function handleClick() {
  onConnected(); // WRONG — not inside an Effect
}
```

```tsx
// GOOD — only called from inside the Effect that owns it
const onConnected = useEffectEvent(() => showNotification(theme));
useEffect(() => {
  connection.on('connected', () => onConnected()); // CORRECT
}, [roomId]);
```
