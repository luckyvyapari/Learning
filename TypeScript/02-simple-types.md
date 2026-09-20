# TypeScript Simple Types (Primitives)

## All 5 Primitive Types

| Type      | Values                         | Example declaration              |
|-----------|--------------------------------|----------------------------------|
| `boolean` | `true` / `false`               | `let isActive: boolean = true`   |
| `number`  | integers, floats, hex, bin, oct| `let score: number = 42`         |
| `string`  | text (single/double/backtick)  | `let name: string = "Alice"`     |
| `bigint`  | integers > 2^53 − 1 (ES2020+) | `const big = BigInt(999999999)`  |
| `symbol`  | unique, immutable identifier   | `const id: symbol = Symbol("id")`|

---

## `boolean`

```ts
let isActive: boolean = true;
let hasPermission = false; // TypeScript infers 'boolean'

if (isActive) {
  console.log("Active!");
}
```

---

## `number`

```ts
let decimal: number = 6;
let hex: number    = 0xf00d;   // 61453
let binary: number = 0b1010;   // 10
let octal: number  = 0o744;    // 484
let float: number  = 3.14;
```

> TypeScript has **one** number type for all numeric values (no int vs float split).

---

## `string`

```ts
let color: string    = "blue";
let fullName: string = 'John Doe';
let age: number      = 30;

// Template literal
let sentence: string = `Hello, my name is ${fullName} and I'll be ${age + 1} next year.`;
```

---

## `bigint` (ES2020+)

```ts
const big: bigint = BigInt(9007199254740991);
const alsoOk = 9007199254740991n; // shorthand syntax

// Use when number exceeds Number.MAX_SAFE_INTEGER (2^53 - 1)
```

---

## `symbol`

```ts
const uniqueKey: symbol = Symbol("description");

const obj = {
  [uniqueKey]: "This is a unique property"
};

console.log(obj[uniqueKey]); // "This is a unique property"
```

> Every `Symbol()` call creates a **unique** value even with the same label.

```ts
Symbol("x") === Symbol("x"); // false — always unique
```
