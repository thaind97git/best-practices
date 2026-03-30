---
name: typescript
description: Comprehensive TypeScript best practices guide sourced from typescriptlang.org official documentation. Covers 10 categories with rules for type system fundamentals, narrowing, generics, functions, classes, modules, configuration, declarations, utility types, and advanced types. Apply when writing, reviewing, or refactoring TypeScript code.
user-invocable: true
metadata:
    source: https://www.typescriptlang.org/docs
    version: '1.0'
---

# TypeScript Best Practices

Comprehensive best practices sourced from the official TypeScript documentation (typescriptlang.org). Contains rules across 10 categories, organized by impact to guide type authoring, configuration, and advanced type usage.

---

## When to Apply

Reference these guidelines when:

- Writing new TypeScript types, interfaces, or generics
- Reviewing type safety and strictness of code
- Configuring tsconfig for a project
- Writing declaration files (`.d.ts`)
- Designing function overloads or callback types
- Using advanced type features (conditional, mapped, template literals)

---

## Rule Categories by Priority

| Priority | Category                 | Impact   | Prefix  |
| -------- | ------------------------ | -------- | ------- |
| 1        | Type System Fundamentals | CRITICAL | ts-     |
| 2        | Configuration (tsconfig) | CRITICAL | cfg-    |
| 3        | Type Narrowing & Guards  | HIGH     | narrow- |
| 4        | Functions                | HIGH     | fn-     |
| 5        | Generics                 | HIGH     | gen-    |
| 6        | Declaration Files        | HIGH     | decl-   |
| 7        | Modules                  | MEDIUM   | mod-    |
| 8        | Classes                  | MEDIUM   | cls-    |
| 9        | Utility Types            | MEDIUM   | util-   |
| 10       | Advanced Types           | LOW-MED  | adv-    |

---

## Quick Reference

### 1. Type System Fundamentals (CRITICAL)

- ts-lowercase-primitives — Use `string`, `number`, `boolean`, not `String`, `Number`, `Boolean`
- ts-no-any — Never use `any`; use `unknown` for truly unknown values
- ts-unknown-over-any — Use `unknown` when you must accept a value but won't interact with it directly
- ts-never-exhaustive — Use `never` + `assertNever()` for exhaustiveness checking in discriminated unions
- ts-interface-for-public-api — Prefer `interface` for public API shapes that may need extension
- ts-type-for-computed — Use `type` for unions, mapped types, conditional types, template literals
- ts-interface-extends-perf — `interface extends` is more compiler-performant than `&` intersections

---

### 2. Configuration (CRITICAL)

- cfg-strict-true — Always enable `"strict": true` in tsconfig
- cfg-no-implicit-any — `noImplicitAny` errors when TypeScript falls back to `any`
- cfg-strict-null-checks — `strictNullChecks` makes `null`/`undefined` distinct types
- cfg-no-unchecked-indexed — `noUncheckedIndexedAccess` adds `undefined` to array/record access
- cfg-isolated-modules — Use `isolatedModules: true` with single-file transpilers (Babel, esbuild)
- cfg-module-resolution — Use `"bundler"` for Vite/webpack, `"NodeNext"` for Node.js ESM
- cfg-verbatim-module-syntax — Use `verbatimModuleSyntax: true` for clarity on import erasure

---

### 3. Type Narrowing & Guards (HIGH)

- narrow-typeof — Use `typeof` guards for primitive type narrowing
- narrow-instanceof — Use `instanceof` for class instance narrowing
- narrow-in-operator — Use the `in` operator for object shape narrowing
- narrow-discriminated-union — Use a shared literal property as a discriminant in unions
- narrow-type-predicate — Write `arg is SomeType` return type for custom type guards
- narrow-never-exhaustive — Assign to `never` in default branch to catch missed union members
- narrow-nullish-equality — Use `x == null` to narrow out both `null` and `undefined`

---

### 4. Functions (HIGH)

- fn-void-not-any — Use `void` not `any` as the return type for ignored callback return values
- fn-no-optional-callback-params — Don't make callback parameters optional unless truly optional
- fn-specific-overloads-first — Put specific overloads before general ones
- fn-collapse-overloads-union — Use union types instead of overloads differing by single argument type
- fn-collapse-overloads-optional — Collapse overloads with trailing params into optional parameters
- fn-callback-max-arity — Write one overload with max arity instead of multiple callback arities
- fn-call-signature — Use call signatures on interfaces for callable objects with properties

