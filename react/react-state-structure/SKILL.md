---
name: react-state-structure
description: React state structure rules — 6 rules for designing minimal, non-redundant, non-contradictory state. Apply when adding new state variables or reviewing existing state design.
metadata:
  source: https://react.dev/learn/choosing-the-state-structure
  version: "1.0"
---

# State Structure (HIGH)

6 rules for designing clean state. Poor state structure is the root cause of bugs where the UI shows stale, duplicated, or contradictory data.

---

### state-group-related

**One-liner:** Group state variables that always change together into a single object.

**When to apply:** When you find yourself calling multiple `setState` calls back-to-back for the same user action (e.g., updating x and y coordinates together).

```tsx
// BAD — two separate calls; easy to forget one
const [x, setX] = useState(0);
const [y, setY] = useState(0);

function handleMove(e: React.MouseEvent) {
  setX(e.clientX);
  setY(e.clientY); // must remember both
}
```

```tsx
// GOOD — grouped; always updated atomically
const [position, setPosition] = useState({ x: 0, y: 0 });

function handleMove(e: React.MouseEvent) {
  setPosition({ x: e.clientX, y: e.clientY });
}
```

**Note:** If the number of fields is dynamic (e.g., form with arbitrary fields), an object is even more important.

---

### state-no-contradiction

**One-liner:** Replace multiple boolean flags that can be `true` simultaneously (but shouldn't be) with a single status string.

**When to apply:** When you have two or more booleans representing phases of a process (loading, success, error states).

```tsx
// BAD — both isSending and isSent could be true at the same time
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);
```

```tsx
// GOOD — mutually exclusive states expressed as a union type
type Status = 'typing' | 'sending' | 'sent';
const [status, setStatus] = useState<Status>('typing');

const isSending = status === 'sending';
const isSent = status === 'sent';
```

---

### state-no-redundant

**One-liner:** Never store a value in state if it can be computed from existing state or props during render.

**When to apply:** When reviewing state variables — ask "can I compute this instead?"

```tsx
// BAD — fullName is always derivable; storing it causes double renders
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

```tsx
// GOOD — computed inline; no extra state or Effect needed
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = `${firstName} ${lastName}`;
```

```tsx
// GOOD — use useMemo only when the computation is genuinely expensive
const visibleItems = useMemo(
  () => items.filter(item => item.category === selectedCategory),
  [items, selectedCategory]
);
```

---

### state-no-duplication

**One-liner:** Store IDs (or minimal keys) instead of full objects to avoid stale duplicates when the source data changes.

**When to apply:** When state stores an item that also exists in another state array or list.

```tsx
// BAD — selectedItem duplicates data from items; can become stale
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState<Item>(items[0]);

function handleRename(item: Item, name: string) {
  setItems(items.map(i => i.id === item.id ? { ...i, name } : i));
  // selectedItem still has the old name — bug!
}
```

```tsx
// GOOD — store only the ID; derive the object during render
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState<number>(items[0].id);
const selectedItem = items.find(item => item.id === selectedId);
```

---

### state-no-deep-nesting

**One-liner:** Flatten deeply nested state structures — normalize them like a database table keyed by ID.

**When to apply:** When state represents a tree or hierarchy that needs to be mutated at arbitrary depths.

```tsx
// BAD — updating a deeply nested node requires spreading every level
const [tree, setTree] = useState({
  id: 0,
  children: [{ id: 1, children: [{ id: 2, children: [] }] }]
});
```

```tsx
// GOOD — flat map keyed by ID; each node stores only child IDs
type Node = { id: number; childIds: number[] };
const [nodes, setNodes] = useState<Record<number, Node>>({
  0: { id: 0, childIds: [1] },
  1: { id: 1, childIds: [2] },
  2: { id: 2, childIds: [] },
});

function removeNode(id: number) {
  setNodes(prev => {
    const next = { ...prev };
    delete next[id];
    return next;
  });
}
```

---

### state-no-mirror-props

**One-liner:** Do not copy a prop into state — use the prop directly in render. If initialization is intentional, use the `initial` prefix convention.

**When to apply:** When a component receives a prop and immediately puts it in `useState` with the same name.

```tsx
// BAD — messageColor prop changes are ignored after first render
function Message({ messageColor }: { messageColor: string }) {
  const [color, setColor] = useState(messageColor); // ignores updates
  return <p style={{ color }}>{/* ... */}</p>;
}
```

```tsx
// GOOD — use the prop directly if updates should be reflected
function Message({ messageColor }: { messageColor: string }) {
  return <p style={{ color: messageColor }}>{/* ... */}</p>;
}
```

```tsx
// ACCEPTABLE — initialColor naming makes the intent explicit: only first render matters
function ColorPicker({ initialColor }: { initialColor: string }) {
  const [color, setColor] = useState(initialColor);
  return <input value={color} onChange={e => setColor(e.target.value)} />;
}
```
