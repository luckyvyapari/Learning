# TypeScript Union Types

## What is a Union?

A union means a value can be **one of several types** — like an OR condition.

```ts
// This parameter accepts string OR number
function printStatusCode(code: string | number) {
  console.log(`My status code is ${code}.`);
}

printStatusCode(404);   // ✅ number
printStatusCode("404"); // ✅ string
printStatusCode(true);  // ❌ Error: boolean not in union
```

---

## Syntax

```ts
type A | type B | type C
```

```ts
let id: string | number;
id = 101;     // ✅
id = "E101";  // ✅
id = true;    // ❌ Error
```

---

## Common Union Patterns

| Pattern                    | Example                                  |
|----------------------------|------------------------------------------|
| Two primitives             | `string \| number`                       |
| Literal union (fixed set)  | `"success" \| "error" \| "loading"`     |
| Nullable value             | `string \| null`                         |
| Optional (undefined)       | `string \| undefined`                    |
| Object union               | `Cat \| Dog`                             |
| Array union                | `string[] \| number[]`                   |

---

## Literal Union — Fixed Set of Values

```ts
type Status = "success" | "error" | "loading";

let state: Status = "success"; // ✅
state = "error";               // ✅
state = "done";                // ❌ Error: "done" not in union
```

```ts
type Direction = "up" | "down" | "left" | "right";

function move(dir: Direction) {
  console.log(`Moving ${dir}`);
}

move("up");    // ✅
move("fly");   // ❌
```

---

## Union with `null` / `undefined`

```ts
function greet(name: string | null) {
  if (name === null) {
    console.log("Hello, stranger!");
  } else {
    console.log(`Hello, ${name}!`);
  }
}

greet("Alice"); // "Hello, Alice!"
greet(null);    // "Hello, stranger!"
```

---

## Narrowing — Using the Union Safely

When you have a union, TypeScript forces you to check the type before using type-specific methods:

```ts
function format(value: string | number) {
  // ❌ Can't call .toUpperCase() — might be a number
  // value.toUpperCase();

  if (typeof value === "string") {
    console.log(value.toUpperCase()); // ✅ safe — string confirmed
  } else {
    console.log(value.toFixed(2));    // ✅ safe — number confirmed
  }
}

format("hello"); // "HELLO"
format(3.14159); // "3.14"
```

---

## Object Union

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

type Pet = Cat | Dog;

function makeNoise(pet: Pet) {
  if ("meow" in pet) {
    pet.meow(); // ✅ TypeScript knows it's a Cat
  } else {
    pet.bark(); // ✅ TypeScript knows it's a Dog
  }
}
```

---

## Union vs `any`

| | `any` | Union |
|---|---|---|
| Type safe? | ❌ No | ✅ Yes |
| Forces type check? | ❌ No | ✅ Yes |
| Autocomplete works? | ❌ No | ✅ Yes |
| Use when | Migrating JS code | Value has known possible types |

```ts
// ❌ any — no safety
function print(val: any) { val.toUpperCase(); } // no error even if number

// ✅ union — safe
function print(val: string | number) { val.toUpperCase(); } // ❌ Error caught!
```

---

## Quick Reference

```ts
// OR — value is one of these
type X = string | number | boolean;

// Literal OR — value is one of these exact strings
type Dir = "north" | "south" | "east" | "west";

// Nullable
type MaybeString = string | null;

// Optional
type MaybeAge = number | undefined;
```
