# TypeScript Object Types

## Inline Object Type

```ts
const car: { type: string; model: string; year: number } = {
  type: "Toyota",
  model: "Corolla",
  year: 2009
};
```

---

## Type Inference on Objects

TypeScript infers property types from values — no annotation needed:

```ts
const car = {
  type: "Toyota",
};

car.type = "Ford"; // ✅ string → string fine
car.type = 2;      // ❌ Error: number not assignable to string
```

---

## Optional Properties (`?`)

| Syntax            | Required? | Error if missing? |
|-------------------|-----------|-------------------|
| `prop: string`    | Yes       | ✅ Error          |
| `prop?: string`   | No        | ❌ No error        |

```ts
// ❌ Without optional — missing property causes error
const car: { type: string; mileage: number } = {
  type: "Toyota", // Error: 'mileage' is missing
};

// ✅ With optional — no error
const car: { type: string; mileage?: number } = {
  type: "Toyota"
};

car.mileage = 2000; // can be set later
```

---

## Index Signatures

When you don't know the property names ahead of time:

```ts
const nameAgeMap: { [index: string]: number } = {};

nameAgeMap.Jack = 25;       // ✅
nameAgeMap.Mark = "Fifty";  // ❌ Error: string not assignable to number
```

General pattern:
```ts
{ [key: KeyType]: ValueType }
// KeyType can be: string | number | symbol
```

---

## Reusing Object Types

For reusable shapes use `type` alias or `interface` (covered in detail in interfaces section):

```ts
type Car = {
  type: string;
  model: string;
  year: number;
};

const car1: Car = { type: "Toyota", model: "Corolla", year: 2009 };
const car2: Car = { type: "Honda",  model: "Civic",   year: 2021 };
```

---

## All Patterns at a Glance

| Pattern              | Syntax                                     |
|----------------------|--------------------------------------------|
| Inline object type   | `const x: { a: string; b: number } = ...` |
| Optional property    | `{ name?: string }`                        |
| Index signature      | `{ [key: string]: number }`                |
| Reusable shape       | `type Obj = { ... }` or `interface Obj {}` |
| Inferred object type | `const x = { a: "hi" }` — TS infers       |
