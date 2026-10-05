# TypeScript Tooling

---

## Ecosystem Overview

| Category | Tools |
|---|---|
| Code quality | ESLint + `@typescript-eslint` |
| Editor | VS Code (built-in TS support) |
| Dev server | Vite, ts-node + nodemon |
| Bundlers | Vite, Webpack, Parcel |
| Type check only | `tsc --noEmit` |

---

## ESLint Setup

```bash
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

---

## `.eslintrc.json`

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ],
  "parserOptions": {
    "project": "./tsconfig.json",
    "ecmaVersion": 2020,
    "sourceType": "module"
  },
  "rules": {
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  }
}
```

Key rules:

| Rule | Level | What it catches |
|---|---|---|
| `explicit-function-return-type` | warn | Functions without return type |
| `no-explicit-any` | warn | `any` usage |
| `no-unused-vars` | error | Unused variables (ignores `_` prefix) |

---

## `package.json` Scripts

```json
{
  "scripts": {
    "lint":       "eslint . --ext .ts,.tsx",
    "lint:fix":   "eslint . --ext .ts,.tsx --fix",
    "type-check": "tsc --noEmit"
  }
}
```

| Script | What it does |
|---|---|
| `npm run lint` | Check for lint errors |
| `npm run lint:fix` | Auto-fix fixable issues |
| `npm run type-check` | TS type check only — no output files |

---

## VS Code Tips

VS Code has built-in TypeScript support — no plugin needed.

Useful extensions:
- **ESLint** — shows lint errors inline
- **Prettier** — auto-format on save
- **Error Lens** — shows errors on same line

`settings.json` for auto-fix on save:
```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "editor.formatOnSave": true
}
```

---

## Prettier + ESLint Together

```bash
npm install --save-dev prettier eslint-config-prettier
```

Add `"prettier"` last in `extends` to disable conflicting ESLint format rules:
```json
{
  "extends": [
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ]
}
```

---

## Quick CI Workflow

```bash
npm run type-check   # no type errors
npm run lint         # no lint errors
npm run build        # compiles cleanly
```

Run all three before merge/deploy.
