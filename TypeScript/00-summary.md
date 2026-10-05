# TypeScript — Complete Quick Revision

One file. All 35 topics. Enough to revise everything before an interview or project.

---

## 01. What is TypeScript

- Superset of JS with **static typing** — errors caught at compile time, not runtime
- Transpiles to plain JS — runs everywhere JS runs
- `npx tsc hello.ts` → compiles | `node hello.js` → runs

```ts
function greet(name: string): string { return `Hello, ${name}!`; }
```

---

## 02. Simple Types (Primitives)

| Type | Example |
|---|---|
| `boolean` | `let active: boolean = true` |
| `number` | `let score: number = 42` |
| `string` | `let name: string = "Alice"` |
| `bigint` | `const big = BigInt(9999999)` |
| `symbol` | `const id: symbol = Symbol("id")` |

---

## 03. Explicit vs Inference

```ts
let x = "hello";          // inferred string
let y: string;             // explicit — always for uninitialized vars
function add(a: number, b: number): number { return a + b; } // explicit params
```

**Rule:** infer simple vars, always explicit on function params + empty arrays.

---

## 04. Special Types

| Type | Safe? | Use |
|---|---|---|
| `any` | ❌ | Disables type check entirely |
| `unknown` | ✅ | Must narrow before use |
| `never` | ✅ | Unreachable code / exhaustive checks |
| `undefined` | ✅ | Declared but not assigned |
| `null` | ✅ | Intentionally empty |

```ts
let v: unknown = getData();
if (typeof v === "string") v.toUpperCase(); // must narrow first
```

---

## 05. Arrays

```ts
let nums: number[] = [1, 2, 3];          // or Array<number>
let ro: readonly string[] = ["a", "b"];  // immutable
let mix: (string | number)[] = [1, "a"]; // union
let users: User[] = [];                   // always annotate empty arrays
```

---

## 06. Tuples

Fixed-length, each position has its own type.

```ts
let person: [string, number] = ["Alice", 30];
const [name, age] = person; // destructure
type Point = [x: number, y: number]; // named tuple
```

Array = same type, any length. Tuple = mixed types, fixed positions.

---

## 07. Object Types

```ts
const car: { type: string; year: number } = { type: "Toyota", year: 2021 };
const car2: { type: string; mileage?: number } = { type: "Honda" }; // optional ?
const map: { [key: string]: number } = {};                           // index sig
```

---

## 08. Enums

```ts
enum Dir { North = "NORTH", South = "SOUTH" }  // string (preferred)
enum Status { Active = 1, Inactive = 2 }         // numeric
```

String enums preferred — readable in logs, no accidental numeric comparisons.

---

## 09. Type Aliases & Interfaces

```ts
interface User { id: number; name: string; }        // extendable, mergeable
type Status = "active" | "inactive";                // unions, primitives
type AdminUser = User & { permissions: string[] };  // intersection

// interface → extends keyword
interface Admin extends User { role: string; }

// type → & operator
type Admin = User & { role: string };
```

**Rule:** `interface` for object shapes, `type` for everything else.

---

## 10. Union Types

```ts
let id: string | number;
type Status = "success" | "error" | "loading"; // literal union

// Must narrow before using type-specific methods
function format(v: string | number) {
  if (typeof v === "string") return v.toUpperCase();
  return v.toFixed(2);
}
```

---

## 11. Functions

```ts
function greet(name: string): string { ... }        // explicit return
function log(): void { ... }                         // no return
function add(a: number, b: number, c?: number) { }  // optional ?
function pow(v: number, exp: number = 10) { }       // default
function sum(...rest: number[]) { }                  // rest
type Fn = (x: number) => number;                    // type alias
```

---

## 12. Casting

```ts
let x: unknown = "hello";
(x as string).length;         // as keyword ✅
(<string>x).length;           // angle bracket (avoid in React)
((x as unknown) as number)    // force cast — last resort
```

Casting changes TS's view only. Runtime data unchanged.

---

## 13. Classes

