# TypeScript Type Aliases & Interfaces

## The Simple Idea

Both let you **name a shape/type** so you can reuse it.

```ts
// Without alias — repetitive
const user1: { name: string; age: number } = { name: "Alice", age: 30 };
const user2: { name: string; age: number } = { name: "Bob",   age: 25 };

// With alias — reusable
type User = { name: string; age: number };
const user1: User = { name: "Alice", age: 30 };
const user2: User = { name: "Bob",   age: 25 };
```

---

## `type` — Type Alias

Can name **anything**: primitives, objects, arrays, unions.

```ts
// Primitive alias
type Age  = number;
type Name = string;

// Object alias
type Car = {
  year: number;
  brand: string;
  model: string;
};

const car: Car = { year: 2021, brand: "Toyota", model: "Corolla" };
```

---

## `interface` — Interface

Only for **object shapes**. Looks slightly cleaner for objects.

```ts
interface Rectangle {
  height: number;
  width: number;
}

const rect: Rectangle = { height: 20, width: 10 };
```

---

## type vs interface — Side by Side

| Feature                  | `type`                        | `interface`                    |
|--------------------------|-------------------------------|--------------------------------|
| Objects                  | ✅ Yes                        | ✅ Yes                          |
| Primitives               | ✅ `type Age = number`        | ❌ No                           |
| Arrays                   | ✅ `type IDs = number[]`      | ❌ No                           |
| Union types              | ✅ `type X = A \| B`          | ❌ No                           |
| Intersection             | ✅ `type X = A & B`           | ❌ No                           |
| Extending                | ✅ `type X = A & { extra }` | ✅ `extends` keyword            |
| Declaration merging      | ❌ No                         | ✅ Yes (add props later)        |
| Classes can implement    | ✅ Yes                        | ✅ Yes                          |

---

## Extending — Making Bigger Types from Smaller Ones

### Interface `extends`

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {   // Dog gets everything Animal has + more
  breed: string;
}

const dog: Dog = { name: "Rex", breed: "Labrador" }; // ✅ needs both
```

### Type intersection `&`

```ts
type Animal = { name: string };
type Bear   = Animal & { honey: boolean }; // Bear = Animal + honey

const bear: Bear = { name: "Winnie", honey: true }; // ✅
```

Both do the same thing — just different syntax.

---

## Union Types (only `type` can do this)

```ts
type Status = "success" | "error" | "loading";

let response: Status = "success"; // ✅
response = "error";               // ✅
response = "done";                // ❌ Error: not in the union
```

---

## Declaration Merging (only `interface` can do this)

```ts
// Define Animal once...
interface Animal { name: string; }

// ...then add more properties later
interface Animal { age: number; }

// TypeScript merges them automatically
const dog: Animal = { name: "Fido", age: 5 }; // ✅ needs both
```

With `type` this would throw an error — you can't redefine a type alias.

---

## Which One to Use?

```
Object shape for a class / public API  →  interface
Everything else (union, primitives, arrays, intersections)  →  type
Not sure?  →  type works for almost everything
```

### Real examples:

```ts
// Use interface — it's an object shape
interface UserProfile {
  id: number;
  name: string;
  email: string;
}

// Use type — it's a union
type ButtonVariant = "primary" | "secondary" | "danger";

// Use type — combining two shapes
type AdminUser = UserProfile & { permissions: string[] };
```

---

## Full Comparison Example

```ts
// ---- type ----
type Point = { x: number; y: number };
type Point3D = Point & { z: number };

const p: Point3D = { x: 1, y: 2, z: 3 };

// ---- interface ----
interface Shape { color: string; }
interface Square extends Shape { sideLength: number; }

const sq: Square = { color: "red", sideLength: 10 };
```

Both work. Pick based on the table above.
