# TypeScript Literal Types

Exact values a variable can hold — more precise than `string` or `number`.

---

## String Literals

```ts
let direction: "north" | "south" | "east" | "west";

direction = "north";     // ✅
direction = "northeast"; // ❌ Error
```

```ts
function move(dir: "north" | "south" | "east" | "west") { ... }
move("east"); // ✅
move("up");   // ❌
```

---

## Numeric Literals

```ts
let dice: 1 | 2 | 3 | 4 | 5 | 6;

dice = 3;   // ✅
dice = 0;   // ❌
dice = 2.5; // ❌
```

---

## Boolean Literals

```ts
type AlwaysTrue = true;

function check(): true { return true; } // must return literal true
```

Rarely used alone — useful combined:
```ts
type SuccessFlag = true | "success" | 1;
type FailureFlag = false | "failure" | 0;
```

---

## Literal Types in Objects

```ts
type HTTPSuccess = {
  status: 200 | 201 | 204;
  statusText: "OK" | "Created" | "No Content";
};

type HTTPError = {
  status: 400 | 401 | 403 | 404 | 500;
  statusText: "Bad Request" | "Unauthorized" | "Not Found";
  error: string;
};

type HTTPResponse = HTTPSuccess | HTTPError;
```

---

## Template Literal Types (TS 4.1+)

Combine literal unions into new string types:

```ts
type Direction = "north" | "south" | "east" | "west";
type Distance  = "1km" | "5km" | "10km";

type Route = `${Direction}-${Distance}`;
// "north-1km" | "north-5km" | ... | "west-10km"

let r: Route = "north-5km"; // ✅
let b: Route = "north-2km"; // ❌ not in union
```

String manipulation helpers:
```ts
type EventType   = "click" | "hover" | "scroll";
type HandlerName = `on${Capitalize<EventType>}`;
// "onClick" | "onHover" | "onScroll"
```

CSS units:
```ts
type CssUnit  = "px" | "em" | "rem" | "%" | "vh" | "vw";
type CssValue = `${number}${CssUnit}`;

let width: CssValue  = "100px"; // ✅
let height: CssValue = "50%";   // ✅
let bad: CssValue    = "10";    // ❌ no unit
```

API URL pattern:
```ts
type ApiVersion = "v1" | "v2" | "v3";
type Endpoint   = "users" | "products" | "orders";
type ApiUrl     = `https://api.example.com/${ApiVersion}/${Endpoint}`;
```

Extract route params with `infer`:
```ts
type ExtractParams<T extends string> =
  T extends `${string}:${infer P}/${infer Rest}`
    ? P | ExtractParams<Rest>
    : T extends `${string}:${infer P}`
    ? P
    : never;

type Params = ExtractParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"
```

---

## Literal vs Enum — When to Use Which

| | Literal union | Enum |
|---|---|---|
| Syntax | `"a" \| "b" \| "c"` | `enum X { A, B, C }` |
| JS output | None (erased) | Compiled to object |
| Reverse lookup | ❌ | ✅ (numeric enums) |
| Best for | Short fixed sets | Named constants, large sets |

---

## Quick Reference

```ts
// String literal
type Dir = "up" | "down" | "left" | "right";

// Numeric literal
type Dice = 1 | 2 | 3 | 4 | 5 | 6;

// Template literal
type CssVal = `${number}px`;
type Handler = `on${Capitalize<"click" | "blur">}`; // "onClick" | "onBlur"

// Narrow a const to literal
const env = "production" as const; // type: "production" not string
```
