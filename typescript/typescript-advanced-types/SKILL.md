---
name: typescript-advanced-types
description: TypeScript advanced type features — conditional types, infer keyword, distributive types, mapped types with modifiers, key remapping, template literal types, keyof, and indexed access. Apply when building type utilities or generic abstractions.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
  version: "1.0"
---

# Advanced Types (LOW-MEDIUM)

Rules for writing correct and intentional advanced TypeScript type constructs. These features are powerful but can produce unexpected results if misapplied.

---

## Conditional Types

### adv-conditional-syntax

**One-liner:** Use `T extends U ? X : Y` to branch on type relationships.

**When to apply:** When a type should resolve differently based on what `T` is.

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false
type C = IsString<"hello">; // true (string literal extends string)
```

---

### adv-infer-extract

**One-liner:** Use `infer` within the `extends` clause to capture and reuse a type variable during inference.

**When to apply:** When you need to extract a part of a type (e.g., return type, array element, Promise inner type).

```typescript
// Extract array element type
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type Str = Flatten<string[]>; // string
type Num = Flatten<number>;   // number (not an array — returns T itself)

// Extract Promise inner type (simplified Awaited)
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type Value = Unwrap<Promise<string>>; // string

// Extract function return type (simplified ReturnType)
type GetReturn<T> = T extends (...args: never[]) => infer R ? R : never;

// Extract first element of a tuple
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;
type Head = First<[string, number, boolean]>; // string
```

**Why:** `infer` is the primary mechanism for "reading" the structure of a type from within a conditional, enabling powerful type utilities.

---

### adv-prevent-distribution

**One-liner:** Wrap type parameters in a tuple `[T]` to prevent distributive behavior in conditional types.

**When to apply:** When you want a conditional type to apply to a union as a whole, not to each member separately.

```typescript
// DISTRIBUTIVE — applies condition to each union member separately
type ToArray<T> = T extends any ? T[] : never;
type StrOrNumArr = ToArray<string | number>;
// string[] | number[]  (distributed over the union)

// NON-DISTRIBUTIVE — wrapping in tuple prevents distribution
type ToArrayNoDist<T> = [T] extends [any] ? T[] : never;
type Mixed = ToArrayNoDist<string | number>;
// (string | number)[]  (treated as a single union type)
```

**Why:** When `T` is a bare type parameter, conditional types automatically distribute over union members. Wrapping in `[T]` opts out of this behavior.

---

## Mapped Types

### adv-mapped-types

**One-liner:** Use `{ [K in keyof T]: ... }` to iterate over object type keys and transform their types.

**When to apply:** When building utility types that transform all properties of an object type.

```typescript
// Make all properties optional (like Partial<T>)
type Optional<T> = { [K in keyof T]?: T[K] };

// Make all properties readonly (like Readonly<T>)
type Immutable<T> = { readonly [K in keyof T]: T[K] };

// Make all properties nullable
type Nullable<T> = { [K in keyof T]: T[K] | null };
```

---

### adv-mapped-modifiers

**One-liner:** Use `+` (add) or `-` (remove) modifiers to control `readonly` and `?` on mapped type properties.

**When to apply:** When creating Required, Mutable, or similar inverse utility types.

```typescript
// Remove optional — like Required<T>
type Concrete<T> = { [K in keyof T]-?: T[K] };

// Remove readonly — Mutable
type Mutable<T> = { -readonly [K in keyof T]: T[K] };

// Add readonly — DeepReadonly
type DeepReadonly<T> = { +readonly [K in keyof T]: T[K] };

// Both: remove optional AND remove readonly
type WritableConcrete<T> = { -readonly [K in keyof T]-?: T[K] };
```

**Why:** Without explicit `-` modifier removal, `Partial<Readonly<T>>` would preserve readonly. Using `-readonly` removes it explicitly.

---

### adv-key-remapping

**One-liner:** Use `as` in mapped types to rename or filter keys, including with template literals.

**When to apply:** When building derived types that rename or subset the keys of another type.

**TypeScript 4.1+**

```typescript
// Rename keys with template literals — generate getter names
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person { name: string; age: number; }
type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// Filter keys by returning 'never'
type OnlyStrings<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K];
};

