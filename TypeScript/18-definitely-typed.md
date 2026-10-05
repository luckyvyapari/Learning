# TypeScript Definitely Typed

## Problem

Many npm packages are pure JavaScript — no TypeScript types included.
Using them in TS = no type safety, no autocomplete.

---

## Solution: `@types/*`

[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped) = community repo of type definitions for untyped npm packages.

```bash
npm install --save-dev @types/jquery
npm install --save-dev @types/lodash
npm install --save-dev @types/node
```

Install as `devDependency` — types only needed at compile time, not runtime.

---

## How it Works

1. Install `@types/package-name`
2. TypeScript picks up types automatically — no config needed
3. VS Code / editors suggest `@types` install when types are missing

---

## Check if Types Exist

```bash
npm info @types/package-name
```

If `404` / not found — no community types exist. Options:
- Write your own `.d.ts` declaration file
- Use `any` as a workaround (type unsafe)

---

## Quick Pattern

| Situation | What to do |
|---|---|
| Package has built-in types | Just `npm install` — done |
| Package has `@types/` available | `npm install --save-dev @types/package` |
| No types exist anywhere | Create `custom.d.ts` with `declare module 'pkg'` |

---

## Minimal Custom Declaration (last resort)

```ts
// custom.d.ts
declare module "untyped-package" {
  const value: any;
  export default value;
}
```

Silences TS errors. Not type-safe, but unblocks you.
