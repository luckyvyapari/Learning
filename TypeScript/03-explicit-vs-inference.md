# Explicit Types vs Type Inference

## Quick Comparison

| Approach       | You write the type? | When to use                                          |
|----------------|---------------------|------------------------------------------------------|
| Explicit       | Yes                 | Function params/returns, object literals, unclear init|
| Type Inference | No (TS figures it out) | Simple vars with immediate assignment               |

---

## Explicit Type Annotations

You tell TypeScript exactly what type to expect.

```ts
let greeting: string  = "Hello, TypeScript!";
let userCount: number = 42;
let isLoading: boolean = true;
let scores: number[]  = [100, 95, 98];
```

**Function with explicit types:**
```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

greet("Alice"); // ✅ OK
greet(42);      // ❌ Error: Argument of type 'number' is not assignable to type 'string'
```

---

## Type Inference

TypeScript figures out the type from the assigned value — no annotation needed.

```ts
let username = "alice";        // inferred: string
let score    = 100;            // inferred: number
let flags    = [true, false];  // inferred: boolean[]

function add(a: number, b: number) {
  return a + b; // inferred return type: number
}
```

**Object inference:**
```ts
const user = {
  name: "Alice",
  age: 30,
  isAdmin: true
};

console.log(user.name);  // ✅ OK
console.log(user.email); // ❌ Error: Property 'email' does not exist
```

---

## When Inference Falls Back to `any`

Uninitialized variables without an annotation get type `any`:

```ts
let value;        // type: any  ❌ unsafe
value = "hello";  // still any
value = 42;       // still any

let safe: string; // type: string  ✅ explicit
safe = 42;        // ❌ Error caught immediately
```

---

## Rule of Thumb

```
Function parameters / return types  →  always explicit
Simple variable with obvious value  →  let inference work
Uninitialized variable              →  always explicit
```
