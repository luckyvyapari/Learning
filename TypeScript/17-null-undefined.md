# TypeScript Null & Undefined

Enable strict handling in `tsconfig.json`:
```json
{ "compilerOptions": { "strictNullChecks": true } }
```

---

## Types

```ts
let value: string | undefined | null = null;
value = "hello";
value = undefined;
```

With `strictNullChecks` on — TS forces you to handle null/undefined explicitly.

---

## 4 Tools for Handling Null/Undefined

| Tool | Operator | What it does |
|------|----------|-------------|
| Optional chaining | `?.` | Access property safely — returns `undefined` if null |
| Nullish coalescing | `??` | Fallback only for `null`/`undefined` |
| Null assertion | `!` | Tell TS "trust me, not null" — unsafe |
| Type guard | `if (x !== null)` | Narrow explicitly — safest |

---

## `?.` — Optional Chaining

Access nested property that might not exist:

```ts
interface House {
  sqft: number;
  yard?: { sqft: number };
}

function printYardSize(house: House) {
  const yardSize = house.yard?.sqft; // undefined if no yard — no crash

  if (yardSize === undefined) {
    console.log("No yard");
  } else {
    console.log(`Yard is ${yardSize} sqft`);
  }
}

printYardSize({ sqft: 500 }); // "No yard"
```

---

## `??` — Nullish Coalescing

Fallback **only** when value is `null` or `undefined` (not `0`, `""`, `false`):

```ts
function printMileage(mileage: number | null | undefined) {
  console.log(`Mileage: ${mileage ?? "Not Available"}`);
}

printMileage(null);      // "Mileage: Not Available"
printMileage(undefined); // "Mileage: Not Available"
printMileage(0);         // "Mileage: 0"  ← 0 is valid, not replaced
```

`??` vs `||`:
```ts
0 ?? "default"   // 0      — ?? only replaces null/undefined
0 || "default"   // "default" — || replaces any falsy (0, "", false)
```

---

## `!` — Null Assertion

Tell TypeScript "this is definitely not null/undefined":

```ts
function getValue(): string | undefined {
  return "hello";
}

let value = getValue();
console.log(value!.length); // ✅ TS stops complaining
// ⚠️ Unsafe — if getValue() returns undefined, runtime crash
```

Use only when you're 100% sure. Prefer type guards instead.

---

## Type Guard — Safest Approach

```ts
let value: string | null = getValue();

if (value !== null) {
  console.log(value.toUpperCase()); // ✅ TS knows it's string here
}
```

---

## Quick Rule

```
?.   → safe access of maybe-null property
??   → fallback for null/undefined only (not 0 or "")
!    → override TS warning — use sparingly
if   → safest, TS narrows the type for you
```
