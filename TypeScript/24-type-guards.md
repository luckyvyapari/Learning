# TypeScript Type Guards

Narrow a broad type (union) down to a specific type inside a scope.

---

## 6 Type Guard Patterns

| Pattern | Best for |
|---|---|
| `typeof` | Primitives (string, number, boolean) |
| `instanceof` | Class instances |
| Custom predicate (`is`) | Complex object validation |
| Discriminated union | Objects with a shared literal field |
| `in` operator | Check if property exists |
| Assertion function (`asserts`) | Runtime validation that throws |

---

## `typeof` — Primitives

```ts
function formatValue(value: string | number): string {
  if (typeof value === "string") {
    return value.trim().toUpperCase(); // TS knows: string here
  } else {
    return value.toFixed(2);          // TS knows: number here
  }
}

formatValue("  hello  "); // "HELLO"
formatValue(42.1234);     // "42.12"
```

---

## `instanceof` — Classes

```ts
class Bird { fly()  { console.log("Flying"); } }
class Fish { swim() { console.log("Swimming"); } }

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly();  // TS knows: Bird
  } else {
    animal.swim(); // TS knows: Fish
  }
}
```

---

## Custom Type Predicate — `value is Type`

For objects where `typeof`/`instanceof` won't work:

```ts
interface Car        { make: string; model: string; }
interface Motorcycle { make: string; model: string; type: "sport" | "cruiser"; }

// Return type "vehicle is Car" = type predicate
function isCar(vehicle: Car | Motorcycle): vehicle is Car {
  return (vehicle as Motorcycle).type === undefined;
}

function showInfo(vehicle: Car | Motorcycle) {
  if (isCar(vehicle)) {
    console.log("Car");          // TS knows: Car
  } else {
    console.log(vehicle.type);  // TS knows: Motorcycle
  }
}
```

---

## Discriminated Union — Shared Literal Field

Most reliable pattern. Add a `kind`/`type` field to each variant:

```ts
interface Circle { kind: "circle"; radius: number; }
interface Square { kind: "square"; sideLength: number; }

type Shape = Circle | Square;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;  // TS knows: Circle
    case "square": return shape.sideLength ** 2;         // TS knows: Square
  }
}
```

---

## `in` Operator — Property Existence

```ts
interface Dog { bark(): void; }
interface Cat { meow(): void; }

function makeSound(animal: Dog | Cat) {
  if ("bark" in animal) {
    animal.bark(); // TS knows: Dog
  } else {
    animal.meow(); // TS knows: Cat
  }
}
```

---

## Assertion Functions — Throw on Wrong Type

```ts
// Narrows AND throws if wrong type
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") throw new Error("Not a string");
}

function processInput(input: unknown) {
  assertIsString(input);
  console.log(input.toUpperCase()); // ✅ TS knows: string after assertion
}
```

Generic assert:
```ts
function assert(condition: unknown, message: string): asserts condition {
  if (!condition) throw new Error(message);
}

function double(value: unknown): number {
  assert(typeof value === "number", "Must be a number");
  return value * 2; // TS knows: number
}
```

---

## When to Use Which

```
Primitive value?           →  typeof
Class instance?            →  instanceof
Object with unique prop?   →  in operator
Related objects + switch?  →  discriminated union (best)
Complex custom check?      →  type predicate (is)
Must throw if invalid?     →  assertion function (asserts)
```
