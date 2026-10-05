# TypeScript Type Inference (Advanced)

> Basics covered in [03-explicit-vs-inference.md](03-explicit-vs-inference.md). This file covers deeper patterns.

---

## Where Inference Happens

| Context | Example | Inferred type |
|---|---|---|
| Variable init | `let x = 42` | `number` |
| Array literal | `let a = [1, 2, 3]` | `number[]` |
| Mixed array | `let m = [1, "two", true]` | `(string \| number \| boolean)[]` |
| Function return | `return a + b` | from expression |
| Callback param | `arr.forEach(x => ...)` | from array element type |
| Object literal | `const u = { name: "Alice" }` | `{ name: string }` |

---

## Function Return Type Inference

```ts
function greet(name: string) { return `Hello, ${name}!`; }
// inferred return: string

function add(a: number, b: number) { return a + b; }
// inferred return: number

function getValue(key: string) {
  if (key === "name") return "Alice";
  return 42;
}
// inferred return: string | number
```

---

## Contextual Typing

TS infers callback param types from context:

```ts
const names = ["Alice", "Bob", "Charlie"]; // string[]

names.forEach(name => {
  name.toUpperCase(); // ✅ name inferred as string
});

const lengths = names.map(name => name.length);
// lengths inferred as number[]
```

DOM event handlers:
```ts
document.addEventListener("click", event => {
  console.log(event.clientX); // event inferred as MouseEvent
});
```

---

## Object Literal Inference

```ts
const user = {
  id: 1,
  name: "Alice",
  details: {
    age: 30,
    address: { city: "New York" }
  }
};

user.name.toUpperCase();              // ✅
user.details.address.city.toLowerCase(); // ✅
user.age;                              // ❌ Error: doesn't exist
```

---

## `as const` — Narrow to Literal Types

Without `as const` — widens to base type:
```ts
let name = "Alice";       // type: string (wide)
const name = "Alice";     // type: string (still wide)
```

With `as const` — locks to exact literal:
```ts
const name = "Alice" as const;  // type: "Alice" (narrow)

const config = {
  env: "production",
  retries: 3,
  roles: ["admin", "user"] as const
} as const;
// config.env  type: "production" (not string)
// config.roles type: readonly ["admin", "user"] (not string[])
// config.retries = 5 ❌ readonly
```

---

## Control Flow Narrowing

TS tracks type through `if`/`switch`/`return`:

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    value.toUpperCase(); // ✅ string here
  } else {
    value.toFixed(2);    // ✅ number here
  }
}

// After early return — TS knows remaining type
function getLength(value: string | null): number {
  if (value === null) return 0; // exit null case
  return value.length;          // TS knows: string here
}
```

---

## When to Use Explicit Types

| Situation | Use explicit? |
|---|---|
| Simple var `let x = 42` | ❌ Let TS infer |
| Function parameters | ✅ Always |
| Function return (complex) | ✅ Yes |
| Empty array `[]` | ✅ `string[]` else inferred as `never[]` |
| Empty object `{}` | ✅ `Record<string, unknown>` |
| Public API / library | ✅ Always |

```ts
// ❌ Empty array without annotation — unsafe
const items = []; // inferred as never[]

// ✅ Explicit
const items: string[] = [];
const config: Record<string, unknown> = {};
```

---

## Quick Rules

```
Let TS infer          →  simple vars with clear initial value
Always explicit       →  function params, empty arrays/objects, public API
Use as const          →  lock object/array to literal types (config, enums)
Trust control flow    →  TS narrows after typeof/null checks automatically
```
