# TypeScript Conditional Types

Type-level if-else: `T extends U ? X : Y`

"If T is assignable to U, use X, otherwise use Y."

---

## Basic Syntax

```ts
type IsString<T> = T extends string ? true : false;

type R1 = IsString<string>;  // true
type R2 = IsString<number>;  // false
type R3 = IsString<"hello">; // true — literals extend their base type
```

---

## Distributive Conditional Types (Unions)

When T is a union, the condition runs on **each member separately**:

```ts
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// = ToArray<string> | ToArray<number>
// = string[] | number[]
```

Extract specific types from a union:
```ts
type ExtractStrings<T> = T extends string ? T : never;

type Strings = ExtractStrings<string | number | boolean | "hello">;
// = string | "hello"
```

---

## `infer` — Extract a Type from Inside Another Type

```ts
// Extract return type of any function
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet()     { return "hello"; }
function getNumber() { return 42; }

type A = ReturnType<typeof greet>;     // string
type B = ReturnType<typeof getNumber>; // number
```

```ts
// Extract element type from array
type ElementType<T> = T extends (infer U)[] ? U : never;

type C = ElementType<number[]>; // number
type D = ElementType<string[]>; // string
```

---

## Built-in Conditional Utility Types

| Type | What it does | Example |
|---|---|---|
| `Extract<T, U>` | Keep types assignable to U | `Extract<string \| number, string>` = `string` |
| `Exclude<T, U>` | Remove types assignable to U | `Exclude<string \| number, string>` = `number` |
| `NonNullable<T>` | Remove null and undefined | `NonNullable<string \| null>` = `string` |
| `Parameters<T>` | Param types as tuple | `Parameters<(a: string) => void>` = `[string]` |
| `ReturnType<T>` | Return type of function | `ReturnType<() => number>` = `number` |

```ts
type A = NonNullable<string | null | undefined>; // string
type B = Extract<string | number | boolean, string>; // string
type C = Exclude<string | number | boolean, string>; // number | boolean
```

---

## If-Else Chain (Type-Level Switch)

```ts
type TypeName<T> =
  T extends string    ? "string"    :
  T extends number    ? "number"    :
  T extends boolean   ? "boolean"   :
  T extends Function  ? "function"  :
  "object";

type T0 = TypeName<string>;     // "string"
type T1 = TypeName<42>;         // "number"
type T2 = TypeName<() => void>; // "function"
type T3 = TypeName<Date[]>;     // "object"
```

---

## Recursive Conditional Types

Unwrap nested types:

```ts
type UnwrapPromise<T> = T extends Promise<infer U> ? UnwrapPromise<U> : T;

type A = UnwrapPromise<Promise<string>>;           // string
type B = UnwrapPromise<Promise<Promise<number>>>;  // number
type C = UnwrapPromise<boolean>;                   // boolean
```

---

## Quick Reference

| Concept | Syntax |
|---|---|
| Basic conditional | `T extends U ? X : Y` |
| infer (extract inner type) | `T extends Promise<infer U> ? U : never` |
| Distribute over union | Happens automatically when T is a union |
| Recursive | Call itself in the true/false branch |

---

## Rule of Thumb

```
Simple type check       →  typeof / instanceof (runtime)
Type-level logic        →  conditional types (compile time)
Extract inner type      →  infer
Remove from union       →  Exclude<T, U>
Keep from union         →  Extract<T, U>
Strip null/undefined    →  NonNullable<T>
```
