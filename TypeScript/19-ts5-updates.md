# TypeScript 5.x Updates

Features available in TS 5.x+ only.

---

## Template Literal Types

Create precise string types using template literal syntax — like template strings but for types.

```ts
type Color = "red" | "green" | "blue";
type HexColor<T extends Color> = `#${string}`;

let myColor: HexColor<"blue"> = "#0000FF"; // ✅
let bad: HexColor<"blue"> = "0000FF";      // ❌ must start with #
```

More examples:

```ts
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`; // "onClick" | "onFocus" | "onBlur"

type CssUnit = `${number}px` | `${number}rem` | `${number}%`;
let size: CssUnit = "16px";  // ✅
let bad: CssUnit  = "16";    // ❌
```

---

## Index Signature Labels

Label index signatures with a pattern — only keys matching the pattern allowed.

```ts
type DynamicObject = { [key: `dynamic_${string}`]: string };

let obj: DynamicObject = { dynamic_key: "value" };   // ✅
let bad: DynamicObject = { normal_key: "value" };     // ❌ must start with dynamic_
```

More patterns:

```ts
type ApiKeys  = { [key: `api_${string}`]: string };
type DataKeys = { [key: `data_${string}`]: number };
```

---

## Native Private Fields (`#`)

JavaScript native private — truly private at runtime (not just compile-time like TS `private`).

```ts
class Counter {
  #count = 0;           // JS native private — truly inaccessible outside

  increment() { this.#count++; }
  get value() { return this.#count; }
}

const c = new Counter();
c.increment();
console.log(c.value);   // ✅ 1
console.log(c.#count);  // ❌ SyntaxError — even at runtime
```

| | TS `private` | JS `#` |
|---|---|---|
| Enforced at | Compile time only | Compile + Runtime |
| JS output | Regular property | True private field |
| Accessible in JS | Yes (just TS warning) | No — hard error |

---

## Quick Summary

| Feature | Syntax | Use case |
|---|---|---|
| Template literal type | `` `prefix_${string}` `` | Constrain string shape |
| Index signature label | `[key: \`prefix_${string}\`]` | Only allow patterned keys |
| Native private | `#field` | True runtime privacy |