---

### 5. Generics (HIGH)

- gen-no-unused-params — Don't declare generic type parameters that are never used
- gen-keyof-constraint — Use `K extends keyof T` to constrain key access
- gen-descriptive-names — Use descriptive names (`TElement`) for complex generics; single letters for simple ones
- gen-fewer-params — Fewer type parameters = simpler, more readable code
- gen-prefer-constraints — Prefer constraints over `any` for flexible but type-safe generics

---

### 6. Declaration Files (HIGH)

- decl-use-reference-types — Use `/// <reference types="..." />` for type package dependencies
- decl-no-reference-path — Don't use `/// <reference path="..." />` for module dependencies
- decl-no-umd-reference — Don't use `/// <reference` in module files for UMD libraries; use imports
- decl-collapse-overloads — Collapse overloads with union types and optional parameters

---

### 7. Modules (MEDIUM)

- mod-import-type — Use `import type` / `export type` for type-only imports
- mod-prefer-esm — Prefer ES modules over namespaces in modern TypeScript
- mod-namespace-augment-only — Use namespaces only for declaration files or module augmentation
- mod-module-augmentation — Use `declare module "x" {}` to extend existing module types

---

### 8. Classes (MEDIUM)

- cls-parameter-properties — Use parameter property shorthand for concise constructor definitions
- cls-hard-private — Use `#field` (JS private) for true runtime privacy, not `private`
- cls-override-keyword — Use `override` when overriding base class methods
- cls-abstract-for-base — Use `abstract` classes for base types that must be subclassed
- cls-this-type — Use `this` return type for fluent/builder patterns

---

### 9. Utility Types (MEDIUM)

- util-partial-for-updates — Use `Partial<T>` for update DTOs and optional-all scenarios
- util-omit-pick-reshape — Use `Omit<T, K>` and `Pick<T, K>` to reshape types without duplication
- util-record-for-maps — Use `Record<K, V>` for object maps with known key types
- util-awaited-for-promises — Use `Awaited<T>` to recursively unwrap Promise types
- util-returntype-inference — Use `ReturnType<typeof fn>` to infer return type without importing it

---

### 10. Advanced Types (LOW-MEDIUM)

- adv-infer-extract — Use `infer` in conditional types to extract and reuse type components
- adv-prevent-distribution — Wrap `[T]` instead of bare `T` to prevent distributive conditional types
- adv-mapped-modifiers — Use `-?` and `-readonly` to create Required/Mutable variants
- adv-key-remapping — Use `as` in mapped types to rename or filter keys
- adv-template-literal-types — Use template literal types for string composition and type-safe event names

---

## Category Files

Each category has its own `SKILL.md` with full rule details, anti-patterns, and code examples:

| Category                 | File                                                                       | Priority |
| ------------------------ | -------------------------------------------------------------------------- | -------- |
| Type System Fundamentals | [typescript-type-system/SKILL.md](./typescript-type-system/SKILL.md)       | CRITICAL |
| Configuration            | [typescript-config/SKILL.md](./typescript-config/SKILL.md)                 | CRITICAL |
| Type Narrowing & Guards  | [typescript-narrowing/SKILL.md](./typescript-narrowing/SKILL.md)           | HIGH     |
| Functions                | [typescript-functions/SKILL.md](./typescript-functions/SKILL.md)           | HIGH     |
| Generics                 | [typescript-generics/SKILL.md](./typescript-generics/SKILL.md)             | HIGH     |
| Declaration Files        | [typescript-declarations/SKILL.md](./typescript-declarations/SKILL.md)     | HIGH     |
| Modules                  | [typescript-modules/SKILL.md](./typescript-modules/SKILL.md)               | MEDIUM   |
| Classes                  | [typescript-classes/SKILL.md](./typescript-classes/SKILL.md)               | MEDIUM   |
| Utility Types            | [typescript-utility-types/SKILL.md](./typescript-utility-types/SKILL.md)   | MEDIUM   |
| Advanced Types           | [typescript-advanced-types/SKILL.md](./typescript-advanced-types/SKILL.md) | LOW-MED  |
