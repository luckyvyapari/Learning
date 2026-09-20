# What is TypeScript?

## Core Idea

| Concept         | What it means                                                     |
|-----------------|-------------------------------------------------------------------|
| Superset of JS  | All valid JavaScript is valid TypeScript                          |
| Static typing   | Type errors caught **before** running code (at compile time)      |
| Transpiled      | TypeScript → compiled → plain JavaScript → runs everywhere JS runs |

---

## Install & Setup

```bash
# Install compiler
npm install typescript --save-dev

# Generate tsconfig.json
npx tsc --init
```

---

## Your First TypeScript File

**`hello.ts`**
```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

const message: string = greet("World");
console.log(message);
```

**Compile it:**
```bash
npx tsc hello.ts
```

**Output (`hello.js`):**
```js
function greet(name) {
  return "Hello, ".concat(name, "!");
}
const message = greet("World");
console.log(message);
```

**Run it:**
```bash
node hello.js
# Hello, World!
```

---

## Key Points

- TypeScript **only checks types at compile time** — types are stripped at runtime
- The compiled `.js` file is what actually executes
- With `tsconfig.json`: put `.ts` files in `src/`, compiled output goes to `build/`
