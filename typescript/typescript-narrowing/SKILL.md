---
name: typescript-narrowing
description: TypeScript type narrowing and guards — typeof, instanceof, in operator, discriminated unions, type predicates, and exhaustiveness checking. Apply when writing conditional logic that works with union types or when accepting values of multiple types.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
  version: "1.0"
---

# Type Narrowing & Guards (HIGH)

TypeScript uses **control flow analysis** to narrow types based on runtime checks. These rules ensure you use the right narrowing technique for each scenario.

---

### narrow-typeof

**One-liner:** Use `typeof` guards to narrow primitive types (`string`, `number`, `boolean`, `bigint`, `symbol`, `undefined`, `function`).

**When to apply:** When a parameter is a union of primitive types and you need to handle each case differently.

```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value; // padding: number
  }
  return padding + value; // padding: string
}
```

**Why:** TypeScript recognizes all valid `typeof` return values as type guards. Note: `typeof null === "object"` — always check for `null` separately when working with objects.

---

### narrow-instanceof

**One-liner:** Use `instanceof` to narrow a value to a specific class or constructor.

**When to apply:** When working with class instances, Error types, or DOM elements.

```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString()); // x: Date
  } else {
    console.log(x.toUpperCase()); // x: string
  }
}

// Error handling pattern
function handleError(err: unknown) {
  if (err instanceof Error) {
    console.error(err.message);
  } else {
    console.error(String(err));
  }
}
```

**Why:** `instanceof` checks the prototype chain and narrows the type to the specific class, enabling access to class-specific methods and properties.

---

### narrow-in-operator

**One-liner:** Use the `in` operator to narrow union types based on whether a property exists on the object.

**When to apply:** When narrowing between object types that share no common discriminant literal property.

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim(); // animal: Fish
  } else {
    animal.fly();  // animal: Bird
  }
}
```

**Why:** The `in` operator narrows to types that have an optional or required property with the given name, without needing a literal discriminant.

---

### narrow-discriminated-union

**One-liner:** Add a shared literal property (`kind`, `type`, `tag`) as a discriminant to enable exhaustive union narrowing.

**When to apply:** When designing union types that represent different cases of the same concept.

```typescript
// GOOD — discriminated union with 'kind' field
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":    return Math.PI * shape.radius ** 2;
    case "square":    return shape.side ** 2;
    case "rectangle": return shape.width * shape.height;
  }
}
```

**Why:** Discriminated unions let TypeScript narrow each branch completely, giving full type safety without type assertions. They also enable exhaustiveness checking.

---

### narrow-type-predicate

**One-liner:** Write custom type guards with `arg is SomeType` return types when built-in narrowing is insufficient.

**When to apply:** When you need to check a complex runtime property or validate that a value conforms to a specific type.

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Usage
if (isFish(myPet)) {
  myPet.swim(); // myPet: Fish — TypeScript narrows based on the type predicate
} else {
  myPet.fly();  // myPet: Bird
}

// Array filtering with type guard
const animals: (Fish | Bird)[] = [...];
const allFish = animals.filter(isFish); // Fish[]
```

**Why:** Type predicates allow you to teach TypeScript how to narrow based on custom runtime logic, enabling type-safe filtering and narrowing that TypeScript cannot infer automatically.

---

### narrow-never-exhaustive

**One-liner:** Assign an unhandled union member to `never` in the default branch to get a compile error if a case is missed.

**When to apply:** In `switch` statements or `if-else` chains that must handle all members of a union.

```typescript
function assertNever(x: never): never {
  throw new Error("Unhandled case: " + String(x));
}

type Direction = "up" | "down" | "left" | "right";

function move(dir: Direction): void {
  switch (dir) {
    case "up":    return moveUp();
    case "down":  return moveDown();
    case "left":  return moveLeft();
    case "right": return moveRight();
    default:      return assertNever(dir); // Compile error if any case is missing
  }
}
```

**Why:** If a new member is added to the union and no case is added, TypeScript reports a compile error because the new member cannot be assigned to `never`. This prevents missed cases from slipping through.

---

### narrow-nullish-equality

**One-liner:** Use `x == null` (loose equality) to narrow out both `null` and `undefined` at once.

**When to apply:** When you want to guard against both `null` and `undefined` in a single check.

```typescript
function greet(name: string | null | undefined) {
  // Loose equality narrows out both null and undefined
  if (name != null) {
    console.log(name.toUpperCase()); // name: string
  }
}

// Equivalent to:
if (name !== null && name !== undefined) { ... }
```

**Why:** `x == null` is true for both `null` and `undefined`. Using it is concise and idiomatic for guarding against nullish values.

---

### narrow-truthiness

**One-liner:** Use truthiness checks to guard against `null`, `undefined`, `0`, `""`, `NaN`, and `false`.

**When to apply:** When guarding against nullish values, but be careful with primitive values like `0` or `""` which are falsy but valid.

```typescript
// GOOD — guard against null/undefined
function print(val: string | null | undefined) {
  if (val) {
    console.log(val.toUpperCase()); // val: string
  }
}

// CAREFUL — 0 and "" are falsy
function processCount(count: number | null) {
  if (count) { // WRONG — skips count=0
    process(count);
  }
  if (count != null) { // GOOD — only skips null/undefined
    process(count);
  }
}
```

**Why:** Truthiness narrowing is convenient for `string | null` or `T | undefined` unions, but can incorrectly filter valid falsy values like `0`, `""`, or `false`.
