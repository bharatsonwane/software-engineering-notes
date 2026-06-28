# 12. Best Practices

> File 12 — avoid any, interface vs type, ESLint, JS migration.

Production TypeScript is less about syntax and more about **team habits** — consistent strictness, lint rules, and a sensible migration path from JavaScript.

---

## 1. Avoiding `any`

`any` disables the type checker for that value and propagates unsafety:

```ts
// Bad — errors hidden until runtime
function parse(data: any) {
  return data.items.map((i: any) => i.name);
}

// Better — validate at boundary
function parse(data: unknown) {
  if (!isValidPayload(data)) {
    throw new Error("Invalid payload");
  }
  return data.items.map((i) => i.name);
}
```

### 1.1. Alternatives to `any`

| Instead of | Use |
|------------|-----|
| Unknown API response | `unknown` + narrowing / Zod |
| Generic key-value bag | `Record<string, unknown>` |
| Mixed array | `(string \| number)[]` or union |
| Third-party untyped lib | `@types/*` or local `.d.ts` |
| Temporary migration | `// @ts-expect-error` with ticket + deadline |

### 1.2. ESLint rules

```json
{
  "rules": {
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/no-unsafe-assignment": "error",
    "@typescript-eslint/no-unsafe-member-access": "error"
  }
}
```

Start with `"warn"` on legacy codebases; tighten to `"error"` over time.

### 1.3. `unknown` at boundaries, concrete types inside

```ts
async function fetchJson(url: string): Promise<unknown> {
  const res = await fetch(url);
  return res.json();
}

const raw = await fetchJson("/api/user");
const user = UserSchema.parse(raw); // concrete User type after validation
```

---

## 2. Prefer `interface` vs `type` in teams

There is no universal winner — **consistency** matters.

### 2.1. Practical convention

```ts
// Object shapes / class contracts
interface User {
  id: string;
  name: string;
}

// Unions, tuples, utilities
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: Error };

type UserId = string;
```

### 2.2. When `interface` wins

- Public API objects you may **extend** later
- **Declaration merging** (augmenting Express, Jest globals)
- Class `implements` contracts

### 2.3. When `type` wins

- **Unions and intersections**
- **Mapped / conditional** types
- Aliasing primitives and computed types

Document the team rule in README or ESLint — avoid bikeshedding per PR.

---

## 3. ESLint and TypeScript

Use **`typescript-eslint`** for type-aware linting:

```bash
npm install -D eslint @eslint/js typescript-eslint
```

```js
// eslint.config.js (flat config)
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.recommendedTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  }
);
```

### 3.1. High-value rules

| Rule | Why |
|------|-----|
| `no-floating-promises` | Catch unhandled async |
| `no-misused-promises` | Don't pass async fn where sync expected |
| `await-thenable` | Don't await non-promises |
| `consistent-type-imports` | Use `import type` for types |
| `no-unused-vars` | Clean dead code |

### 3.2. Formatting

Pair ESLint with **Prettier** — ESLint for logic, Prettier for formatting. Avoid duplicate stylistic rules.

### 3.3. CI pipeline

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "check": "npm run typecheck && npm run lint"
  }
}
```

Run `check` on every PR — types and lint together catch different bug classes.

---

## 4. Migration from JavaScript

### 4.1. Rename strategy

```
app.js  →  app.ts
utils.js → utils.ts (one file at a time)
```

Enable gradual adoption:

```json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": false,
    "strict": true
  }
}
```

Turn on `"checkJs": true` for `.js` files with `// @ts-check` at the top.

### 4.2. Migration order

1. Add `tsconfig.json` + `typescript` dev dependency
2. Rename **leaf** modules first (no dependents)
3. Add types at **boundaries** (API responses, config, shared models)
4. Replace `any` introduced during migration
5. Enable stricter flags incrementally (`strictNullChecks`, `noImplicitAny`)

### 4.3. Typing existing JS libraries

```bash
npm install -D @types/lodash
```

If no types exist:

```ts
// types/shims.d.ts
declare module "legacy-lib";
```

Or write accurate `.d.ts` for the subset you use.

### 4.4. `jsdoc` bridge

Before renaming, add JSDoc to `.js`:

```js
/**
 * @param {string} name
 * @returns {string}
 */
export function greet(name) {
  return `Hello, ${name}`;
}
```

`checkJs` uses JSDoc as types — smooth stepping stone to `.ts`.

### 4.5. Do not rewrite everything at once

Migrate **by feature** or **by folder**. Mixed `.js` and `.ts` is normal for months in large apps.

---

## 5. Common questions

**5.1. Is `any` ever acceptable?**  
A: During migration or with truly dynamic plugins — isolate it, document why, and plan removal.

**5.2. Should strict mode be non-negotiable?**  
A: For new projects, yes. Legacy projects enable flags incrementally.

**5.3. `eslint` vs `tsc` — which catches what?**  
A: `tsc` — type correctness. ESLint — patterns, async misuse, style, some type-aware rules. Use both.

**5.4. How long does JS → TS migration take?**  
A: Depends on size. Leaf-first + boundaries-first often gives value in weeks, full strict coverage in months.

**5.5. `@ts-ignore` vs `@ts-expect-error`?**  
A: Prefer `@ts-expect-error` — fails if the error is fixed, so you remove the comment. `@ts-ignore` silently stays.

**5.6. Should tests be `.ts` too?**  
A: Yes when the app is TS — tests get the same type safety and refactoring support.

**5.7. Monorepo: one tsconfig or many?**  
A: Per package with project references (File 09) — shared strict base config extended per package.

**5.8. What is the single best habit for maintainable TS?**  
A: Treat **external data as `unknown`** until validated — never trust `JSON.parse`, form input, or query params as typed.
