# TypeScript Casting

## What is Casting?

Overriding a variable's type — telling TypeScript "trust me, this is X".

Casting changes **TypeScript's view** of the type, NOT the actual runtime data.

---

## 3 Ways to Cast

| Method         | Syntax                    | Works in TSX/React? |
|----------------|---------------------------|---------------------|
| `as` keyword   | `x as string`             | ✅ Yes               |
| Angle brackets | `<string>x`               | ❌ No (use `as`)     |
| Force cast     | `(x as unknown) as number`| ✅ Yes               |

---

## `as` — Main Way

```ts
let x: unknown = "hello";

console.log((x as string).length); // ✅ 5
```

---

## `<>` — Same Thing, Different Syntax

```ts
let x: unknown = "hello";

console.log((<string>x).length); // ✅ 5
```

Avoid in React files — JSX parser confuses `<string>` with a JSX tag.

---

## Casting Doesn't Change Runtime Data

```ts
let x: unknown = 4;

console.log((x as string).length); // ⚠️ undefined — 4 is still a number at runtime
```

TypeScript trusts you. If you lie about the type, bugs happen at runtime.

---

## TypeScript Blocks Nonsense Casts

```ts
console.log((4 as string).length);
// ❌ Error: Conversion of type 'number' to type 'string' may be a mistake
//    because neither type sufficiently overlaps with the other.
```

TypeScript catches casts that make no sense.

---

## Force Cast — Override Everything

Cast to `unknown` first, then to target type. Bypasses TypeScript's overlap check.

```ts
let x = "hello";

console.log(((x as unknown) as number).length);
// ⚠️ No TS error, but returns undefined — x is still a string at runtime
```

Use only when you **know** TypeScript is wrong (e.g. a badly typed library).

---

## When to Actually Use Casting

```ts
// 1. API response typed as unknown
const data: unknown = await fetchUser();
const user = data as { name: string; age: number };

// 2. DOM elements
const input = document.getElementById("email") as HTMLInputElement;
console.log(input.value); // ✅ .value exists on HTMLInputElement

// 3. Fixing bad library types
const result = badLibraryFunction() as string;
```

---

## Quick Rule

```
Prefer unknown → narrow with typeof/instanceof instead of casting
Cast with as   → when you're sure TypeScript is wrong
Force cast     → last resort only
```
