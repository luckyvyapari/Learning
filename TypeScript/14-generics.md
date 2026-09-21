# TypeScript Basic Generics

Generics = reusable code jo multiple data types ke saath kaam kare.

---

## Problem Without Generics

```ts
function getValue(value: string): string {
  return value;
}
// sirf string accept karega — number/boolean pass karo to error
```

---

## Solution: Generics `<T>`

`T` ek placeholder type hai — call karte waqt decide hota hai.

```ts
function getValue<T>(value: T): T {
  return value;
}

getValue<string>("Hello");  // ✅ T = string
getValue<number>(123);      // ✅ T = number
getValue<boolean>(true);    // ✅ T = boolean
```

---

## Real Example — Array

```ts
function firstElement<T>(arr: T[]): T {
  return arr[0];
}

const firstNumber = firstElement([1, 2, 3]);   // T inferred as number
const firstString = firstElement(["A", "B"]);  // T inferred as string

console.log(firstNumber); // 1
console.log(firstString); // "A"
```

TypeScript automatically type infer kar leta hai — `<number>` explicitly likhna zaruri nahi.

---

## Generic Interface

```ts
interface ApiResponse<T> {
  success: boolean;
  data: T;
}

const userResponse: ApiResponse<{ id: number; name: string }> = {
  success: true,
  data: { id: 1, name: "Lucky" }
};
```

---

## Generic Class

```ts
class Box<T> {
  constructor(private value: T) {}

  getValue(): T {
    return this.value;
  }
}

const stringBox = new Box<string>("Hello");
const numberBox = new Box<number>(100);

console.log(stringBox.getValue()); // "Hello"
console.log(numberBox.getValue()); // 100
```

---

## Laravel/PHP vs TypeScript

| PHP | TypeScript |
|-----|------------|
| `mixed $data` | `data: T` |
| No type safety | Full type safety |
| One class for all types | One generic class for all types |

```php
// PHP
class ApiResponse {
  public function __construct(public mixed $data) {}
}
```

```ts
// TypeScript — same idea, type-safe
class ApiResponse<T> {
  constructor(public data: T) {}
}
```

---

## 4 Common Generic Patterns

```ts
// Function
function getData<T>(data: T): T { ... }

// Class
class Box<T> { ... }

// Interface
interface Response<T> { success: boolean; data: T; }

// Type alias
type ApiResult<T> = { data: T; };
```

---

## Simple Rule

```
<T> = "Type baad mein decide karenge"

Jaise Laravel me mixed use karte ho —
TypeScript me <T> se type safety bhi milti hai aur reusability bhi.
```
