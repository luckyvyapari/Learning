# TypeScript Index Signatures

When you know the **type** of keys/values but not the exact property names.

Syntax: `{ [key: KeyType]: ValueType }`

---

## String Index Signature

```ts
interface StringDictionary {
  [key: string]: string;
}

const names: StringDictionary = {
  firstName: "Alice",
  lastName: "Smith",
};

names["city"] = "Delhi";  // ✅ dynamic add
names["age"]  = 30;       // ❌ Error: number not assignable to string
```

---

## Number Index Signature

```ts
interface NumberDictionary {
  [index: number]: any;
}

const scores: NumberDictionary = {
  0: "Zero",
  1: 100,
};
```

> JS stores all keys as strings internally, but TS treats number indexes separately to help catch array vs object mistakes.

---

## Mixed: Explicit Props + Index Signature

Known properties + dynamic ones together:

```ts
interface UserInfo {
  name: string;            // required, fixed
  age: number;             // required, fixed
  [key: string]: string | number; // all other keys
}

const user: UserInfo = {
  name: "Alice",
  age: 30,
  address: "123 Main St", // ✅ dynamic string
  zipCode: 12345,         // ✅ dynamic number
  isAdmin: true,          // ❌ boolean not in string | number
};
```

**Rule:** Explicit property types must be assignable to the index signature value type.

---

## Readonly Index Signature

```ts
interface ReadOnlyArray {
  readonly [index: number]: string;
}

const names: ReadOnlyArray = ["Alice", "Bob"];
names[0];        // ✅ read
names[0] = "X";  // ❌ Error: only permits reading
```

---

## Common Pitfall — Type Conflict

```ts
// ❌ Error — name: string conflicts with [key: string]: number
interface Bad {
  [key: string]: number;
  name: string;
}

// ✅ Fix — widen index type to include string
interface Good {
  [key: string]: number | string;
  name: string;   // string is part of number | string
  age: number;    // number is part of number | string
}
```

---

## Index Signature vs `Record<K, V>`

| | Index Signature | `Record<K, V>` |
|---|---|---|
| Syntax | `{ [key: string]: V }` | `Record<string, V>` |
| Mix with explicit props | ✅ Yes | ❌ No |
| Simple mapping | Verbose | ✅ Cleaner |

```ts
// Same result — Record is shorter for simple cases
interface StringMap  { [key: string]: string }
type    StringRecord = Record<string, string>;

// Use index signature when mixing with known props
interface Config {
  version: number;
  [key: string]: string | number; // plus dynamic keys
}
```

---

## Real Use: API Response

```ts
interface ApiResponse<T> {
  data: { [resourceType: string]: T[] };
  meta: {
    page: number;
    total: number;
    [key: string]: any; // allow extra metadata
  };
}

const res: ApiResponse<{ id: number; name: string }> = {
  data: { users: [{ id: 1, name: "Alice" }] },
  meta: { page: 1, total: 1, timestamp: "2024-01-01" }
};
```

---

## Quick Rule

```
Keys known in advance?        →  explicit interface properties
Keys dynamic, values same?    →  index signature  { [key: string]: V }
Simple key → value map?       →  Record<K, V>
Need explicit + dynamic both? →  mix index sig + explicit props
```
