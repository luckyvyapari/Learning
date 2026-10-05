# TypeScript Configuration (`tsconfig.json`)

```bash
npx tsc --init   # generate tsconfig.json
```

---

## Top-Level Fields

| Field | What it does |
|---|---|
| `compilerOptions` | How TS compiles code |
| `include` | Files/folders to compile |
| `exclude` | Files/folders to skip |
| `files` | Explicit file list (rare) |
| `extends` | Inherit from another config |
| `references` | Multi-package / monorepo setup |

---

## Minimal Config

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs"
  },
  "include": ["src/**/*"]
}
```

---

## Advanced Config

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@app/*": ["src/app/*"]
    },
    "outDir": "dist",
    "esModuleInterop": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## Key `compilerOptions` Explained

| Option | What it does |
|---|---|
| `target` | JS version to compile to (`es6`, `es2020`, `esnext`) |
| `module` | Module system (`commonjs` for Node, `esnext` for bundlers) |
| `strict` | Enables all strict type checks — always turn on |
| `outDir` | Where compiled JS goes (usually `dist/`) |
| `baseUrl` | Root for path aliases |
| `paths` | Alias shortcuts (`@app/*` → `src/app/*`) |
| `esModuleInterop` | Fixes default imports from CommonJS packages |
| `declaration` | Generate `.d.ts` type files (for libraries) |
| `strictNullChecks` | `null`/`undefined` must be handled explicitly |

---

## `extends` — Share Config

```json
// tsconfig.base.json
{
  "compilerOptions": { "strict": true, "target": "es2020" }
}

// tsconfig.json (inherits base, adds more)
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": { "outDir": "dist" },
  "include": ["src"]
}
```

Useful in monorepos — one shared base, each package extends it.

---

## By Project Type

| Project type | Key options |
|---|---|
| Node.js app | `"target": "es2020"`, `"module": "commonjs"`, `"strict": true` |
| React / bundler | `"module": "esnext"`, `"jsx": "react-jsx"`, `"esModuleInterop": true` |
| Library | `"declaration": true`, `"outDir": "dist"` |
| Monorepo | `"references": [...]`, `"composite": true` |

---

## Best Practices

```
Always enable "strict": true
Exclude "node_modules" and "dist"
Never commit dist/ to git — add to .gitignore
Use "extends" in monorepos instead of duplicating config
Paths not resolving? Check "baseUrl" + "paths" match
```
