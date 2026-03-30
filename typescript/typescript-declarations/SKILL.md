---
name: typescript-declarations
description: TypeScript declaration file best practices — Do's and Don'ts from the official handbook, overload ordering, collapsing overloads, reference directives, and declaration patterns. Apply when writing .d.ts files or declaration-style interfaces.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
  version: "1.0"
---

# Declaration Files & Do's/Don'ts (HIGH)

Official rules from the TypeScript handbook for writing declaration files and public API types. These rules prevent common mistakes in library definitions and type contracts.

---

## General Types: Complete Rules

| Category | DON'T | DO |
|---|---|---|
| Primitives | `Number`, `String`, `Boolean`, `Symbol`, `Object` | `number`, `string`, `boolean`, `symbol`, `object` |
| Catch-all | `any` | `unknown` (or a specific type) |
| Unused generics | `function f<T>(): void` (T unused) | `function f(): void` |
| Callback return | `(cb: () => any) => void` | `(cb: () => void) => void` |
| Callback optionals | `(cb: (x?: number) => void)` | `(cb: (x: number) => void)` |

---

### decl-use-reference-types

**One-liner:** Use `/// <reference types="..." />` to declare dependencies on type packages.

**When to apply:** At the top of declaration files that depend on types from another package (e.g., `@types/node`).

```typescript
// GOOD — declares dependency on @types/node types
/// <reference types="node" />

export function readFile(path: string): Buffer;
```

---

### decl-no-reference-path

**One-liner:** Don't use `/// <reference path="..." />` in declaration files for module dependencies.

**When to apply:** When you might be tempted to reference another declaration file using a path.

```typescript
// BAD — use imports instead of reference path
/// <reference path="../utils/types.d.ts" />

// GOOD — use regular imports
import { MyType } from "../utils/types";
```

**Why:** `/// <reference path="..." />` is a legacy mechanism. Modern TypeScript uses ES module imports for referencing types between files.

---

### decl-no-umd-reference

**One-liner:** Don't use `/// <reference` directives in module files to declare UMD library dependencies — use regular imports.

**When to apply:** When a module file needs types from a UMD library.

```typescript
// BAD — in a module file (.ts with imports/exports)
/// <reference types="moment" />
moment.format(...); // relies on global UMD

// GOOD — use a regular import
import moment from "moment";
moment.format(...);
```

---

### decl-specific-overloads-first

**One-liner:** Place more specific overloads before more general ones in declaration files.

**When to apply:** Any time you declare overloaded functions in `.d.ts` files or interfaces.

```typescript
// BAD — the general overload shadows specific ones
declare function fn(x: any): any;
declare function fn(x: HTMLDivElement): number; // unreachable
declare function fn(x: HTMLElement): string;    // unreachable

// GOOD — specific first, general last
declare function fn(x: HTMLDivElement): number;
declare function fn(x: HTMLElement): string;
declare function fn(x: any): any;
```

---

### decl-collapse-overloads-optional

**One-liner:** Collapse overloads with differing trailing parameters into one signature with optional parameters.

**When to apply:** When multiple overloads share the same return type and differ only in how many trailing parameters are provided.

```typescript
// BAD
interface Example {
  diff(one: string): number;
  diff(one: string, two: string): number;
  diff(one: string, two: string, three: boolean): number;
}

// GOOD
interface Example {
  diff(one: string, two?: string, three?: boolean): number;
}
```

---

### decl-collapse-overloads-union

**One-liner:** Collapse overloads with different argument types and same return type into one signature with a union.

**When to apply:** When multiple overloads differ only by one parameter's type.

```typescript
// BAD
interface Moment {
  utcOffset(): number;
  utcOffset(b: number): Moment;
  utcOffset(b: string): Moment;
}

// GOOD
interface Moment {
  utcOffset(): number;
  utcOffset(b: number | string): Moment;
}
```

---

## Declaration Patterns Reference

### Global Variables

```typescript
declare var foo: number;          // mutable global
declare const bar: string;        // read-only global
declare let baz: boolean;
```

### Global Functions

```typescript
declare function greet(greeting: string): void;

// Overloaded:
declare function greet(greeting: string): string;
declare function greet(greeting: string, name: string): string;
```

### Objects with Properties (Namespace)

```typescript
declare namespace myLib {
  function makeGreeting(s: string): string;
  let numberOfGreetings: number;
  interface GreetingSettings {
    greeting: string;
    duration?: number;
    color?: string;
  }
}
```

### Reusable Types

```typescript
// Interface for object shape
interface GreetingSettings {
  greeting: string;
  duration?: number;
  color?: string;
}

// Type alias for union
type StringOrNumber = string | number;
```

### Classes

```typescript
declare class Greeter {
  constructor(customGreeting?: string);
  greet(): string;
  currentGreeting: string;
}
```

### Module with Default Export

```typescript
declare function myFunc(x: string): string;
declare namespace myFunc {
  let version: string;
}
export = myFunc;
```

### Module Augmentation

```typescript
// Extend an existing module's types
import { Observable } from "./observable";
declare module "./observable" {
  interface Observable<T> {
    map<U>(projector: (el: T) => U): Observable<U>;
  }
}
```

### Global Augmentation from a Module

```typescript
// Add a property to the global Window object
export {};
declare global {
  interface Window {
    myCustomProperty: string;
  }
}
```
