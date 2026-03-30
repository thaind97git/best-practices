---
name: typescript-type-system
description: TypeScript type system fundamentals — primitive types, any/unknown/never, interfaces vs type aliases, union/intersection types, and type safety rules. Apply when writing or reviewing any TypeScript type declarations, variable annotations, or API contracts.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
  version: "1.0"
---

# Type System Fundamentals (CRITICAL)

Core rules for choosing and using TypeScript types correctly. Violations lead to type unsafety, runtime errors, and misleading type signatures.

---

### ts-lowercase-primitives

**One-liner:** Always use lowercase primitive types — `string`, `number`, `boolean`, `symbol`, `object`.

**When to apply:** Any time you annotate a variable, parameter, return type, or property with a primitive type.

```typescript
// BAD — boxed object types, almost never the right choice
function reverse(s: String): String { ... }
function count(n: Number): Boolean { ... }
```

```typescript
// GOOD — lowercase primitive types
function reverse(s: string): string { ... }
function count(n: number): boolean { ... }
```

**Why:** `String`, `Number`, `Boolean`, `Symbol`, and `Object` are boxed JavaScript object types, not primitives. They behave differently from their lowercase counterparts and are almost never appropriate in TypeScript annotations.

---

### ts-no-any

**One-liner:** Never use `any` unless migrating a JavaScript project.

**When to apply:** Any time you reach for `any` to silence a type error or accept unknown input.

```typescript
// BAD — turns off type checking entirely
function process(value: any): any {
  return value.doSomething(); // no type safety
}
```

```typescript
// GOOD — use unknown for truly unknown input
function process(value: unknown): void {
  if (typeof value === 'string') {
    console.log(value.toUpperCase()); // safe
  }
}
```

**Why:** Using `any` is equivalent to placing `@ts-ignore` around every usage — it turns off type checking entirely, propagating unsafety throughout the codebase.

---

### ts-unknown-over-any

**One-liner:** Use `unknown` when you must accept a value but won't interact with it directly.

**When to apply:** When writing utilities, error handlers, or generic adapters that accept untyped input.

```typescript
// BAD — any propagates unsafety to callers
function serialize(value: any): string {
  return JSON.stringify(value);
}

// GOOD — unknown forces callers to narrow before use
function logError(error: unknown): void {
  if (error instanceof Error) {
    console.error(error.message);
  } else {
    console.error(String(error));
  }
}
```

**Why:** `unknown` is the type-safe counterpart of `any`. It accepts any value but forces the caller to prove what the value is before using it.

---

### ts-never-exhaustive

**One-liner:** Use `never` with `assertNever()` to ensure exhaustive handling of all union members.

**When to apply:** In `switch` or `if-else` chains over discriminated unions where all cases must be handled.

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function assertNever(x: never): never {
  throw new Error("Unexpected value: " + String(x));
}

// GOOD — compile error if a new Shape member is added and not handled
function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.side ** 2;
    default: return assertNever(shape); // Error if case is missed
  }
}
```

**Why:** The `never` type is assignable to every type but nothing is assignable to `never`. Passing an unhandled union member to `assertNever` causes a compile-time error.

---

### ts-interface-for-public-api

**One-liner:** Prefer `interface` for public API shapes that may need to be extended.

**When to apply:** When defining the contract of a public API, library export, or data model that consumers might extend.

```typescript
// GOOD — interfaces support declaration merging and are more extensible
interface User {
  id: string;
  name: string;
}

interface AdminUser extends User {
  permissions: string[];
}

// Also GOOD — module augmentation works with interfaces
interface Window {
  myCustomProperty: string;
}
```

**Why:** Interfaces support declaration merging (re-declaring extends the type), making them better suited for public contracts. `interface extends` is also more compiler-performant than `&` intersections.

---

### ts-type-for-computed

**One-liner:** Use `type` for unions, intersections, computed types, and non-extendable shapes.

**When to apply:** When creating union types, mapped types, conditional types, template literal types, or aliases for primitive unions.

```typescript
// GOOD — type for unions
type ID = string | number;

// GOOD — type for computed/conditional expressions
type Result<T> = T extends Promise<infer U> ? U : T;

// GOOD — type for mapped transformations
type Optional<T> = { [K in keyof T]?: T[K] };

// GOOD — type for template literals
type EventKey = `on${Capitalize<string>}`;
```

**Why:** `type` is more expressive for computed type expressions. `interface` cannot represent union types or conditional types.

---

### ts-interface-extends-perf

**One-liner:** Prefer `interface extends` over `&` intersection for combining object shapes.

**When to apply:** When you need to merge or combine two object type shapes.

```typescript
// LESS PREFERRED — intersection with &
type Combined = TypeA & TypeB;

// PREFERRED — interface extends is more compiler-performant
interface Combined extends TypeA, TypeB {}
```

**Why:** TypeScript's compiler caches interface relationships better than intersection types, leading to faster type checking in large codebases. Extending an interface is also more explicit and readable.

---

### ts-no-union-overloads

**One-liner:** Use union types instead of multiple overloads that differ only in a single argument type.

**When to apply:** When you have two or more overloads where the only difference is the type of one parameter.

```typescript
// BAD — unnecessary overloads
function format(x: number): string;
function format(x: string): string;
function format(x: number | string): string { ... }

// GOOD — union type
function format(x: number | string): string { ... }
```

**Why:** Multiple overloads differing by single argument type add noise. Union types are simpler and equally expressive.