```ts
class Animal {
  constructor(public name: string, private age: number) {}
}
class Dog extends Animal { ... }              // extends
class Cat implements Speakable { ... }        // implements interface
abstract class Shape { abstract area(): number; } // abstract

// override parent method
public override toString(): string { ... }
```

| Modifier | Accessible from |
|---|---|
| `public` | Everywhere |
| `private` | Class only |
| `protected` | Class + subclasses |

---

## 14. Generics

`<T>` = type placeholder decided at call time.

```ts
function getValue<T>(v: T): T { return v; }
getValue<string>("Hello");

interface ApiResponse<T> { success: boolean; data: T; }
class Box<T> { constructor(private v: T) {} getValue(): T { return this.v; } }
type Result<T> = { data: T; error?: string };
```

---

## 15. Utility Types

| Utility | Effect |
|---|---|
| `Partial<T>` | All optional |
| `Required<T>` | All required |
| `Readonly<T>` | All readonly |
| `Pick<T, K>` | Keep listed keys |
| `Omit<T, K>` | Remove listed keys |
| `Record<K, V>` | Key→value map |
| `Exclude<T, U>` | Remove from union |
| `NonNullable<T>` | Remove null/undefined |
| `ReturnType<T>` | Function return type |
| `Parameters<T>` | Function param types |

```ts
type UpdateUser = Partial<Omit<User, "id">>; // patch endpoint pattern
```

---

## 16. keyof

```ts
type PersonKeys = keyof Person; // "name" | "age"

function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // type-safe property access
}
```

---

## 17. Null & Undefined

```ts
let v: string | null = null;
v?.length;        // ?. — optional chaining, returns undefined if null
v ?? "default";   // ?? — fallback only on null/undefined (not 0 or "")
v!.length;        // ! — null assertion, unsafe

// ?? vs ||
0 ?? "x"   // 0    — ?? keeps 0
0 || "x"   // "x"  — || replaces any falsy
```

Enable: `"strictNullChecks": true` in tsconfig.

---

## 18. Definitely Typed

```bash
npm install --save-dev @types/jquery   # add types for untyped packages
npm install --save-dev @types/node
```

Type packages are `devDependency` — compile time only, not in bundle.

---

## 19. TypeScript 5.x Updates

```ts
// Template literal types
type CssVal = `${number}px`;
type Handler = `on${Capitalize<"click" | "blur">}`; // "onClick" | "onBlur"

// Index signature labels
type DynObj = { [key: `data_${string}`]: string };

// Native private fields (true runtime privacy)
class Counter { #count = 0; }  // # vs private — # enforced at runtime too
```

---

## 20. tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

## 21. TypeScript with Node.js

```bash
npm install typescript @types/node --save-dev
npm install --save-dev ts-node nodemon @types/express
```

```json
{ "dev": "nodemon --exec ts-node src/server.ts",
  "build": "tsc",  "start": "node dist/server.js" }
```

`ts-node` + `nodemon` = dev only. `tsc` + `node dist/` = production.

---

## 22. TypeScript with React

```ts
type Props = { name: string; age?: number };
function Greeting({ name, age }: Props) { ... }

const [user, setUser] = useState<User | null>(null);
const inputRef = useRef<HTMLInputElement>(null);

// Event types
(e: React.ChangeEvent<HTMLInputElement>) => e.target.value
(e: React.FormEvent<HTMLFormElement>) => e.preventDefault()

type CardProps = { title: string; children: ReactNode };
```

---

## 23. Tooling

```bash
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin prettier
```

```json
{ "lint": "eslint . --ext .ts,.tsx",
  "lint:fix": "eslint . --ext .ts,.tsx --fix",
  "type-check": "tsc --noEmit" }
```

---

## 24. Type Guards

```ts
typeof v === "string"           // primitives
v instanceof Bird               // class instances
"bark" in animal                // property existence
shape.kind === "circle"         // discriminated union ← best pattern
function isCar(v): v is Car {}  // custom predicate
function assertStr(v): asserts v is string {} // throws on wrong type
```

