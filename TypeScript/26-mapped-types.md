# TypeScript Mapped Types

Transform every property of a type in one go.

Syntax: `{ [P in keyof T]: ... }`

---

## Basic Syntax

```ts
type Person = { name: string; age: number };

// Make all optional
type PartialPerson = { [P in keyof Person]?: Person[P] };

// Make all readonly
type ReadonlyPerson = { readonly [P in keyof Person]: Person[P] };
```

`P` = each property key, `keyof Person` = `"name" | "age"`, `Person[P]` = value type at that key.

---

## Built-in Mapped Types (recap)

| Utility | What it does |
|---|---|
| `Partial<T>` | All props optional |
| `Required<T>` | All props required |
| `Readonly<T>` | All props readonly |
| `Pick<T, K>` | Keep only listed keys |
| `Omit<T, K>` | Remove listed keys |
| `Record<K, V>` | Map key set to value type |

```ts
interface User { id: number; name: string; email: string; isAdmin: boolean; }

type PartialUser     = Partial<User>;               // all optional
type ReadonlyUser    = Readonly<User>;               // all readonly
type UserCredentials = Pick<User, "email" | "id">;  // { email, id }
type PublicUser      = Omit<User, "id" | "isAdmin">; // { name, email }
type UserRoles       = Record<"admin" | "user" | "guest", string>;
```

---

## Custom Mapped Types

Convert all property types to string:
```ts
type Stringify<T> = { [P in keyof T]: string };

type StringProduct = Stringify<{ id: number; name: string; price: number }>;
// { id: string; name: string; price: string }
```

Validator shape from type:
```ts
type Validator<T> = { [P in keyof T]: (value: T[P]) => boolean };

const validate: Validator<{ id: number; name: string }> = {
  id:   (v) => v > 0,
  name: (v) => v.length > 0,
};
```

---

## Add / Remove Modifiers

Use `+` (add) or `-` (remove) before `readonly` and `?`:

```ts
// Remove readonly — make mutable
type Mutable<T> = { -readonly [P in keyof T]: T[P] };

// Remove optional — make all required
type RequiredProps<T> = { [P in keyof T]-?: T[P] };

// Add optional + readonly
type FrozenPartial<T> = { +readonly [P in keyof T]+?: T[P] };
```

| Modifier syntax | Effect |
|---|---|
| `readonly` / `+readonly` | Add readonly |
| `-readonly` | Remove readonly |
| `?` / `+?` | Make optional |
| `-?` | Make required |

---

## Combine with Conditional Types

Transform property types conditionally:
```ts
// Convert number props to string, leave others unchanged
type FormatNumbers<T> = {
  [P in keyof T]: T[P] extends number ? string : T[P];
};

type Formatted = FormatNumbers<{ data: unknown; status: number; message: string }>;
// { data: unknown; status: string; message: string }
```

Filter: keep only string properties:
```ts
type StringPropsOnly<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P];
};

type R = StringPropsOnly<{ id: number; name: string; email: string }>;
// { name: string; email: string }
```

`as` clause renames/filters keys — `never` drops the key.

---

## Quick Reference

```ts
// Transform all values
type ToNull<T>    = { [P in keyof T]: null };

// Conditional transform
type Nullify<T>   = { [P in keyof T]: T[P] extends string ? null : T[P] };

// Filter keys (as + never)
type OnlyStrings<T> = { [P in keyof T as T[P] extends string ? P : never]: T[P] };

// Remove readonly
type Mutable<T>   = { -readonly [P in keyof T]: T[P] };

// Remove optional
type Complete<T>  = { [P in keyof T]-?: T[P] };
```
