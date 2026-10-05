# TypeScript Declaration Merging

Multiple declarations with the same name = TypeScript merges them into one.

---

## Interface Merging

Same interface name declared twice — TS combines all properties:

```ts
interface Person { name: string; age: number; }
interface Person { address: string; email: string; }

// TS merges to:
// { name: string; age: number; address: string; email: string; }

const person: Person = {
  name: "John",
  age: 30,
  address: "123 Main St",
  email: "john@example.com"
}; // ✅ must provide all 4 fields
```

Only `interface` supports this — `type` alias does NOT:

```ts
type User = { name: string };
type User = { age: number };  // ❌ Error: duplicate identifier
```

---

## Namespace Merging

Namespaces with the same name merge their exports:

```ts
namespace App {
  export const version = "1.0";
}

namespace App {
  export function start() { console.log("Starting v" + version); }
}

App.version; // ✅
App.start();  // ✅
```

---

## Function + Namespace Merging

Add static properties to a function:

```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

namespace greet {
  export const defaultName = "World";
  export function formal(name: string) { return `Good day, ${name}.`; }
}

greet("Alice");              // ✅ "Hello, Alice!"
greet.defaultName;           // ✅ "World"
greet.formal("Bob");         // ✅ "Good day, Bob."
```

---

## Class + Namespace Merging

Add static members to a class via namespace:

```ts
class Point {
  constructor(public x: number, public y: number) {}
}

namespace Point {
  export const origin = new Point(0, 0);
  export function distance(p1: Point, p2: Point) {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
  }
}

const p = new Point(3, 4);
Point.origin;                      // ✅ { x: 0, y: 0 }
Point.distance(p, Point.origin);   // ✅ 5
```

---

## Module Augmentation — Most Common Real Use

Extend types from a third-party package without modifying it:

```ts
// Extend Express Request (common pattern)
declare module "express" {
  interface Request {
    user?: { id: number; role: string };
    requestId?: string;
  }
}

// Now req.user is typed everywhere
app.use((req, res, next) => {
  req.user = { id: 1, role: "admin" }; // ✅ typed
  next();
});
```

---

## Global Augmentation

Add to the global scope:

```ts
declare global {
  interface Window {
    analytics: { track: (event: string) => void };
  }

  interface Array<T> {
    last(): T | undefined;
  }
}

// Now available everywhere
window.analytics.track("page_view");
[1, 2, 3].last(); // would be undefined unless you also implement it
```

---

## What Can Merge vs What Cannot

| Declaration type | Can merge? |
|---|---|
| `interface` | ✅ Yes — always |
| `namespace` | ✅ Yes |
| `function` + `namespace` | ✅ Yes |
| `class` + `namespace` | ✅ Yes |
| `type` alias | ❌ No — error |
| `class` + `class` | ❌ No — error |
| `enum` + `namespace` | ✅ Yes (add methods to enum) |

---

## Quick Reference

```ts
// Interface merging
interface A { x: number }
interface A { y: number }
// merged: { x: number; y: number }

// Module augmentation — extend 3rd party types
declare module "some-library" {
  interface SomeType { newProp: string; }
}

// Global augmentation
declare global {
  interface Window { myTool: () => void; }
}
```

---

## Rule of Thumb

```
Extend existing interface across files   →  interface merging
Extend 3rd party library types           →  declare module "lib"
Add to browser globals (window, Array)   →  declare global
Add static methods to function/class     →  function/class + namespace
```
