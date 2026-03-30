---
name: typescript-utility-types
description: TypeScript utility types reference and usage patterns — Partial, Required, Readonly, Record, Pick, Omit, Exclude, Extract, NonNullable, ReturnType, Awaited, and string intrinsics. Apply when transforming or deriving types without duplication.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/utility-types.html
  version: "1.0"
---

# Utility Types (MEDIUM)

Built-in TypeScript utilities for transforming types. Avoid duplicating type transformations — use these instead.

---

## Object Transformation Utilities

### `Partial<T>`

Makes all properties of `T` optional.

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

// GOOD — use for update DTOs, patch operations
type UpdateUserDto = Partial<User>;
// { id?: string; name?: string; email?: string }

function updateUser(id: string, changes: Partial<User>): User { ... }
```

---

### `Required<T>`

Makes all properties of `T` required (removes `?`).

```typescript
interface Options {
  timeout?: number;
  retries?: number;
  debug?: boolean;
}

// All properties are required
type ResolvedOptions = Required<Options>;
// { timeout: number; retries: number; debug: boolean }
```

---

### `Readonly<T>`

Makes all properties of `T` read-only.

```typescript
interface Config {
  host: string;
  port: number;
}

// GOOD — freeze configuration after creation
const config: Readonly<Config> = { host: "localhost", port: 3000 };
config.host = "remote"; // Error — cannot assign to readonly property
```

---

### `Record<K, V>`

Constructs an object type with keys of type `K` and values of type `V`.

```typescript
// GOOD — typed object map
type Routes = Record<"/home" | "/about" | "/contact", React.ComponentType>;
type HttpStatus = Record<number, string>;

// GOOD — dynamic dictionary with known key type
type CacheMap = Record<string, { data: unknown; expires: number }>;
```

---

### `Pick<T, K>`

Constructs a type with only the specified properties from `T`.

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
}

// Select only the safe-to-expose fields
type PublicUserSummary = Pick<User, "id" | "name">;
// { id: string; name: string }
```

---

### `Omit<T, K>`

Constructs a type without the specified properties from `T`.

```typescript
// Remove sensitive fields
type PublicUser = Omit<User, "passwordHash">;
// { id: string; name: string; email: string; createdAt: Date }

// GOOD — prefer Omit over Pick when excluding a small number of fields
type ApiUser = Omit<User, "passwordHash" | "internalId">;
```

---

## Union/Set Operation Utilities

### `Exclude<T, U>`

Removes from union `T` any member assignable to `U`.

```typescript
type Status = "active" | "inactive" | "banned" | "pending";

type ActiveStatus = Exclude<Status, "banned" | "inactive">;
// "active" | "pending"

type NonString = Exclude<string | number | boolean, string>;
// number | boolean
```

---

### `Extract<T, U>`

Keeps from union `T` only members assignable to `U`.

```typescript
type StringsOnly = Extract<string | number | boolean, string>;
// string

type CommonMembers = Extract<"a" | "b" | "c", "b" | "c" | "d">;
// "b" | "c"
```

---

### `NonNullable<T>`

Removes `null` and `undefined` from type `T`.

```typescript
type MaybeUser = User | null | undefined;
type DefiniteUser = NonNullable<MaybeUser>;
// User

// Useful when narrowing has already happened
function processUser(user: NonNullable<typeof maybeUser>) { ... }
```

---

## Function/Class Introspection Utilities

### `ReturnType<T>`

Extracts the return type of a function type `T`.

```typescript
// GOOD — derive return type without importing it explicitly
async function fetchUser(id: string) {
  return { id, name: "Alice", email: "alice@example.com" };
}

type UserResponse = Awaited<ReturnType<typeof fetchUser>>;
// { id: string; name: string; email: string }
```

---

### `Parameters<T>`

Extracts the parameter types of function type `T` as a tuple.

```typescript
function createUser(name: string, email: string, age: number): User { ... }

type CreateUserArgs = Parameters<typeof createUser>;
// [name: string, email: string, age: number]

// Useful for forwarding arguments
function wrappedCreateUser(...args: Parameters<typeof createUser>): User {
  console.log("Creating user...");
  return createUser(...args);
}
```

---

### `ConstructorParameters<T>`

Extracts the constructor parameter types of class `T` as a tuple.

```typescript
class HttpClient {
  constructor(baseUrl: string, timeout: number, headers: Record<string, string>) { ... }
}

type ClientArgs = ConstructorParameters<typeof HttpClient>;
// [baseUrl: string, timeout: number, headers: Record<string, string>]
```

---

### `InstanceType<T>`

Extracts the type produced by `new T`.

```typescript
class Store { state = { count: 0 }; }

type StoreInstance = InstanceType<typeof Store>;
// Store

// Useful when working with class constructors as values
function createInstance<T extends new (...args: any[]) => any>(
  Cls: T
): InstanceType<T> {
  return new Cls();
}
```

---

## Promise Utilities

### `Awaited<T>`

Recursively unwraps `Promise<T>` to the inner type.

```typescript
type A = Awaited<Promise<string>>;                  // string
type B = Awaited<Promise<Promise<number>>>;          // number
type C = Awaited<string | Promise<number>>;          // string | number

// GOOD — use with ReturnType to get async function's resolved value
async function fetchData() { return { items: [], total: 0 }; }
type FetchResult = Awaited<ReturnType<typeof fetchData>>;
// { items: never[]; total: number }
```

---

## String Intrinsic Utilities

```typescript
type A = Uppercase<"hello">;     // "HELLO"
type B = Lowercase<"HELLO">;     // "hello"
type C = Capitalize<"hello">;    // "Hello"
type D = Uncapitalize<"Hello">;  // "hello"

// Common pattern with template literal types
type EventHandler<T extends string> = `on${Capitalize<T>}`;
type ClickHandler = EventHandler<"click">; // "onClick"
```

---

## Common Patterns

```typescript
// Update form — all properties optional
type UpdateDto<T> = Partial<Omit<T, "id" | "createdAt">>;

// Deep partial (manual implementation needed)
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

// Extract function's return type without importing the type explicitly
type ApiData = Awaited<ReturnType<typeof fetchApiData>>;

// Typed lookup map
type StatusMap = Record<"active" | "inactive" | "banned", { label: string; color: string }>;
```

---

### util-pick-vs-omit-guidance

**One-liner:** Use `Pick` when selecting a few properties; use `Omit` when excluding a few properties.

```typescript
// If keeping 2 out of 10 fields → use Pick (smaller list)
type Summary = Pick<User, "id" | "name">;

// If excluding 2 out of 10 fields → use Omit (smaller list)
type SafeUser = Omit<User, "passwordHash" | "internalToken">;
```
