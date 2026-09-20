# TypeScript Arrays

## Declaring Arrays

Two syntax options — both are equivalent:

```ts
// Syntax 1: type[]
let numbers: number[] = [1, 2, 3];
let names: string[]   = ["Alice", "Bob"];

// Syntax 2: Array<type>  (generic)
let scores: Array<number> = [100, 95, 88];
let flags: Array<boolean>  = [true, false, true];
```

---

## Array Methods (type-safe)

```ts
let fruits: string[] = ["apple", "banana", "mango"];

fruits.push("orange");       // ✅ OK
fruits.push(42);             // ❌ Error: number not assignable to string

let first = fruits[0];       // inferred: string
let len   = fruits.length;   // inferred: number
```

---

## Readonly Arrays

Prevents mutation — useful for constants/config:

```ts
const colors: readonly string[] = ["red", "green", "blue"];

colors.push("yellow"); // ❌ Error: Property 'push' does not exist on readonly array
colors[0] = "pink";    // ❌ Error: Index signature is read-only
```

---

## Mixed Types with Union

```ts
let mixed: (string | number)[] = ["Alice", 42, "Bob", 99];
```

---

## Array of Objects

```ts
interface User {
  name: string;
  age: number;
}

const users: User[] = [
  { name: "Alice", age: 30 },
  { name: "Bob",   age: 25 },
];

users.push({ name: "Charlie", age: 28 }); // ✅
users.push({ name: "Dan" });               // ❌ Error: missing 'age'
```

---

## Multidimensional Arrays

```ts
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
];

console.log(matrix[0][1]); // 2
```

---

## Useful Array Type Patterns

| Pattern                    | Syntax                        |
|----------------------------|-------------------------------|
| Number array               | `number[]`                    |
| String array (generic)     | `Array<string>`               |
| Readonly array             | `readonly string[]`           |
| Union type array           | `(string \| number)[]`        |
| Array of objects           | `User[]`                      |
| 2D array                   | `number[][]`                  |
| Empty (let TS infer later) | `let arr: string[] = []`      |
