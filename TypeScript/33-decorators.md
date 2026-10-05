# TypeScript Decorators

Used heavily in **Angular**, **NestJS**, **TypeORM**, **class-validator**.

---

## Enable in `tsconfig.json`

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

---

## 4 Decorator Types

| Type | Applied to | Parameters |
|---|---|---|
| Class | Class declaration | `(constructor: Function)` |
| Method | Class methods | `(target, key, descriptor: PropertyDescriptor)` |
| Property | Class properties | `(target, key)` |
| Parameter | Method params | `(target, key, parameterIndex: number)` |

Syntax: `@decoratorName` placed above what it decorates.

---

## Class Decorator

```ts
function logClass(constructor: Function) {
  console.log(`Class ${constructor.name} defined`);
}

@logClass
class UserService { }
// Logs on file load, not on instantiation
```

Decorator factory (with config):
```ts
function versioned(version: string) {
  return function(constructor: Function) {
    constructor.prototype.version = version;
  };
}

@versioned("1.0.0")
class ApiClient { }

new ApiClient().version; // "1.0.0"
```

---

## Method Decorator

Wraps the original method — useful for logging, auth, timing.

```ts
// Measure execution time
function measureTime(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    const start = performance.now();
    const result = original.apply(this, args);
    console.log(`${key} took ${(performance.now() - start).toFixed(2)}ms`);
    return result;
  };
  return descriptor;
}

class DataProcessor {
  @measureTime
  process(data: number[]) { return data.map(x => x * 2); }
}
```

Role-based access (decorator factory):
```ts
function AllowedRoles(...roles: string[]) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      const userRoles = ["viewer"]; // from auth context
      if (!roles.some(r => userRoles.includes(r))) {
        throw new Error(`Not authorized to call ${key}`);
      }
      return original.apply(this, args);
    };
  };
}

class DocumentService {
  @AllowedRoles("admin", "editor")
  delete(id: string) { console.log(`Deleted ${id}`); }
}
```

Deprecation warning:
```ts
function deprecated(msg: string) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.warn(`${key} is deprecated. ${msg}`);
      return original.apply(this, args);
    };
  };
}
```

---

## Property Decorator

Uses `Object.defineProperty` to intercept get/set:

```ts
function logProperty(target: any, key: string) {
  let value: any;
  Object.defineProperty(target, key, {
    get: () => { console.log(`Getting ${key}`); return value; },
    set: (v) => { console.log(`Setting ${key} = ${v}`); value = v; },
    enumerable: true,
    configurable: true
  });
}

class Product {
  @logProperty name: string;
  @logProperty price: number;
}
```

---

## Parameter Decorator

Records which params to validate (usually paired with method decorator):

```ts
function validateParam(type: string) {
  return (target: any, key: string, index: number) => {
    const meta = Reflect.getOwnMetadata("validations", target, key) || [];
    meta.push({ index, type });
    Reflect.defineMetadata("validations", meta, target, key);
  };
}
```

---

## Decorator Order

Multiple decorators: **factories top→bottom**, **execution bottom→top**:

```ts
@first()   // factory evaluated 1st, executes 2nd
@second()  // factory evaluated 2nd, executes 1st
method() {}
```

---

## Real-World: NestJS-Style Controller

```ts
const ROUTES: any[] = [];

function Controller(prefix = "") {
  return (constructor: Function) => { constructor.prototype.prefix = prefix; };
}

function Get(path = "") {
  return (target: any, key: string, descriptor: PropertyDescriptor) => {
    ROUTES.push({ method: "GET", path, target: target.constructor });
  };
}

@Controller("/users")
class UserController {
  @Get("/")        getAllUsers()   { return []; }
  @Get("/:id")     getUserById()  { return {}; }
}
// ROUTES = [ { GET /users/ }, { GET /users/:id } ]
```

---

## Quick Reference

```ts
// Class
@sealed class MyClass { }

// Method (factory)
@log({ level: "warn" }) method() { }

// Property
@readonly propName: string;

// Parameter
greet(@validate("string") msg: string) { }
```

---

## Where Decorators Are Used

| Framework | Usage |
|---|---|
| NestJS | `@Controller`, `@Get`, `@Injectable`, `@Body` |
| Angular | `@Component`, `@Injectable`, `@Input` |
| TypeORM | `@Entity`, `@Column`, `@PrimaryGeneratedColumn` |
| class-validator | `@IsEmail`, `@IsNotEmpty`, `@MinLength` |