interface Mixed { id: number; name: string; active: boolean; label: string; }
type StringFields = OnlyStrings<Mixed>;
// { name: string; label: string }
```

**Why:** Key remapping enables generating accessor names, filtering out unwanted properties, and building event systems — all within the type system.

---

## Template Literal Types

### adv-template-literal-types

**One-liner:** Use template literal types to build string types from string unions and enable type-safe string patterns.

**When to apply:** When typing event names, CSS classes, API routes, or any string that follows a pattern.

**TypeScript 4.1+**

```typescript
// Type-safe event names
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// Cross-product of unions
type Alignment = "left" | "right" | "center";
type Position = "top" | "bottom";
type AnchorClass = `${Position}-${Alignment}`;
// "top-left" | "top-right" | "top-center" | "bottom-left" | ...

// Type-safe prop change events
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;
const watched = makeWatchedObject({ name: "Alice", age: 30 });
watched.on("nameChanged", (name) => { /* name: string */ });
watched.on("ageChanged", (age) => { /* age: number */ });
watched.on("unknownChanged", ...); // Error — "unknown" is not a key
```

---

## Keyof and Indexed Access Types

### adv-keyof-indexed-access

**One-liner:** Use `keyof T` and `T[K]` for type-safe property key and value type extraction.

**When to apply:** When writing generic utilities that need to work with object properties.

```typescript
// keyof — produces a union of all property keys
type Point = { x: number; y: number };
type PointKey = keyof Point; // "x" | "y"

// Indexed access — get the type of a specific property
type XType = Point["x"]; // number

// Combine for generic value extraction
type ValueOf<T> = T[keyof T];
type PointValues = ValueOf<Point>; // number

// Array element type via numeric index
const arr = ["a", "b", "c"];
type ArrElement = typeof arr[number]; // string

// Nested access
type NestedValue = { user: { name: string; age: number } };
type UserName = NestedValue["user"]["name"]; // string
```

---

### adv-typeof-operator

**One-liner:** Use the `typeof` type operator to reference the type of a value without importing a separate type declaration.

**When to apply:** When you need the type of a variable, function, or object without explicitly declaring it.

```typescript
// Get type of a function's return value
function createUser(name: string) { return { id: 1, name }; }
type User = ReturnType<typeof createUser>; // { id: number; name: string }

// Get type of a complex config object
const defaultConfig = {
  host: "localhost",
  port: 3000,
  debug: false,
} as const;

type Config = typeof defaultConfig;
// { readonly host: "localhost"; readonly port: 3000; readonly debug: false }

// Get instance type of a class
class Store { state = { count: 0 }; }
type StoreState = InstanceType<typeof Store>["state"]; // { count: number }
```

---

### adv-const-assertion

**One-liner:** Use `as const` to infer the narrowest possible literal type from an object or array.

**When to apply:** When defining configuration objects, lookup tables, or tuples that should have literal types.

```typescript
// Without as const — widened types
const config = { env: "production", port: 3000 };
// type: { env: string; port: number }

// With as const — literal types preserved
const config = { env: "production", port: 3000 } as const;
// type: { readonly env: "production"; readonly port: 3000 }

// Tuple type inference
const pair = [1, "hello"] as const;
// type: readonly [1, "hello"] — not (number | string)[]

// Enum-like object pattern
const Direction = { Up: "UP", Down: "DOWN", Left: "LEFT", Right: "RIGHT" } as const;
type Direction = typeof Direction[keyof typeof Direction];
// "UP" | "DOWN" | "LEFT" | "RIGHT"
```

**Why:** Without `as const`, TypeScript widens string literals to `string` and number literals to `number`. `as const` preserves the exact values as their literal types, which is essential for discriminated unions and lookup patterns.
