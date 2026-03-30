---
name: typescript-functions
description: TypeScript function best practices — overloads, callback types, optional parameters, void vs any return types, call signatures, and rest parameters. Apply when writing function signatures, overloads, or callback-accepting APIs.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/functions.html
  version: "1.0"
---

# Functions (HIGH)

Rules for typing functions correctly and avoiding common mistakes in overloads, callback types, and parameter annotations.

---

### fn-void-not-any

**One-liner:** Use `void` — not `any` — as the return type for callbacks whose return value is ignored.

**When to apply:** When writing a higher-order function that accepts a callback and does not use its return value.

```typescript
// BAD — 'any' return type allows callers to accidentally rely on the return value
function forEach(arr: any[], callback: (arg: any) => any): void { ... }

// GOOD — 'void' signals the return value is intentionally ignored
function forEach<T>(arr: T[], callback: (arg: T) => void): void { ... }
```

**Why:** `void` prevents accidental misuse of the return value. A function that returns `void` can still return a value internally, but callers are signaled not to use it.

---

### fn-no-optional-callback-params

**One-liner:** Don't make callback parameters optional unless the callback can truly be called with fewer arguments.

**When to apply:** When defining the type of a callback function that will always be called with a specific number of arguments.

```typescript
// BAD — the 'elapsed' parameter is always provided; making it optional is wrong
interface Timer {
  setTimeout(handler: (elapsed?: number) => void): void;
}

// GOOD — the callback always receives 'elapsed'
interface Timer {
  setTimeout(handler: (elapsed: number) => void): void;
}
```

**Why:** It's always legal to pass a function with fewer parameters than required. Making parameters optional when they're always provided adds unnecessary `undefined` handling for the callback's caller and misleads the type.

---

### fn-specific-overloads-first

**One-liner:** Always put specific overloads before general ones — TypeScript matches the first applicable overload.

**When to apply:** Any time you write function overloads with different levels of specificity.

```typescript
// BAD — the general 'unknown' overload is matched first; 'number' overload is unreachable
declare function fn(x: unknown): unknown;
declare function fn(x: number): number; // never reached

// GOOD — specific overloads come first
declare function fn(x: HTMLDivElement): number;
declare function fn(x: HTMLElement): number;
declare function fn(x: any): any; // most general last
```

**Why:** TypeScript selects the first overload whose parameters match the call. If a general overload appears first, it will shadow all specific overloads below it.

---

### fn-collapse-overloads-union

**One-liner:** Collapse overloads that differ only by a single argument type into one signature with a union type.

**When to apply:** When you have multiple overloads where only one parameter's type varies and the return type is the same.

```typescript
// BAD — unnecessary overloads
function format(x: number): string;
function format(x: string): string;
function format(x: number | string): string { ... }

// GOOD — single signature with union
function format(x: number | string): string { ... }
```

---

### fn-collapse-overloads-optional

**One-liner:** Collapse overloads that differ only in trailing parameters into one signature with optional parameters.

**When to apply:** When all overloads share the same return type and the later overloads just add more trailing parameters.

```typescript
// BAD — multiple overloads for trailing optional params
interface Example {
  diff(one: string): number;
  diff(one: string, two: string): number;
  diff(one: string, two: string, three: boolean): number;
}

// GOOD — use optional parameters
interface Example {
  diff(one: string, two?: string, three?: boolean): number;
}
```

---

### fn-callback-max-arity

**One-liner:** Write one overload with maximum arity instead of multiple overloads differing only in callback parameter count.

**When to apply:** When an API accepts a callback that may or may not receive certain arguments.

```typescript
// BAD — two overloads for callback arity
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;

// GOOD — one overload with maximum arity
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number
): void;
```

**Why:** It's always valid to provide a callback with fewer parameters than declared. The single max-arity overload covers both cases.

---

### fn-call-signature

**One-liner:** Use call signatures on object types to describe callable objects that also have properties.

**When to apply:** When a function object needs additional properties alongside its call behavior.

```typescript
// GOOD — object type with a call signature
interface Callable {
  description: string;
  (arg: number): boolean;
}

// Combining call and construct signatures
interface CallOrConstruct {
  (n?: number): string;
  new (s: string): Date;
}
```

---

### fn-overload-implementation-not-callable

**One-liner:** The implementation signature of an overloaded function is not visible to callers — only the overload signatures are.

**When to apply:** When writing overloaded functions to avoid confusing the implementation signature with a callable overload.

```typescript
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
// The implementation signature below cannot be called directly:
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  }
  return new Date(mOrTimestamp);
}

makeDate(12345678);           // OK — first overload
makeDate(5, 5, 5);            // OK — second overload
makeDate(1, 3);               // Error — no matching overload
```

---

### fn-void-return-design

**One-liner:** Understand that `() => void` does not prevent returning a value — it only signals the return value won't be observed.

**When to apply:** When designing APIs that accept callbacks and need to be compatible with functions that return values.

```typescript
type VoidFunc = () => void;

// All of these are valid assignments to () => void
const f1: VoidFunc = () => true;
const f2: VoidFunc = () => { return true; };

// This allows array forEach to accept any function:
[1, 2, 3].forEach((n) => n * 2); // OK, even though forEach expects () => void
```

**Why:** This design allows functions like `Array.prototype.forEach` to accept callbacks that happen to return values without type errors.

---

### fn-rest-params

**One-liner:** Type rest parameters as `T[]` or `Array<T>` (or a tuple type for heterogeneous rest args).

**When to apply:** When a function accepts a variable number of arguments of the same type.

```typescript
// GOOD
function multiply(n: number, ...nums: number[]): number[] {
  return nums.map(x => x * n);
}

// Typed tuple for heterogeneous args
function log(message: string, ...args: [number, boolean]): void { ... }
```
