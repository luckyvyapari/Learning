# TypeScript Best Practices

---

## tsconfig — Enable Strict Mode

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

`"strict": true` enables all strict checks in one flag.

---

## Type System

| Rule | Bad | Good |
|---|---|---|
| Simple vars | `const x: string = "hi"` | `const x = "hi"` — let TS infer |
| Function params | `function f(u)` | `function f(u: User)` — always explicit |
| Avoid `any` | `value: any` | `value: unknown` + narrow |
| Objects | `type x = {...}` for unions | `interface` for object shapes |

```ts
// any — unsafe
function log(v: any) { v.toUpperCase(); } // no error at compile time

// unknown — safe
function log(v: unknown) {
  if (typeof v === "string") v.toUpperCase(); // ✅ narrowed
}

// generics — flexible + safe
function log<T extends { name: string }>(items: T[]) {
  return items.map(i => i.name);
}
```

---

## interface vs type — Quick Rule

```
Object shape → interface
Union / tuple / mapped type → type
```

```ts
interface User { id: number; name: string; }
interface AdminUser extends User { permissions: string[]; }

type Role = "admin" | "editor" | "viewer";
type UserId = number | string;
type Point = [number, number];
```

---

## Functions

```ts
// Default params instead of conditionals
function createUser(name: string, role: Role = "viewer"): User { ... }

// Rest params
function sum(...nums: number[]): number { return nums.reduce((a, b) => a + b, 0); }

// Typed callback param
function process(user: User, notify: (msg: string) => void): void { ... }
```

Single responsibility — split big functions:
```ts
// ❌ does too much
function processUserData(data: any) { validate(); transform(); save(); notify(); }

// ✅ focused
function validate(data: unknown): UserData { ... }
function transform(data: UserData): ProcessedUser { ... }
```

---

## Async

```ts
// Generic fetch with proper types
async function fetchData<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as T;
}

// Parallel over sequential
const [user, orders] = await Promise.all([getUser(id), getOrders(id)]);

// Flat over nested
async function load(id: string) {
  const user = await getUser(id);
  if (!user) return null;

  const orders = await getOrders(user.id);
  if (!orders.length) return { user, items: [] };

  return { user, items: await getItems(orders[0].id) };
}
```

---

## `as const` — Lock to Literal Types

```ts
const colors = ["red", "green", "blue"] as const;
// type: readonly ["red", "green", "blue"]

type Color = typeof colors[number]; // "red" | "green" | "blue"

const config = { url: "https://api.com", timeout: 5000 } as const;
// all values are readonly literals
```

---

## Type-Only Imports

```ts
// ✅ Separate type imports — don't ship types to runtime bundle
import type { User } from "./types";
import { fetchUser } from "./api";

export type { User };  // type-only export
```

---

## Null/Undefined Handling

```ts
// ❌ Possible runtime crash
function getLen(s: string | null) { return s.length; }

// ✅ Guard
function getLen(s: string | null) { return s?.length ?? 0; }

// Nested optional chaining
const name = user?.profile?.name ?? "Anonymous";
```

---

## Dependency Injection = Testable Code

```ts
// ❌ Hard to test — direct dependency
class PaymentProcessor {
  async pay(amount: number) { return new PaymentGateway().charge(amount); }
}

// ✅ Inject interface — easy to mock in tests
interface Gateway { charge(amount: number): Promise<boolean>; }

class PaymentProcessor {
  constructor(private gateway: Gateway) {}
  async pay(amount: number): Promise<boolean> {
    if (amount <= 0) throw new Error("Amount must be > 0");
    return this.gateway.charge(amount);
  }
}
```

---

## Quick Checklist

```
✅ strict: true in tsconfig
✅ Never use any — use unknown + narrow
✅ Explicit types on function params
✅ Let TS infer simple variable types
✅ interface for objects, type for unions
✅ as const for config/enum-like arrays
✅ import type for type-only imports
✅ Always handle null/undefined
✅ Async: try/catch + re-throw, Promise.all for parallel
✅ Single responsibility functions
✅ Inject dependencies for testability
```
