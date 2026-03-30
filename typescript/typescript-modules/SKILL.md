---
name: typescript-modules
description: TypeScript module best practices — ES module syntax, import type, verbatimModuleSyntax, namespaces vs modules, module augmentation, and module resolution. Apply when writing imports/exports or structuring module architecture.
metadata:
  source: https://www.typescriptlang.org/docs/handbook/2/modules.html
  version: "1.0"
---

# Modules (MEDIUM)

Rules for writing correct, tree-shakeable, and runtime-safe TypeScript modules.

---

### mod-import-type

**One-liner:** Use `import type` / `export type` for type-only imports and exports.

**When to apply:** When importing or re-exporting types, interfaces, or type aliases that have no runtime value.

```typescript
// GOOD — type-only imports are erased at runtime
import type { User, ApiResponse } from "./types";
export type { User };

// Inline type modifier (TypeScript 4.5+) — mix value and type imports in one statement
import { fetchUser, type User } from "./api";

// Required when isolatedModules: true — any type import must use 'import type'
import type { SomeInterface } from "./interfaces";
```

**Why:** `import type` is completely erased at compile time and produces no runtime code. With `isolatedModules: true`, it's required for all type-only imports. It also signals intent clearly.

---

### mod-prefer-esm

**One-liner:** Prefer ES module files over TypeScript namespaces for organizing code in modern projects.

**When to apply:** When deciding how to structure code across multiple files.

```typescript
// BAD — namespace (TypeScript-specific, legacy pattern)
namespace MyLib {
  export function process(): void { ... }
  export interface Config { ... }
}

// GOOD — standard ES module
export function process(): void { ... }
export interface Config { ... }

// In another file
import { process, Config } from "./myLib";
```

**Why:** Namespaces predate ES modules and are a TypeScript-specific mechanism. Modern tooling (bundlers, Node.js, browsers) understands ES modules natively. Namespaces don't benefit from tree-shaking.

---

### mod-namespace-augment-only

**One-liner:** Use namespaces only in declaration files or for module augmentation — not to organize runtime code.

**When to apply:** When writing `.d.ts` files for libraries or augmenting existing module types.

```typescript
// ACCEPTABLE — namespace in declaration file to group types
declare namespace Express {
  interface Request {
    user?: { id: string };
  }
}

// ACCEPTABLE — namespace for grouping related exported types
export namespace ApiTypes {
  export interface Response<T> { data: T; status: number; }
  export interface Error { message: string; code: number; }
}
```

---

### mod-module-augmentation

**One-liner:** Use `declare module "x" {}` inside a module file to extend an existing module's types.

**When to apply:** When adding properties to existing module types (e.g., Express Request, Vue component options).

```typescript
// Augment Express Request type
import "express";
declare module "express" {
  interface Request {
    user?: { id: string; roles: string[] };
    sessionId: string;
  }
}

// Augment a third-party library
import { Observable } from "rxjs";
declare module "rxjs" {
  interface Observable<T> {
    cache(duration: number): Observable<T>;
  }
}
```

**Why:** Module augmentation extends existing types without forking the library's type definitions. The file must be a module (have at least one `import` or `export`).

---

### mod-verbatim-module-syntax

**One-liner:** With `verbatimModuleSyntax: true`, any import without `type` is kept in output; any import with `type` is dropped.

**When to apply:** When `verbatimModuleSyntax: true` is enabled in tsconfig.

```typescript
// This import IS kept in the output JS
import { fetchUser } from "./api";

// This import is completely erased
import type { User } from "./types";

// Mixed — 'fetchUser' is kept, 'User' type annotation is erased
import { fetchUser, type User } from "./api";
```

**Why:** `verbatimModuleSyntax` makes the import erasure behavior explicit and predictable, preventing accidentally including type-only imports in the runtime bundle.

---

### mod-esm-syntax

**One-liner:** Use standard ESM import/export syntax, not CommonJS `require()` or `module.exports`.

**When to apply:** In all TypeScript projects targeting modern environments.

```typescript
// GOOD — named exports
export const PI = 3.14;
export function add(a: number, b: number) { return a + b; }

// GOOD — default export
export default class Logger { ... }

// GOOD — re-exports
export { PI } from "./constants";
export * from "./utils";
export * as utils from "./utils"; // namespace re-export

// AVOID unless targeting CommonJS specifically
const utils = require("./utils");
module.exports = { PI };
```

---

### mod-no-barrel-re-export-types

**One-liner:** When re-exporting types from barrel files, use `export type` to prevent accidental runtime inclusion.

**When to apply:** When writing `index.ts` barrel files that re-export from multiple modules.

```typescript
// BAD — might include type-only exports in runtime output
export { User, fetchUser } from "./api";

// GOOD — explicit about what is a type vs. a value
export type { User } from "./api";
export { fetchUser } from "./api";
```
