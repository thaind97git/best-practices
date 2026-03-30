---
name: typescript-classes
description: TypeScript class best practices — access modifiers, parameter properties, hard private fields, override keyword, abstract classes, and the this type. Apply when writing or reviewing class definitions.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/classes.html
  version: "1.0"
---

# Classes (MEDIUM)

Rules for writing correct, encapsulated, and extensible TypeScript classes.

---

### cls-parameter-properties

**One-liner:** Use parameter property shorthand to declare and initialize class properties in the constructor concisely.

**When to apply:** When a constructor parameter's only purpose is to initialize a class field of the same name.

```typescript
// VERBOSE — declare field, then assign in constructor
class Point {
  x: number;
  y: number;
  private readonly id: string;

  constructor(x: number, y: number, id: string) {
    this.x = x;
    this.y = y;
    this.id = id;
  }
}

// CONCISE — parameter property shorthand
class Point {
  constructor(
    public x: number,
    public y: number,
    private readonly id: string
  ) {}
}
```

**Why:** Parameter properties reduce boilerplate while declaring, assigning, and annotating visibility all at once.

---

### cls-hard-private

**One-liner:** Use `#field` (JavaScript private) for true runtime privacy; `private` is TypeScript-only.

**When to apply:** When you need a field to be inaccessible outside the class at runtime (not just at compile time).

```typescript
class BankAccount {
  // TypeScript-only private — accessible via (account as any).balance at runtime
  private balance: number;

  // JavaScript private field — truly inaccessible at runtime
  #pin: string;

  constructor(balance: number, pin: string) {
    this.balance = balance;
    this.#pin = pin;
  }

  verify(pin: string): boolean {
    return this.#pin === pin;
  }
}

const account = new BankAccount(1000, "1234");
(account as any).balance; // Works — TypeScript 'private' is not enforced at runtime
(account as any).#pin;    // SyntaxError — truly private at runtime
```

**Why:** TypeScript's `private` modifier is erased at compile time. If runtime privacy matters (security, encapsulation), use `#field`.

---

### cls-override-keyword

**One-liner:** Always use the `override` keyword when overriding a base class method.

**When to apply:** Any time a derived class method is intended to override a base class method.

```typescript
class Animal {
  move(): void { console.log("Moving..."); }
  speak(): void { console.log("..."); }
}

// BAD — no indication this is an intentional override
class Dog extends Animal {
  move(): void { console.log("Running!"); }
}

// GOOD — explicit override catches typos and base class changes
class Dog extends Animal {
  override move(): void { console.log("Running!"); }
  override speak(): void { console.log("Woof!"); }
}

// TypeScript error if base method doesn't exist
class Cat extends Animal {
  override fly(): void { ... } // Error — 'fly' doesn't exist in Animal
}
```

**Why:** Without `override`, removing or renaming a method in the base class orphans the subclass method silently. Enable `noImplicitOverride: true` in tsconfig to enforce this.

---

### cls-abstract-for-base

**One-liner:** Use `abstract` classes for base types that define a contract but cannot be instantiated directly.

**When to apply:** When designing a class hierarchy where the base class provides shared implementation but must always be subclassed.

```typescript
abstract class Shape {
  abstract area(): number; // Must be implemented by subclasses

  // Concrete implementation shared by all subclasses
  toString(): string {
    return `Area: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) { super(); }
  area(): number { return Math.PI * this.radius ** 2; }
}

class Square extends Shape {
  constructor(private side: number) { super(); }
  area(): number { return this.side ** 2; }
}

new Shape(); // Error — cannot create an instance of an abstract class
new Circle(5); // OK
```

---

### cls-implements-vs-extends

**One-liner:** Use `implements` to check a class satisfies an interface contract; use `extends` to inherit implementation.

**When to apply:** When deciding how a class relates to an interface or another class.

```typescript
interface Serializable {
  serialize(): string;
  deserialize(data: string): this;
}

// implements — verifies the contract, does not inherit
class User implements Serializable {
  serialize(): string { return JSON.stringify(this); }
  deserialize(data: string): this { return Object.assign(this, JSON.parse(data)); }
}

// extends — inherits implementation
class AdminUser extends User {
  permissions: string[] = [];
}
```

**Why:** `implements` only checks conformance — it does not provide any implementation. `extends` provides inheritance. A class can implement multiple interfaces but only extend one class.

---

### cls-this-type

**One-liner:** Use `this` as the return type for methods that enable fluent/builder patterns in subclasses.

**When to apply:** When writing a base class method that should return the most-derived type in subclasses.

```typescript
class Builder {
  protected value = 0;

  add(n: number): this {
    this.value += n;
    return this;
  }
}

class SpecialBuilder extends Builder {
  multiply(n: number): this {
    this.value *= n;
    return this;
  }
}

const result = new SpecialBuilder()
  .add(5)      // returns SpecialBuilder (not just Builder)
  .multiply(3) // still available because 'this' preserves the derived type
  .add(1);
```

**Why:** Returning `this` (not `Builder`) preserves the derived type through chained calls, enabling method chaining to work correctly in subclasses.

---

### cls-access-modifiers

**One-liner:** Use access modifiers (`public`, `protected`, `private`) to document intended visibility, even when `public` is the default.

**When to apply:** When declaring class members, especially in shared or library code.

```typescript
class Animal {
  public name: string;           // accessible from anywhere (default)
  protected species: string;     // accessible within class and subclasses
  private internalState: string; // accessible only within this class

  constructor(name: string, species: string) {
    this.name = name;
    this.species = species;
    this.internalState = "active";
  }
}
```

**Avoid** using built-in `Function` property names as static members: `name`, `length`, `call` — they conflict with the class's own inherited `Function` properties.
