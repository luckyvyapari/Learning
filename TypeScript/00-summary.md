# TypeScript — Complete Summary

Quick reference for all topics. Click file links for full notes.

---

## 01. What is TypeScript → [01-intro.md](01-intro.md)

- Superset of JavaScript with static typing
- Errors caught at **compile time**, not runtime
- Transpiles to plain JS — runs everywhere JS runs

```ts
npx tsc hello.ts   // compile
node hello.js      // run
```

---

## 02. Simple Types / Primitives → [02-simple-types.md](02-simple-types.md)

| Type | Example |
|------|---------|
| `boolean` | `let active: boolean = true` |
| `number` | `let score: number = 42` |
| `string` | `let name: string = "Alice"` |
| `bigint` | `const big = BigInt(9999999)` |
| `symbol` | `const id: symbol = Symbol("id")` |

---

## 03. Explicit vs Inference → [03-explicit-vs-inference.md](03-explicit-vs-inference.md)

```ts
let x: string = "hello";  // explicit
let y = "hello";           // inferred as string
```

- Function params/returns → always explicit
- Simple vars with obvious value → let TS infer
- Uninitialized vars → always explicit (else becomes `any`)

---

## 04. Special Types → [04-special-types.md](04-special-types.md)

| Type | Meaning | Safe? |
|------|---------|-------|
| `any` | Skip type checking | ❌ No |
| `unknown` | Could be anything, must check first | ✅ Yes |
| `never` | This path never runs | ✅ Yes |
| `undefined` | Declared, not assigned | ✅ Yes |
| `null` | Intentionally empty | ✅ Yes |

---

## 05. Arrays → [05-arrays.md](05-arrays.md)

```ts
let nums: number[] = [1, 2, 3];       // or Array<number>
let ro: readonly string[] = ["a"];    // read-only
let mix: (string | number)[] = [...]; // union array
```

---

## 06. Tuples → [06-tuples.md](06-tuples.md)

Fixed-length array where each position has its own type.

```ts
let person: [string, number] = ["Alice", 30];
const [name, age] = person; // destructuring
```

Array = same type, any length. Tuple = mixed types, fixed positions.

---

## 07. Object Types → [07-object-types.md](07-object-types.md)

```ts
const car: { type: string; year: number } = { type: "Toyota", year: 2021 };

// Optional property
const car: { type: string; mileage?: number } = { type: "Toyota" };

// Index signature (unknown keys)
const map: { [key: string]: number } = {};
```

---

## 08. Enums → [08-enums.md](08-enums.md)

```ts
enum Status { Active = 1, Inactive = 2 }         // numeric
enum Dir { North = "NORTH", South = "SOUTH" }     // string (preferred)
```

String enums preferred — readable in logs, no accidental numeric comparisons.

---

## 09. Type Aliases & Interfaces → [09-type-aliases-and-interfaces.md](09-type-aliases-and-interfaces.md)

| | `type` | `interface` |
|---|---|---|
| Objects | ✅ | ✅ |
| Unions `A \| B` | ✅ | ❌ |
| Extend | `&` operator | `extends` keyword |
| Declaration merging | ❌ | ✅ |

Rule: `interface` for object shapes, `type` for everything else.

---

## 10. Union Types → [10-union-types.md](10-union-types.md)

Value can be one of several types (OR logic).

```ts
let id: string | number;
type Status = "success" | "error" | "loading"; // literal union
```

Must narrow before using type-specific methods:
```ts
if (typeof value === "string") { value.toUpperCase(); }
```

---

## 11. Functions → [11-functions.md](11-functions.md)

| Param type | Syntax |
|---|---|
| Normal | `a: number` |
| Optional | `c?: number` |
| Default | `exp: number = 10` |
| Rest | `...rest: number[]` |
| Named | `{ a, b }: { a: number; b: number }` |

```ts
function greet(name: string): string { ... }  // explicit return type
function log(): void { ... }                   // no return
type Fn = (x: number) => number;              // function type alias
```

---

## 12. Casting → [12-casting.md](12-casting.md)

```ts
let x: unknown = "hello";
(x as string).length;      // as keyword (use this)
(<string>x).length;        // angle bracket (avoid in React)
((x as unknown) as number) // force cast — last resort
```

Casting changes TS's view, NOT runtime data.

---

## 13. Classes → [13-classes.md](13-classes.md)

| Modifier | Accessible from |
|---|---|
| `public` | Everywhere (default) |
| `private` | Class only |
| `protected` | Class + subclasses |

```ts
class Animal {
  constructor(public name: string, private age: number) {}
}
class Dog extends Animal { ... }          // extends
class Cat implements Speakable { ... }    // implements interface
abstract class Shape { abstract getArea(): number; } // abstract
```

---

## 14. Generics → [14-generics.md](14-generics.md)

`<T>` = type placeholder, decided at call time.

```ts
function getValue<T>(value: T): T { return value; }
getValue<string>("Hello");  // T = string
getValue<number>(123);      // T = number

interface ApiResponse<T> { success: boolean; data: T; }
class Box<T> { constructor(private value: T) {} }
type ApiResult<T> = { data: T; };
```

---

## 15. Utility Types → [15-utility-types.md](15-utility-types.md)

| Utility | What it does |
|---------|-------------|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties read-only |
| `Record<K,V>` | Object with typed keys + values |
| `Pick<T, Keys>` | Keep only listed keys |
| `Omit<T, Keys>` | Remove listed keys |
| `Exclude<T, U>` | Remove types from union |
| `ReturnType<T>` | Extract function return type |
| `Parameters<T>` | Extract function param types |

```ts
type UpdateUser = Partial<Omit<User, "id">>; // patch endpoint pattern
```
