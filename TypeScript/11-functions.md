# TypeScript Functions

## Parameter & Return Type Syntax

```ts
function name(param: type): returnType {
  // ...
}
```

---

## Return Types

```ts
function getTime(): number {
  return new Date().getTime(); // ✅ must return number
}

function greet(): string {
  return "Hello!"; // ✅ must return string
}
```

TypeScript infers return type if you omit it — but explicit is safer for public functions.

---

## `void` — No Return Value

```ts
function printHello(): void {
  console.log("Hello!"); // no return
}
```

---

## All Parameter Types at a Glance

| Type             | Syntax                              | Required? |
|------------------|-------------------------------------|-----------|
| Normal           | `a: number`                         | Yes       |
| Optional         | `c?: number`                        | No        |
| Default value    | `exp: number = 10`                  | No (has fallback) |
| Rest (spread)    | `...rest: number[]`                 | No        |
| Named/destructured | `{ a, b }: { a: number; b: number }` | Yes    |

---

## Normal Parameters

```ts
function multiply(a: number, b: number) {
  return a * b; // return type inferred as number
}

multiply(3, 4);    // ✅ 12
multiply(3, "4");  // ❌ Error: string not assignable to number
```

---

## Optional Parameters (`?`)

```ts
function add(a: number, b: number, c?: number) {
  return a + b + (c || 0);
}

add(1, 2);    // ✅ 3  — c is undefined, falls back to 0
add(1, 2, 3); // ✅ 6
```

---

## Default Parameters

```ts
function pow(value: number, exponent: number = 10) {
  return value ** exponent;
}

pow(2);     // ✅ 2^10 = 1024 — uses default
pow(2, 3);  // ✅ 2^3  = 8   — overrides default
```

TypeScript infers `exponent` type as `number` from the default value.

---

## Named / Destructured Parameters

```ts
function divide({ dividend, divisor }: { dividend: number; divisor: number }) {
  return dividend / divisor;
}

divide({ dividend: 10, divisor: 2 }); // ✅ 5
```

Cleaner with a type alias:

```ts
type DivideArgs = { dividend: number; divisor: number };

function divide({ dividend, divisor }: DivideArgs) {
  return dividend / divisor;
}
```

---

## Rest Parameters (`...`)

Accepts any number of extra arguments — always typed as an array:

```ts
function add(a: number, b: number, ...rest: number[]) {
  return a + b + rest.reduce((total, n) => total + n, 0);
}

add(1, 2);           // ✅ 3
add(1, 2, 3, 4, 5);  // ✅ 15
```

---

## Function Type Alias

Name a function signature so it can be reused:

```ts
type Negate = (value: number) => number;

const negateFunction: Negate = (value) => value * -1;
// TypeScript infers `value` is number — no annotation needed on the arrow function

negateFunction(5);  // ✅ -5
negateFunction("x"); // ❌ Error
```

More examples:

```ts
type Callback = (error: string | null, result?: number) => void;

type Transformer = (input: string) => string;
const upper: Transformer = (s) => s.toUpperCase();
```

---

## Quick Summary

```ts
// Required params
function greet(name: string): string { ... }

// Optional param
function greet(name?: string): string { ... }

// Default param
function greet(name: string = "stranger"): string { ... }

// Rest params
function sum(...nums: number[]): number { ... }

// Named params
function log({ level, message }: { level: string; message: string }): void { ... }

// Function type alias
type Handler = (event: string) => void;
```
