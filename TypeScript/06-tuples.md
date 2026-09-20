# TypeScript Tuples

## What is a Tuple?

An array with a **fixed number of elements** where **each position has a specific type**.

| Feature       | Array             | Tuple                      |
|---------------|-------------------|----------------------------|
| Length        | Any length        | Fixed length               |
| Types         | All same type     | Each position has own type |
| Use case      | List of same data | Structured pair/group      |

---

## Basic Tuple

```ts
// [string, number] — position 0 must be string, position 1 must be number
let person: [string, number] = ["Alice", 30];

console.log(person[0]); // "Alice" — inferred as string
console.log(person[1]); // 30      — inferred as number

person[0] = "Bob";  // ✅
person[0] = 42;     // ❌ Error: number not assignable to string
person[2] = "extra"; // ❌ Error: index 2 doesn't exist
```

---

## Named Tuples (TS 4.0+)

Labels make tuples self-documenting:

```ts
let employee: [name: string, age: number, isActive: boolean];
employee = ["Alice", 30, true]; // ✅

// Labels are just documentation — no effect on behavior
```

---

## Destructuring Tuples

```ts
const point: [number, number] = [10, 20];
const [x, y] = point;

console.log(x); // 10
console.log(y); // 20
```

```ts
// Real-world example: function returning multiple values
function getRange(arr: number[]): [number, number] {
  return [Math.min(...arr), Math.max(...arr)];
}

const [min, max] = getRange([3, 1, 7, 2, 9]);
console.log(min, max); // 1  9
```

---

## Optional Tuple Elements

```ts
let data: [string, number?];

data = ["Alice"];       // ✅ second element optional
data = ["Alice", 30];   // ✅ also fine
data = ["Alice", 30, true]; // ❌ Error: too many elements
```

---

## Rest Elements in Tuples

```ts
let record: [string, ...number[]];

record = ["scores", 10, 20, 30]; // ✅ one string, any number of numbers
```

---

## Readonly Tuples

```ts
const rgb: readonly [number, number, number] = [255, 128, 0];

rgb[0] = 100; // ❌ Error: cannot assign to read-only element
```

---

## Common Use Cases

```ts
// 1. Key-value pair
type Entry = [string, number];
const price: Entry = ["apple", 1.5];

// 2. Coordinates
type Point2D = [x: number, y: number];
type Point3D = [x: number, y: number, z: number];

// 3. useState-style (React)
function useState<T>(initial: T): [T, (val: T) => void] {
  let state = initial;
  const setState = (val: T) => { state = val; };
  return [state, setState];
}

const [count, setCount] = useState(0);
setCount(5);   // ✅
setCount("x"); // ❌ Error: string not assignable to number
```

---

## Tuple vs Array — at a glance

```ts
// Array: all same type, any length
let arr: number[] = [1, 2, 3, 4, 5];

// Tuple: fixed positions, mixed types
let tup: [string, number, boolean] = ["Alice", 30, true];
```
