# TypeScript Utility Types

Built-in types that transform existing types. No need to write from scratch.

---

## All Utility Types at a Glance

| Utility | What it does |
|---------|-------------|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties read-only |
| `Record<K, V>` | Object with key type K, value type V |
| `Pick<T, Keys>` | Keep only specified keys |
| `Omit<T, Keys>` | Remove specified keys |
| `Exclude<T, U>` | Remove types from a union |
| `ReturnType<T>` | Extract return type of a function |
| `Parameters<T>` | Extract parameter types of a function |

---

## `Partial<T>` — Make All Optional

```ts
interface Point { x: number; y: number; }

let p: Partial<Point> = {}; // ✅ x and y both optional now
p.x = 10;
```

Useful for update/patch operations — don't need to pass all fields.

---

## `Required<T>` — Make All Required

```ts
interface Car {
  make: string;
  model: string;
  mileage?: number; // optional
}

let car: Required<Car> = {
  make: "Ford",
  model: "Focus",
  mileage: 12000 // ✅ now required — can't skip it
};
```

---

## `Readonly<T>` — Freeze All Properties

```ts
interface Person { name: string; age: number; }

const person: Readonly<Person> = { name: "Dylan", age: 35 };

person.name = "Israel"; // ❌ Error: cannot assign to read-only property
```

---

## `Record<K, V>` — Key-Value Object Type

```ts
const nameAgeMap: Record<string, number> = {
  Alice: 21,
  Bob: 25
};

// Same as: { [key: string]: number }
```

---

## `Pick<T, Keys>` — Keep Only These Keys

```ts
interface Person { name: string; age: number; location?: string; }

const bob: Pick<Person, "name"> = {
  name: "Bob"
  // age and location don't exist here — picked them out
};
```

---

## `Omit<T, Keys>` — Remove These Keys

```ts
interface Person { name: string; age: number; location?: string; }

const bob: Omit<Person, "age" | "location"> = {
  name: "Bob"
  // age and location removed — can't add them
};
```

`Pick` = keep listed. `Omit` = remove listed. Opposite of each other.

---

## `Exclude<T, U>` — Remove from Union

```ts
type Primitive = string | number | boolean;

const value: Exclude<Primitive, string> = true; // ✅
// string removed from union — only number | boolean remain
```

---

## `ReturnType<T>` — Extract Function Return Type

```ts
type PointGenerator = () => { x: number; y: number };

const point: ReturnType<PointGenerator> = { x: 10, y: 20 }; // ✅
```

Useful when you don't control the function but need its return type.

---

## `Parameters<T>` — Extract Function Parameter Types

```ts
type PointPrinter = (p: { x: number; y: number }) => void;

const point: Parameters<PointPrinter>[0] = { x: 10, y: 20 };
// [0] = first parameter's type
```

---

## Pick vs Omit — Quick Comparison

```ts
interface User { id: number; name: string; email: string; password: string; }

// Pick — choose what to KEEP
type PublicUser = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit — choose what to REMOVE
type SafeUser = Omit<User, "password">;
// { id: number; name: string; email: string }
```

---

## Real-World Combinations

```ts
// Patch endpoint — all fields optional
type UpdateUser = Partial<Omit<User, "id">>;

// Config that can't be changed after load
type Config = Readonly<Record<string, string>>;
```
