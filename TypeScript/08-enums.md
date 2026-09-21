# TypeScript Enums

A named group of **constants** — values that never change.

## Enum Flavors

| Flavor          | Values     | When to use                          |
|-----------------|------------|--------------------------------------|
| Numeric default | 0, 1, 2…   | Simple ordered sets                  |
| Numeric init    | 1, 2, 3…   | When you need a specific start value |
| Fully custom    | 404, 200…  | HTTP codes, fixed IDs                |
| String          | "North"…   | Readability, intent, debugging       |

---

## Numeric Enum — Default (starts at 0)

```ts
enum CardinalDirections {
  North, // 0
  East,  // 1
  South, // 2
  West   // 3
}

let dir = CardinalDirections.North;
console.log(dir); // 0

dir = 'North'; // ❌ Error: string not assignable to CardinalDirections
```

---

## Numeric Enum — Custom Start

```ts
enum CardinalDirections {
  North = 1, // 1
  East,      // 2
  South,     // 3
  West       // 4
}

console.log(CardinalDirections.North); // 1
console.log(CardinalDirections.West);  // 4
```

---

## Numeric Enum — Fully Custom Values

```ts
enum StatusCodes {
  NotFound   = 404,
  Success    = 200,
  Accepted   = 202,
  BadRequest = 400
}

console.log(StatusCodes.NotFound); // 404
console.log(StatusCodes.Success);  // 200
```

> Values don't auto-increment when all are manually set.

---

## String Enum

```ts
enum CardinalDirections {
  North = "North",
  East  = "East",
  South = "South",
  West  = "West"
}

console.log(CardinalDirections.North); // "North"
console.log(CardinalDirections.West);  // "West"
```

**Why string enums are preferred:**
- Easier to read in logs/debuggers (`"North"` vs `0`)
- Intent is clear
- No accidental numeric comparisons

---

## Using Enums in Practice

```ts
enum Direction { Up = "UP", Down = "DOWN", Left = "LEFT", Right = "RIGHT" }

function move(dir: Direction) {
  console.log(`Moving ${dir}`);
}

move(Direction.Up);   // ✅ "Moving UP"
move("UP");           // ❌ Error: string not assignable to Direction
```

---

## Reverse Mapping (Numeric Enums Only)

```ts
enum Status { Active = 1, Inactive = 2 }

console.log(Status[1]); // "Active"  — reverse lookup
console.log(Status[2]); // "Inactive"
// String enums do NOT support this
```

---

## Quick Summary

```ts
// Default numeric
enum A { X, Y, Z }              // 0, 1, 2

// Initialized numeric
enum B { X = 10, Y, Z }         // 10, 11, 12

// Fully custom numeric
enum C { X = 100, Y = 200 }     // 100, 200

// String (most common in real apps)
enum D { X = "X", Y = "Y" }    // "X", "Y"
```
