# 09. Modules & Project Setup

> File 09 — ESM/CJS, path aliases, strict flags, project references intro.

Real TypeScript projects need consistent **module settings**, **path resolution**, and **compiler strictness** across files and packages.

---

## 1. ESM vs CommonJS in TS

JavaScript has two major module systems:

| | ESM | CommonJS |
|---|-----|----------|
| Syntax | `import` / `export` | `require()` / `module.exports` |
| Standard | ECMAScript modules | Node.js legacy |
| Browser | Native | Not native |

### 1.1. TypeScript module settings

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2020"
  }
}
```

Common combinations:

| Project type | Typical `module` | `moduleResolution` |
|--------------|------------------|---------------------|
| Vite / modern bundler | `ESNext` | `bundler` |
| Node ESM (`"type": "module"`) | `NodeNext` | `NodeNext` |
| Node CommonJS | `CommonJS` | `node10` / `node` |

### 1.2. Import / export syntax

```ts
// named exports
export const PI = 3.14;
export function add(a: number, b: number) {
  return a + b;
}

// default export
export default class App {}

// imports
import App from "./App";
import { PI, add } from "./math";
import type { User } from "./types"; // type-only import — erased at compile
```

**Type-only imports** (`import type`) ensure no runtime import is emitted — useful for types and circular dependency breaks.

### 1.3. `package.json` and `"type": "module"`

Node treats `.js` as ESM when:

```json
{
  "type": "module"
}
```

Use `.mts` / `.cts` or explicit `"module"` in tsconfig when mixing systems.

---

## 2. Path aliases

Long relative imports are hard to maintain:

```ts
import { Button } from "../../../components/ui/Button";
```

### 2.1. Configure in `tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```

Usage:

```ts
import { Button } from "@/components/ui/Button";
```

### 2.2. Bundler must understand aliases too

TypeScript paths affect **type-checking only**. Vite, webpack, or Next.js need matching alias config:

```ts
// vite.config.ts
import path from "path";
export default {
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
    },
  },
};
```

---

## 3. Strict mode flags

`"strict": true` enables:

| Flag | Effect |
|------|--------|
| `strictNullChecks` | `null` / `undefined` not assignable to other types without handling |
| `noImplicitAny` | Error on untyped `any` inference |
| `strictFunctionTypes` | Stricter function parameter checking |
| `strictBindCallApply` | Typed `bind`, `call`, `apply` |
| `strictPropertyInitialization` | Class fields must be initialized |
| `noImplicitThis` | `this` must have explicit type |
| `alwaysStrict` | Emit `"use strict"` |

### 3.1. Other recommended flags

```json
{
  "compilerOptions": {
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

`noUncheckedIndexedAccess` makes `arr[i]` return `T | undefined` — safer but noisier.

### 3.2. Incremental strict migration

```json
{
  "compilerOptions": {
    "strict": false,
    "strictNullChecks": true
  }
}
```

Enable flags one at a time on legacy codebases. Use `// @ts-expect-error` sparingly with a ticket to fix.

---

## 4. Monorepo / project references (intro)

Large repos split code into packages with **project references**:

```
packages/
  shared/     → tsconfig.json
  api/        → references shared
  web/        → references shared
```

### 4.1. Root `tsconfig.json`

```json
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/api" },
    { "path": "./packages/web" }
  ]
}
```

### 4.2. Package `tsconfig.json`

```json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "references": [{ "path": "../shared" }]
}
```

Build order:

```bash
tsc -b
```

Benefits: faster incremental builds, clear dependency graph, shared types across packages.

Tools like **pnpm workspaces**, **Turborepo**, and **Nx** layer on top of this pattern.

---

## 5. Common questions

**5.1. What is `moduleResolution: "bundler"`?**  
A: TS 5+ mode for apps using bundlers — allows `import` without file extensions in types while matching Vite/webpack behavior.

**5.2. Do path aliases work at runtime?**  
A: Not automatically — the bundler or runtime (ts-node with config) must resolve them.

**5.3. Should I use default or named exports?**  
A: Named exports scale better (refactor-friendly, tree-shaking clarity). Default exports are fine for single-purpose modules (pages, App root).

**5.4. What is `import type` for?**  
A: Import types only — no runtime import. Avoids circular dependency issues and unnecessary JS output.

**5.5. What does `composite: true` require?**  
A: `declaration` enabled, explicit `rootDir`, no mixed project files — needed for project references.

**5.6. ESM vs CJS in Node today?**  
A: New Node projects prefer ESM (`"type": "module"`) or dual packages. Match `module`/`moduleResolution` to your runtime.

**5.7. Can I disable strict for one file?**  
A: `// @ts-nocheck` at top of file — last resort. Prefer fixing types or scoped `@ts-expect-error`.

**5.8. What is `skipLibCheck`?**  
A: Skips type-checking of `.d.ts` in `node_modules` — faster builds; keep enabled unless debugging lib types.