---

## 25. Conditional Types

```ts
type IsString<T> = T extends string ? true : false;

// infer — extract inner type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type ElementType<T> = T extends (infer U)[] ? U : never;

// Built-in conditional utilities
NonNullable<string | null>              // string
Extract<string | number, string>        // string
Exclude<string | number, string>        // number
```

---

## 26. Mapped Types

```ts
type Partial<T>  = { [P in keyof T]?: T[P] };        // all optional
type Readonly<T> = { readonly [P in keyof T]: T[P] }; // all readonly
type Mutable<T>  = { -readonly [P in keyof T]: T[P] };// remove readonly
type Complete<T> = { [P in keyof T]-?: T[P] };         // remove optional

// Filter keys with as + never
type StringOnly<T> = { [P in keyof T as T[P] extends string ? P : never]: T[P] };
```

---

## 27. Type Inference (Advanced)

```ts
// as const — lock to literal
const config = { env: "prod", retries: 3 } as const;
// config.env type: "prod" not string

// Contextual typing
["Alice", "Bob"].forEach(name => name.toUpperCase()); // name inferred string

// Control flow narrowing
if (value === null) return 0;
return value.length; // TS knows: not null here
```

---

## 28. Literal Types

```ts
type Dir = "north" | "south" | "east" | "west"; // string literal
type Dice = 1 | 2 | 3 | 4 | 5 | 6;              // numeric literal
type CssVal = `${number}px`;                       // template literal

// Combine literals into new strings
type Routes = `/${string}`;
type Handler = `on${Capitalize<"click" | "blur">}`; // "onClick" | "onBlur"
```

---

## 29. Namespaces

```ts
namespace App {
  export namespace Utils { export function log(msg: string) { ... } }
}
App.Utils.log("hi");

// Most common real use — extend 3rd party types
declare module "express" {
  interface Request { user?: { id: number; role: string }; }
}
```

Modern projects: use ES modules. Namespaces = legacy / `.d.ts` augmentation.

---

## 30. Index Signatures

```ts
const map: { [key: string]: number } = {};  // string keys, number values
// same as: Record<string, number>

interface UserInfo {
  name: string;                        // known prop
  [key: string]: string | number;      // dynamic props — must include name's type
}

// Readonly
interface ROArray { readonly [i: number]: string; }
```

---

## 31. Declaration Merging

```ts
// Interface merging (automatic)
interface A { x: number; }
interface A { y: number; }
// merged: { x: number; y: number }

// Module augmentation (extend 3rd party)
declare module "express" { interface Request { requestId?: string; } }

// Global augmentation
declare global { interface Window { analytics: () => void; } }
```

Only `interface` merges — `type` alias throws error on duplicate.

---

## 32. Async Programming

```ts
async function fetch<T>(url: string): Promise<T> { ... } // always Promise<T>

const [user, posts] = await Promise.all([getUser(), getPosts()]); // parallel

const timeout = (ms: number): Promise<never> =>  // timeout pattern
  new Promise((_, reject) => setTimeout(() => reject(new Error("Timeout")), ms));
await Promise.race([fetch("/api"), timeout(3000)]);

// allSettled — never throws, returns success/failure for each
const results = await Promise.allSettled([...]);
results.forEach(r => r.status === "fulfilled" ? r.value : r.reason);
```

---

## 33. Decorators

Enable: `"experimentalDecorators": true` in tsconfig.

```ts
@logClass             class Service {}                 // class decorator
@measureTime          method() {}                      // method decorator
@logProperty          name: string;                    // property decorator
greet(@validate msg: string) {}                        // parameter decorator
```

Used in: NestJS (`@Controller`, `@Get`), Angular (`@Component`), TypeORM (`@Entity`).
Decorator order: factories top→bottom, execution bottom→top.

---

## 34. Error Handling

