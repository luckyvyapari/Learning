# TypeScript Special Types

## Quick Reference Table

| Type        | What it means                                   | Type-safe? | Use case                                      |
|-------------|--------------------------------------------------|------------|-----------------------------------------------|
| `any`       | Completely disables type checking               | ❌ No      | Quick JS → TS migration, truly unknown input  |
| `unknown`   | Could be anything, but **must** check before use | ✅ Yes     | External data (APIs, user input)              |
| `never`     | This code path should **never** be reached      | ✅ Yes     | Functions that throw / exhaustive checks       |
| `undefined` | Variable declared but not yet assigned          | ✅ Yes     | Optional function params, uninitialized vars  |
| `null`      | Intentionally empty / no value                  | ✅ Yes     | Resetting values, optional object properties  |

---

## `any` — Skip type checking entirely

```ts
let value: any = 42;
value = "hello";       // ✅ no error
value = true;          // ✅ no error
value.doAnything();    // ✅ no error — TypeScript trusts you blindly
```

> **Avoid it.** Once you use `any`, TypeScript can't help you catch bugs.

---

## `unknown` — Safe version of `any`

```ts
let input: unknown = "hello world";

// ❌ Can't use directly
console.log(input.toUpperCase()); // Error: Object is of type 'unknown'

// ✅ Must check type first
if (typeof input === "string") {
  console.log(input.toUpperCase()); // "HELLO WORLD"
}
```

> Use `unknown` instead of `any` when data comes from outside (API response, user input).

---

## `never` — This should never happen

```ts
// 1. Function that always throws — never returns a value
function crash(msg: string): never {
  throw new Error(msg);
}

// 2. Exhaustive switch — catches missing cases at compile time
type Color = "red" | "green" | "blue";

function getHex(color: Color): string {
  switch (color) {
    case "red":   return "#FF0000";
    case "green": return "#00FF00";
    case "blue":  return "#0000FF";
    default:
      const check: never = color; // ❌ Error if you add a new Color but forget a case
      return check;
  }
}
```

> `never` acts as a compile-time safety net for exhaustive logic.

---

## `undefined` — Not yet assigned

```ts
let name: string | undefined;
console.log(name); // undefined — declared but not set

// Optional parameter (implicitly string | undefined)
function greet(user?: string) {
  return `Hello, ${user ?? "stranger"}`;
}

greet("Alice"); // "Hello, Alice"
greet();        // "Hello, stranger"
```

---

## `null` — Intentionally empty

```ts
let selectedUser: string | null = null; // nothing selected yet

selectedUser = "Alice"; // user picks something
selectedUser = null;    // user clears selection
```

---

## `undefined` vs `null` — at a glance

```ts
let a: undefined = undefined; // "I forgot to set this"
let b: null = null;           // "I deliberately set this to nothing"

console.log(typeof a); // "undefined"
console.log(typeof b); // "object"  ← quirk of JavaScript
```

---

## Strict Null Checks (recommended)

Add to `tsconfig.json` to make TypeScript enforce `null`/`undefined` handling:

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

With this on, you **must** handle both `null` and `undefined` explicitly — TypeScript won't let them sneak in silently.
