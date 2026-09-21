# TypeScript Classes

## Basic Class with Types

```ts
class Person {
  name: string;
}

const person = new Person();
person.name = "Jane"; // ✅
```

---

## Visibility Modifiers

| Modifier    | Accessible from                          | Default? |
|-------------|------------------------------------------|----------|
| `public`    | Anywhere                                 | ✅ Yes   |
| `private`   | Inside the class only                    | No       |
| `protected` | Inside the class + subclasses that extend it | No  |

```ts
class Person {
  private name: string;

  public constructor(name: string) {
    this.name = name;
  }

  public getName(): string {
    return this.name;
  }
}

const person = new Person("Jane");
console.log(person.getName()); // ✅ "Jane"
console.log(person.name);      // ❌ Error: 'name' is private
```

---

## Parameter Properties (Shorthand)

Instead of declaring + assigning manually, put the modifier in the constructor param:

```ts
// Long way
class Person {
  private name: string;
  constructor(name: string) { this.name = name; }
}

// Short way — same result
class Person {
  constructor(private name: string) {}

  public getName(): string { return this.name; }
}
```

---

## `readonly`

Can only be set at declaration or in the constructor — never changed after:

```ts
class Person {
  private readonly name: string;

  constructor(name: string) {
    this.name = name; // ✅ only here
  }
}

// person.name = "Bob"; ❌ Error: cannot assign to readonly property
```

---

## `implements` — Class Must Follow Interface

```ts
interface Shape {
  getArea: () => number;
}

class Rectangle implements Shape {
  constructor(protected readonly width: number, protected readonly height: number) {}

  public getArea(): number {
    return this.width * this.height;
  }
}
```

Class can implement multiple interfaces:
```ts
class Rectangle implements Shape, Printable { ... }
```

---

## `extends` — Inherit from Another Class

```ts
class Square extends Rectangle {
  constructor(width: number) {
    super(width, width); // call parent constructor
  }

  // getArea() inherited from Rectangle — no need to rewrite it
}

const sq = new Square(5);
console.log(sq.getArea()); // ✅ 25
```

One class can only extend **one** other class.

---

## `override` — Replace Parent Method

```ts
class Rectangle {
  constructor(protected readonly width: number, protected readonly height: number) {}

  public toString(): string {
    return `Rectangle[width=${this.width}, height=${this.height}]`;
  }
}

class Square extends Rectangle {
  constructor(width: number) { super(width, width); }

  public override toString(): string {    // explicitly replaces parent method
    return `Square[width=${this.width}]`;
  }
}
```

`override` keyword is optional by default. Enable `noImplicitOverride` in tsconfig to require it.

---

## `abstract` — Base Class Template

Abstract class can't be instantiated directly — only subclasses can.
Abstract methods have no body — subclasses must implement them.

```ts
abstract class Polygon {
  public abstract getArea(): number; // no body — subclass must implement

  public toString(): string {
    return `Polygon[area=${this.getArea()}]`; // can use abstract method here
  }
}

class Rectangle extends Polygon {
  constructor(protected readonly width: number, protected readonly height: number) {
    super();
  }

  public getArea(): number {
    return this.width * this.height; // ✅ required implementation
  }
}

// new Polygon(); ❌ Error: cannot create instance of abstract class
const r = new Rectangle(4, 5);
console.log(r.toString()); // "Polygon[area=20]"
```

---

## Full Hierarchy Example

```
interface Shape      — contract: must have getArea()
     ↑
abstract Polygon     — base: has toString(), forces getArea()
     ↑
class Rectangle      — implements getArea(), has width/height
     ↑
class Square         — calls super(width, width), overrides toString()
```

---

## Quick Reference

```ts
class Animal {
  constructor(public name: string, private age: number) {}
  // public name  — accessible everywhere
  // private age  — class-internal only
}

class Dog extends Animal {
  constructor(name: number, protected breed: string) {
    super(name, 0);
    // protected breed — Dog + subclasses of Dog
  }

  public override toString() { return `${this.name} (${this.breed})`; }
}
```
