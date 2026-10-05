# TypeScript Namespaces

> **Note:** Modern projects use ES modules (`import`/`export`). Namespaces are for legacy code, global libraries, or ambient type declarations.

---

## Basic Syntax

```ts
namespace Validation {
  const lettersRegexp = /^[A-Za-z]+$/; // private — not exported

  export interface StringValidator {    // exported — accessible outside
    isValid(s: string): boolean;
  }

  export class LettersValidator implements StringValidator {
    isValid(s: string) { return lettersRegexp.test(s); }
  }

  export class ZipCodeValidator implements StringValidator {
    isValid(s: string) { return /^[0-9]+$/.test(s) && s.length === 5; }
  }
}

new Validation.LettersValidator().isValid("Hello");  // ✅ true
new Validation.LettersValidator().isValid("Hi123");  // ✅ false
```

---

## Nested Namespaces

```ts
namespace App {
  export namespace Utils {
    export function log(msg: string) { console.log(`[LOG] ${msg}`); }
  }

  export namespace Models {
    export interface User { id: number; name: string; }
    export class UserService {
      getUser(id: number): User { return { id, name: "John" }; }
    }
  }
}

App.Utils.log("Starting");
const user = new App.Models.UserService().getUser(1);
```

---

## Namespace Alias

Long paths get verbose — alias them:

```ts
import Components = VeryLongNamespace.DeeplyNested.Components;

const btn = new Components.Button(); // ✅ instead of VeryLong...Button()
```

---

## Multi-file Namespaces

Split across files with `/// <reference />`:

```ts
// validators.ts
namespace Validation {
  export interface StringValidator { isValid(s: string): boolean; }
}

// letters-validator.ts
/// <reference path="validators.ts" />
namespace Validation {
  export class LettersValidator implements StringValidator {
    isValid(s: string) { return /^[A-Za-z]+$/.test(s); }
  }
}

// main.ts
/// <reference path="validators.ts" />
/// <reference path="letters-validator.ts" />
let v = new Validation.LettersValidator();
```

Compile to single file:
```bash
tsc --outFile bundle.js main.ts
```

---

## Declaration Merging (most common real use)

Extending third-party types (e.g. Express):

```ts
declare namespace Express {
  interface Request {
    user?: { id: number; role: string };
  }
}

// Now req.user is typed everywhere in your Express app
```

---

## Namespaces vs ES Modules

| Feature | Namespaces | ES Modules |
|---|---|---|
| Modern projects | ❌ Avoid | ✅ Use this |
| Global scripts / CDN libs | ✅ | ❌ |
| Tree-shaking | ❌ Poor | ✅ Excellent |
| Declaration merging | ✅ Strong | Limited |
| Tooling support | OK | ✅ Best |
| Syntax | `Namespace.Member` | `import { x } from './x'` |

---

## Migration: Namespace → ES Module

```ts
// Before (namespace)
namespace Services {
  export class UserService { getUser(id: number) { ... } }
}
const s = new Services.UserService();

// After (ES module)
// services/UserService.ts
export class UserService { getUser(id: number) { ... } }

// app.ts
import { UserService } from "./services/UserService";
const s = new UserService();
```

---

## When to Use Namespaces

```
Ambient type declarations (.d.ts)   →  ✅ namespaces
Extending library types (Express)   →  ✅ declare namespace
Legacy / no bundler codebase        →  ✅ namespaces
New modern project                  →  ❌ use ES modules
```
