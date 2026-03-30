---
name: react-refs
description: React refs rules — 4 rules for correctly using useRef for non-rendering data and DOM access. Apply when deciding between state and a ref, or when accessing DOM nodes imperatively.
metadata:
  source: https://react.dev/learn/referencing-values-with-refs
  version: "1.0"
---

# Refs (MEDIUM)

4 rules for `useRef`. Refs are an escape hatch — use them when you need a mutable container that does not affect rendering.

---

### ref-non-rendering

**One-liner:** Use refs to store data that must persist across renders but does NOT need to trigger a re-render when it changes.

**When to apply:** When storing timeout IDs, interval IDs, previous values, animation frame handles, or any "bookkeeping" value that the UI does not display.

```tsx
// GOOD — timeout ID stored in a ref; clearing it does not need to re-render
function DelayedAlert() {
  const timeoutRef = useRef<ReturnType<typeof setTimeout> | null>(null);

  function handleSend() {
    timeoutRef.current = setTimeout(() => alert('Sent!'), 3000);
  }

  function handleUndo() {
    clearTimeout(timeoutRef.current!);
  }

  return (
    <>
      <button onClick={handleSend}>Send</button>
      <button onClick={handleUndo}>Undo</button>
    </>
  );
}
```

---

### ref-state-for-ui

**One-liner:** Use state (not a ref) for any value that is displayed in the rendered output.

**When to apply:** When you find yourself writing `ref.current` inside JSX.

```tsx
// BAD — UI never updates because ref change doesn't trigger re-render
function Counter() {
  const countRef = useRef(0);

  function handleClick() {
    countRef.current++;
  }

  return <button onClick={handleClick}>Clicked {countRef.current} times</button>;
  //                                                    ^^^^^^^^^^^^^^ always shows 0
}
```

```tsx
// GOOD — state change triggers re-render; UI stays in sync
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Clicked {count} times
    </button>
  );
}
```

---

### ref-no-render-read

**One-liner:** Do not read or write `ref.current` during the component's render phase.

**When to apply:** Any time you see `ref.current` used at the top level of a component function body (not inside an event handler, Effect, or callback).

```tsx
// BAD — reading a ref during render is unpredictable in concurrent mode
function MyComponent({ inputRef }: { inputRef: React.RefObject<HTMLInputElement> }) {
  const isVisible = inputRef.current !== null; // WRONG — may not be set yet
  return <div>{isVisible ? 'visible' : 'hidden'}</div>;
}
```

```tsx
// GOOD — read the ref inside an effect or event handler
function MyComponent({ inputRef }: { inputRef: React.RefObject<HTMLInputElement> }) {
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    setIsVisible(inputRef.current !== null);
  }, []);

  return <div>{isVisible ? 'visible' : 'hidden'}</div>;
}
```

**Why:** During rendering React may call the function multiple times (Strict Mode, concurrent rendering). `ref.current` may be `null` on the first call and set on the second, making renders inconsistent.

---

### ref-dom-access

**One-liner:** Use refs attached to JSX elements to access DOM nodes imperatively for focus, scroll, measurement, or integration with third-party libraries.

**When to apply:** When you need to programmatically focus an input, scroll a container, measure an element's size, or integrate a non-React library.

```tsx
// GOOD — focus on mount
function SearchInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="search" />;
}
```

```tsx
// GOOD — scroll to bottom of a chat list
function ChatList({ messages }: { messages: string[] }) {
  const bottomRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  return (
    <div className="chat-list">
      {messages.map((msg, i) => <p key={i}>{msg}</p>)}
      <div ref={bottomRef} />
    </div>
  );
}
```

**Ref vs state quick reference:**

| | `useRef` | `useState` |
|---|---|---|
| Triggers re-render | No | Yes |
| Mutable directly | `ref.current = value` | Use setter only |
| Safe to read during render | No | Yes |
| Primary use case | Non-rendering data, DOM nodes | Anything shown in UI |
