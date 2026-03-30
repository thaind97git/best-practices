---
name: typescript-config
description: TypeScript tsconfig best practices — strict mode, key compiler options, module resolution, and project setup rules. Apply when creating or reviewing tsconfig.json for any TypeScript project.
metadata:
  source: https://www.typescriptlang.org/tsconfig
  version: "1.0"
---

# Configuration (CRITICAL)

tsconfig rules that determine type safety, compatibility, and compiler behavior. These settings are foundational — wrong configuration silently weakens type guarantees.

---

### cfg-strict-true

**One-liner:** Always enable `"strict": true` — this is the TypeScript team's official recommendation.

**When to apply:** Every TypeScript project, from the beginning.

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

`"strict": true` enables the entire strict family of checks:

| Flag | Effect |
|---|---|
| `strictNullChecks` | `null`/`undefined` are distinct types, not assignable to everything |
| `noImplicitAny` | Errors when TypeScript infers `any` without an explicit annotation |
| `strictFunctionTypes` | Correct contravariant checking for function parameters |
| `strictBindCallApply` | Stricter checking of `.bind()`, `.call()`, `.apply()` |
| `strictPropertyInitialization` | Class properties must be initialized in the constructor |
| `noImplicitThis` | Error on `this` expressions with an implied `any` type |
| `useUnknownInCatchVariables` | Catch variables typed as `unknown` instead of `any` |
| `alwaysStrict` | Parses source in strict mode and emits `"use strict"` |

**Why:** Strict mode catches an entire class of bugs at compile time that would otherwise silently cause runtime errors. Libraries that compile without strict may distribute types that break consumers' strict-mode projects.

---

### cfg-no-implicit-any

**One-liner:** Use `"noImplicitAny": true` to prevent TypeScript from silently falling back to `any`.

**When to apply:** Included automatically when `strict: true`. Set explicitly if not using `strict`.

```typescript
// Error with noImplicitAny — parameter 'x' implicitly has an 'any' type
function greet(x) {
  console.log(x.toUpperCase());
}

// Correct — explicit type annotation
function greet(x: string) {
  console.log(x.toUpperCase());
}
```

**Why:** Implicit `any` is a silent type hole. It means TypeScript gave up on inferring the type, which is almost always a signal that an annotation is needed.

---

### cfg-strict-null-checks

**One-liner:** Use `"strictNullChecks": true` to make `null` and `undefined` distinct, non-assignable types.

**When to apply:** Included automatically when `strict: true`. Without it, `null` and `undefined` are assignable to every type.

```typescript
// With strictNullChecks: true
let name: string = null; // Error — null not assignable to string
let optName: string | null = null; // OK — explicitly nullable

function greet(name: string | undefined) {
  console.log(name.toUpperCase()); // Error — name may be undefined
  console.log(name?.toUpperCase()); // OK — optional chaining
}
```

**Why:** Without `strictNullChecks`, TypeScript cannot catch the most common category of runtime errors: accessing properties on `null` or `undefined`.

---

### cfg-no-unchecked-indexed

**One-liner:** Enable `"noUncheckedIndexedAccess": true` to add `undefined` to array and record index access.

**When to apply:** When accessing arrays by index or records by key, since there is no guarantee a value exists.

```typescript
// Without noUncheckedIndexedAccess
const arr: string[] = [];
const x = arr[0]; // type: string (misleading — might be undefined)

// With noUncheckedIndexedAccess
const arr: string[] = [];
const x = arr[0]; // type: string | undefined (correct)
if (x !== undefined) {
  console.log(x.toUpperCase()); // safe
}
```

**Why:** Arrays and records may not have a value at every index. Without this flag, TypeScript lies about the type of index access results, leading to uncaught `undefined` errors at runtime.

---

### cfg-isolated-modules

**One-liner:** Use `"isolatedModules": true` when using single-file transpilers like Babel, esbuild, or swc.

**When to apply:** Any project where TypeScript files are transpiled one at a time without full type information.

```typescript
// Error with isolatedModules — re-exporting a type requires 'export type'
export { SomeType } from "./types"; // Error

// Correct
export type { SomeType } from "./types"; // OK
```

**Why:** Single-file transpilers can't see the full type graph. `isolatedModules` ensures each file is independently transpilable and that type-only exports won't accidentally end up in the output.

---

### cfg-module-resolution

**One-liner:** Use `"moduleResolution": "bundler"` for bundler projects, `"NodeNext"` for Node.js ESM.

**When to apply:** When configuring a new project or updating an existing one that uses a modern build tool.

```json
// For Vite, webpack, esbuild, Rollup
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "module": "ESNext"
  }
}

// For Node.js ESM
{
  "compilerOptions": {
    "moduleResolution": "NodeNext",
    "module": "NodeNext"
  }
}
```

**Why:** The legacy `"node"` resolution doesn't support `package.json` `exports` field and cannot resolve `.js` extensions in ESM imports. `"bundler"` handles modern package exports correctly.

---

### cfg-verbatim-module-syntax

**One-liner:** Use `"verbatimModuleSyntax": true` for clear reasoning about which imports are erased.

**When to apply:** TypeScript 5.0+ projects, especially with `isolatedModules`.

```typescript
// With verbatimModuleSyntax: true
import type { User } from "./types";     // erased at runtime
import { fetchUser } from "./api";       // kept in output

export type { User };                    // erased at runtime
export { fetchUser };                    // kept in output
```

**Why:** Makes it obvious which imports survive to runtime and which are erased. Prevents accidentally keeping type-only imports in the output.

---

### cfg-no-implicit-override

**One-liner:** Enable `"noImplicitOverride": true` to require the `override` keyword on overriding methods.

**When to apply:** Any project using class inheritance.

```typescript
class Base {
  greet() { return "Hello"; }
}

// Error with noImplicitOverride — must use override keyword
class Child extends Base {
  greet() { return "Hi"; } // Error

  override greet() { return "Hi"; } // Correct
}
```

**Why:** Without `override`, removing or renaming a method in the base class silently orphans the overriding method in the subclass. `noImplicitOverride` catches this at compile time.

---

### cfg-recommended-options

**One-liner:** Use this recommended baseline tsconfig for modern TypeScript projects.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true
  }
}
```

**Why:** This baseline catches the most errors, produces clean output, and is compatible with modern toolchains.
