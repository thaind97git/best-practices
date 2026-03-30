---
name: react-state-preservation
description: React state preservation and reset rules — 5 rules governing when React preserves or destroys component state based on tree position, keys, and component identity. Apply when debugging unexpected state resets or stale state issues.
metadata:
  source: https://react.dev/learn/preserving-and-resetting-state
  version: "1.0"
---

# State Preservation & Reset (MEDIUM)

5 rules about how React decides whether to preserve or destroy state between renders. Understanding these prevents bugs where state survives when it should reset — or resets when it should persist.

---

### preserve-position

**One-liner:** State is tied to a component's position in the render tree, not to the JSX variable name or tag.

**When to apply:** When reasoning about whether state will be shared or reset across conditional renders or siblings.

```tsx
// Both branches render <Counter /> at the same tree position (index 0)
// React sees "same component type, same position" → state IS preserved
{isFancy ? <Counter isFancy={true} /> : <Counter isFancy={false} />}
```

```tsx
// Different component types at the same position → state IS reset
{isPaused ? <p>Take a break!</p> : <Counter />}
// When switching from <p> to <Counter>, Counter starts fresh
```

**Key insight:** It is the **position in the tree** that matters, not the JSX expression itself.

---

### preserve-no-nested-defs

**One-liner:** Never define a component function inside another component function.

**When to apply:** Any time you write `function Inner()` or `const Inner = () =>` inside a parent component's function body.

```tsx
// BAD — MyTextField is re-created as a new function reference every render
// React sees a "different component type" every render → state resets on every keystroke
export default function MyForm() {
  function MyTextField() {
    const [text, setText] = useState('');
    return <input value={text} onChange={e => setText(e.target.value)} />;
  }
  return <MyTextField />;
}
```

```tsx
// GOOD — defined at module level; stable identity across renders
function MyTextField() {
  const [text, setText] = useState('');
  return <input value={text} onChange={e => setText(e.target.value)} />;
}

export default function MyForm() {
  return <MyTextField />;
}
```

**Why:** A nested component definition creates a new function object on every parent render. React compares component types by reference — a new function = a different type = full state destruction.

---

### preserve-stable-keys

**One-liner:** Always use stable, unique identifiers as `key` — never use array indices.

**When to apply:** Any time you render a list with `.map()`.

```tsx
// BAD — index-based key: inserting/removing items shifts indices
// React mismatches old and new list items → wrong state associations
{todos.map((todo, index) => <TodoItem key={index} todo={todo} />)}
```

```tsx
// GOOD — stable ID: React correctly tracks each item across reorders
{todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
```

**Why:** When items reorder, the index changes but the `id` doesn't. Using indices causes inputs, checkboxes, and other stateful children to display data for the wrong item.

---

### preserve-key-reset

**One-liner:** Use the `key` prop to deliberately reset a component's state when the key changes.

**When to apply:** When switching between "instances" of the same component type (e.g., different chat rooms, different profiles) and you want each to start with fresh state.

```tsx
// BAD — Chat preserves its draft message when switching contacts
function Messenger({ contacts, selectedContact }) {
  return <Chat contact={selectedContact} />;
}
```

```tsx
// GOOD — key change forces React to unmount and remount with fresh state
function Messenger({ contacts, selectedContact }) {
  return <Chat key={selectedContact.id} contact={selectedContact} />;
}
```

**Tip:** This is the preferred approach over using `useEffect` to reset state when a prop changes.

---

### preserve-consistent-tree

**One-liner:** Keep the component tree structure consistent across conditional branches to avoid unintended state resets.

**When to apply:** When conditionally rendering wrapper elements around a shared component.

```tsx
// BAD — Form is at position 1 when hint is shown, position 0 when not shown
// → state resets whenever showHint toggles
function MyPage({ showHint }) {
  if (showHint) {
    return (
      <div>
        <p className="hint">Hint: ...</p>
        <Form />
      </div>
    );
  }
  return (
    <div>
      <Form />
    </div>
  );
}
```

```tsx
// GOOD — Form is always at the same position; hint is conditional inline
function MyPage({ showHint }) {
  return (
    <div>
      {showHint && <p className="hint">Hint: ...</p>}
      <Form />
    </div>
  );
}
```
