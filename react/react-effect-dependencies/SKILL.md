---
name: react-effect-dependencies
description: React Effect dependency rules — 7 rules for keeping dependency arrays correct and minimal. Apply when writing or reviewing useEffect dependency lists to prevent stale closures, infinite loops, and unnecessary re-runs.
metadata:
  source: https://react.dev/learn/removing-effect-dependencies
  version: "1.0"
---

# Effect Dependencies (HIGH)

7 rules for managing `useEffect` dependency arrays. Incorrect deps cause the two most common Effect bugs: stale closures (missing deps) and infinite re-run loops (unnecessary deps).

---

### deps-exhaustive

**One-liner:** Every reactive value read inside an Effect — props, state, in-component variables — must be listed in the dependency array.

**When to apply:** When writing or reviewing any `useEffect` call. The `react-hooks/exhaustive-deps` ESLint rule enforces this automatically.

```tsx
// BAD — roomId changes are ignored; connection is never re-established
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
}, []); // roomId is missing
```

```tsx
// GOOD
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
```

---

### deps-no-suppress

**One-liner:** Never suppress the `react-hooks/exhaustive-deps` ESLint rule with an inline disable comment.

**When to apply:** Any time you are tempted to write `// eslint-disable-next-line react-hooks/exhaustive-deps`.

```tsx
// BAD — suppression hides a stale closure bug
useEffect(() => {
  fetchData(userId); // userId change is never reacted to
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

```tsx
// GOOD — fix the code so the dep is either listed or provably unnecessary
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

**Why:** The linter is correct. Suppressing it trades a lint warning for a latent, hard-to-reproduce bug.

---

### deps-change-code

**One-liner:** To remove a dependency, change the code so the value is no longer reactive — do not just delete it from the array.

**When to apply:** When an Effect has a dependency that causes unwanted re-runs.

```tsx
// BAD — manually removing a dep the code still reads
useEffect(() => {
  const url = `${serverUrl}/rooms/${roomId}`;
  connect(url);
}, [serverUrl]); // roomId silently missing
```

```tsx
// GOOD option 1: move non-reactive values outside the component
const SERVER_URL = 'https://localhost:1234'; // not reactive

useEffect(() => {
  const connection = createConnection(SERVER_URL, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]); // SERVER_URL is not reactive — correct to omit
```

```tsx
// GOOD option 2: move the object/computation inside the Effect
useEffect(() => {
  const options = { serverUrl, roomId }; // created inside — not a dep
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
```

---

### deps-updater-fn

**One-liner:** Use an updater function (`setState(prev => ...)`) inside Effects to read previous state without adding state to the dependency array.

**When to apply:** When an Effect needs to append to or increment a state value, and adding that state as a dependency would cause an infinite loop or unnecessary reconnects.

```tsx
// BAD — messages in deps causes Effect to re-run (and reconnect) on every message
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.on('message', (msg: string) => {
    setMessages([...messages, msg]); // reads messages
  });
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId, messages]); // messages causes reconnect on every msg
```

```tsx
// GOOD — updater function reads previous state; messages not in deps
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.on('message', (msg: string) => {
    setMessages(prev => [...prev, msg]); // no external read
  });
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
```

---

### deps-extract-primitives

**One-liner:** Destructure object props to primitive values before using them as Effect dependencies.

**When to apply:** When a component receives an object prop and uses its fields inside an Effect.

```tsx
// BAD — options is a new object on every parent render, causing constant reconnects
function ChatRoom({ options }: { options: { serverUrl: string; roomId: string } }) {
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // new object reference every render
}
```

```tsx
// GOOD — primitives only change when their actual values change
function ChatRoom({ options }: { options: { serverUrl: string; roomId: string } }) {
  const { serverUrl, roomId } = options;
  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);
}
```

---

### deps-objects-inside

**One-liner:** Create objects and functions needed by an Effect inside the Effect body, not outside it.

**When to apply:** When an Effect needs a config object or callback that would be re-created on every render if defined outside.

```tsx
// BAD — options is re-created on every render, causing constant reconnects
function ChatRoom({ roomId }: { roomId: string }) {
  const options = { serverUrl: 'https://example.com', roomId };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // new reference every render
}
```

```tsx
// GOOD — defined inside the Effect; not a dependency
function ChatRoom({ roomId }: { roomId: string }) {
  useEffect(() => {
    const options = { serverUrl: 'https://example.com', roomId };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // only roomId matters
}
```

---

### deps-split-effects

**One-liner:** If an Effect handles two independent concerns, split it into two separate Effects.

**When to apply:** When a single Effect has multiple unrelated dependencies that cause each other to re-run unnecessarily.

```tsx
// BAD — changing city re-fetches countries, even though they're independent
useEffect(() => {
  fetchCountries();
  if (city) fetchAreas(city);
}, [city]); // city change unnecessarily re-fetches countries
```

```tsx
// GOOD — each Effect handles one concern
useEffect(() => {
  fetchCountries();
}, []); // runs once

useEffect(() => {
  if (city) fetchAreas(city);
}, [city]); // runs when city changes
```
