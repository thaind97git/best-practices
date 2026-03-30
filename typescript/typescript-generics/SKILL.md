---
name: typescript-generics
description: TypeScript generics best practices — type parameter naming, constraints, unused parameters, keyof, and readability guidelines. Apply when writing generic functions, classes, or interfaces.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/generics.html
  version: "1.0"
---

# Generics (HIGH)

Rules for writing clear, correct, and minimal generics. Overusing or misusing generics makes code harder to read without adding type safety.

---

### gen-no-unused-params

**One-liner:** Don't declare generic type parameters that are never used in the function body or return type.

**When to apply:** When reviewing any generic function, interface, or class definition.

```typescript
// BAD — T is declared but never used
declare function create<T extends object>(): void;
function noop<T>(x: T): void { }

// GOOD — remove unused type parameters
declare function create(): void;
function noop(x: unknown): void { }
```

**Why:** An unused type parameter adds confusion without adding type safety. It forces callers to provide or infer a type parameter that serves no purpose.

---

### gen-identity-pattern

**One-liner:** A type parameter should appear in at least two places — as input and output — to provide value.

**When to apply:** When adding a type parameter to a function or type.

```typescript
// BAD — T only appears once (as output); it's unconstrained and arbitrary
function first<T>(arr: any[]): T { return arr[0]; }

// GOOD — T appears as input AND output, creating a meaningful constraint
function first<T>(arr: T[]): T { return arr[0]; }

// GOOD — T constrains both parameter types
function identity<T>(x: T): T { return x; }
```

**Why:** If a type parameter only appears once, it's not doing anything useful — it's just adding noise. The value of generics comes from linking types together.

---

### gen-keyof-constraint

**One-liner:** Use `K extends keyof T` to safely constrain key access to valid property names.

**When to apply:** When writing a function that accesses a property of an object by key.

```typescript
// BAD — no constraint, key could be anything
function getProp(obj: any, key: string): any {
  return obj[key];
}

// GOOD — key must be a valid property of T
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
getProperty(user, "name"); // OK — returns string
getProperty(user, "age");  // OK — returns number
getProperty(user, "foo");  // Error — "foo" is not keyof typeof user
```

**Why:** `K extends keyof T` ensures the key exists on the object type and infers the exact return type `T[K]`, providing full type safety for property access patterns.

---

### gen-descriptive-names

**One-liner:** Use descriptive type parameter names for complex generics; single letters are fine for simple ones.

**When to apply:** When writing generics with multiple type parameters or when the parameter's role isn't obvious.

```typescript
// GOOD — single letter for simple, standard generic
function identity<T>(x: T): T { return x; }
function map<T, U>(arr: T[], fn: (x: T) => U): U[] { ... }

// GOOD — descriptive names for clarity when intent is non-obvious
function createStore<TState, TAction>(
  reducer: (state: TState, action: TAction) => TState,
  initialState: TState
): Store<TState, TAction> { ... }
```

**Why:** Descriptive names make complex generic code readable. Single letters `T`, `K`, `V`, `U` are well-established conventions for simple cases.

---

### gen-fewer-params

**One-liner:** Use the fewest type parameters that correctly describe the relationship between types.

**When to apply:** When designing a generic API — start with the minimum number of type parameters and add only when necessary.

```typescript
// OVERLY COMPLEX — TInput and TOutput serve no purpose here
function process<TInput, TOutput>(values: TInput[]): TOutput[] { ... }

// SIMPLER — T and U are sufficient
function map<T, U>(values: T[], transform: (x: T) => U): U[] { ... }
```

**Why:** Each additional type parameter increases cognitive load. Fewer parameters make generics easier to understand, infer, and call.

---

### gen-prefer-constraints-over-any

**One-liner:** Use generic constraints instead of `any` when you need flexibility but also want type safety.

**When to apply:** When a function needs to accept multiple types but requires that they have certain properties.

```typescript
// BAD — loses type safety
function getLength(x: any): number {
  return x.length;
}

// GOOD — constrained generic preserves type info
interface HasLength { length: number; }
function getLength<T extends HasLength>(x: T): number {
  return x.length;
}

getLength("hello");    // OK — string has .length
getLength([1, 2, 3]);  // OK — array has .length
getLength(42);         // Error — number doesn't have .length
```

---

### gen-infer-over-explicit

**One-liner:** Let TypeScript infer type arguments when calling generic functions — only specify them explicitly when needed.

**When to apply:** When calling generic functions in application code.

```typescript
// OVERLY EXPLICIT — TypeScript can infer this
const result = identity<string>("hello");

// PREFERRED — let TypeScript infer
const result = identity("hello"); // TypeScript infers T = string

// Explicit annotation IS needed here (return type can't be inferred)
const arr = [] as string[];
```

**Why:** Explicit type arguments add noise and can make code brittle if types change. TypeScript's inference is usually correct.
