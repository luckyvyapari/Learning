# TypeScript `keyof`

Extracts the **keys** of an object type as a union.

---

## Basic Usage

```ts
interface Person {
  name: string;
  age: number;
}

type PersonKeys = keyof Person; // "name" | "age"
```

---

## `keyof` in a Function

Ensures the property argument is a valid key of the object:

```ts
function printProperty(person: Person, property: keyof Person) {
  console.log(`${property}: "${person[property]}"`);
}

printProperty({ name: "Max", age: 27 }, "name"); // ✅ "name: Max"
printProperty({ name: "Max", age: 27 }, "email"); // ❌ Error: not a key of Person
```

---

## With Generics — Generic Property Getter

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };

getProperty(user, "name");  // ✅ returns string
getProperty(user, "age");   // ✅ returns number
getProperty(user, "email"); // ❌ Error: not a key of user
```

`K extends keyof T` = K must be a valid key of T.
`T[K]` = return type is the value type at that key.

---

## `keyof` on Index Signatures

```ts
type StringMap = { [key: string]: number };
type K = keyof StringMap; // string | number
// (number because JS converts numeric keys to strings)
```

---

## Quick Summary

| Expression | Result |
|---|---|
| `keyof Person` | `"name" \| "age"` |
| `keyof { [key: string]: number }` | `string \| number` |
| `K extends keyof T` | K must be a key of T |
| `T[K]` | Type of value at key K in T |