```ts
// Custom errors
class AppError extends Error {
  constructor(msg: string, public code: string) {
    super(msg); this.name = this.constructor.name;
    Object.setPrototypeOf(this, AppError.prototype); // fix instanceof
  }
}

// Always narrow caught errors
catch (e: unknown) {
  if (e instanceof AppError) { ... }
  else if (e instanceof Error) { console.error(e.message); }
}

// Never swallow errors
// ❌ catch {} — never empty
// ✅ catch (e) { console.error(e); throw e; }

// Async
fetchData().catch(console.error); // always handle promise rejections
```

---

## 35. Best Practices

```
tsconfig: strict: true — always
any → never. Use unknown + narrow
interface → object shapes
type → unions, tuples, mapped types
Function params → always explicit
Simple vars → let TS infer
Empty array → annotate: string[] = []
as const → config objects, literal types
import type → type-only imports
?. and ?? → null/undefined safety
Dependency injection → testable code
Single responsibility → focused functions
Promise.all → parallel, not sequential
Type guard → narrow before using union
```

---

## File Index

| # | File | Topic |
|---|---|---|
| 01 | [01-intro.md](01-intro.md) | What is TypeScript |
| 02 | [02-simple-types.md](02-simple-types.md) | Primitives |
| 03 | [03-explicit-vs-inference.md](03-explicit-vs-inference.md) | Explicit vs Inference |
| 04 | [04-special-types.md](04-special-types.md) | any, unknown, never, null, undefined |
| 05 | [05-arrays.md](05-arrays.md) | Arrays |
| 06 | [06-tuples.md](06-tuples.md) | Tuples |
| 07 | [07-object-types.md](07-object-types.md) | Object Types |
| 08 | [08-enums.md](08-enums.md) | Enums |
| 09 | [09-type-aliases-and-interfaces.md](09-type-aliases-and-interfaces.md) | Type Aliases & Interfaces |
| 10 | [10-union-types.md](10-union-types.md) | Union Types |
| 11 | [11-functions.md](11-functions.md) | Functions |
| 12 | [12-casting.md](12-casting.md) | Casting |
| 13 | [13-classes.md](13-classes.md) | Classes |
| 14 | [14-generics.md](14-generics.md) | Generics |
| 15 | [15-utility-types.md](15-utility-types.md) | Utility Types |
| 16 | [16-keyof.md](16-keyof.md) | keyof |
| 17 | [17-null-undefined.md](17-null-undefined.md) | Null & Undefined |
| 18 | [18-definitely-typed.md](18-definitely-typed.md) | Definitely Typed |
| 19 | [19-ts5-updates.md](19-ts5-updates.md) | TypeScript 5.x Updates |
| 20 | [20-tsconfig.md](20-tsconfig.md) | tsconfig.json |
| 21 | [21-typescript-nodejs.md](21-typescript-nodejs.md) | TypeScript + Node.js |
| 22 | [22-typescript-react.md](22-typescript-react.md) | TypeScript + React |
| 23 | [23-tooling.md](23-tooling.md) | Tooling (ESLint, Prettier) |
| 24 | [24-type-guards.md](24-type-guards.md) | Type Guards |
| 25 | [25-conditional-types.md](25-conditional-types.md) | Conditional Types |
| 26 | [26-mapped-types.md](26-mapped-types.md) | Mapped Types |
| 27 | [27-type-inference.md](27-type-inference.md) | Type Inference (Advanced) |
| 28 | [28-literal-types.md](28-literal-types.md) | Literal Types |
| 29 | [29-namespaces.md](29-namespaces.md) | Namespaces |
| 30 | [30-index-signatures.md](30-index-signatures.md) | Index Signatures |
| 31 | [31-declaration-merging.md](31-declaration-merging.md) | Declaration Merging |
| 32 | [32-async-programming.md](32-async-programming.md) | Async Programming |
| 33 | [33-decorators.md](33-decorators.md) | Decorators |
| 34 | [34-error-handling.md](34-error-handling.md) | Error Handling |
| 35 | [35-best-practices.md](35-best-practices.md) | Best Practices |
