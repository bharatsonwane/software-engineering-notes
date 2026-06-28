# 01. Setup & First Types

> File 01 — tsc, tsconfig, annotations.

**TypeScript** is JavaScript with an optional **static type system**. Your `.ts` files compile to plain `.js` that runs anywhere JavaScript runs.

```ts
// app.ts
const name: string = "Bharat";
const age: number = 30;

function greet(person: string): string {
  return `Hello, ${person}!`;
}
```

TypeScript catches many bugs **before** runtime — wrong property names, missing arguments, incorrect return types.

---

## 1. What TypeScript adds to JavaScript

JavaScript is **dynamically typed** — types are checked at runtime (or not at all):

```js
function add(a, b) {
  return a + b;
}
add("1", 2); // "12" — probably not what you wanted
```

TypeScript is **statically typed** — types are checked at **compile time**:

```ts
function add(a: number, b: number): number {
  return a + b;
}
add("1", 2); // Error at compile time: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 1.1. Key benefits

| Benefit | What it means |
|---------|---------------|
| Early error detection | Typos and wrong types fail in the editor / `tsc`, not in production |
| Better tooling | Autocomplete, go-to-definition, safe refactors |
| Self-documenting code | Types describe the shape of data and APIs |
| Gradual adoption | Add types file-by-file; `.js` can coexist with `.ts` |

### 1.2. What TypeScript is *not*

- **Not a new runtime** — browsers and Node still execute JavaScript.
- **Not a guarantee of zero bugs** — logic errors still slip through.
- **Not required everywhere** — you can use `any` or plain `.js` when migrating.

### 1.3. TypeScript vs JavaScript relationship

```
TypeScript (.ts / .tsx)
        │
        ▼  tsc (or bundler: Vite, esbuild, SWC)
JavaScript (.js / .jsx)
        │
        ▼
Browser / Node.js
```

Every valid JavaScript program is valid TypeScript (with `allowJs` or after renaming). TypeScript **adds** syntax; it does not replace JS fundamentals.

---

## 2. Installing and compiling (`tsc`)

### 2.1. Install locally in a project

```bash
npm init -y
npm install typescript --save-dev
npx tsc --version
```

Global install is optional; **project-local** TypeScript keeps teams on the same version:

```bash
npm install -g typescript   # optional
tsc --version
```

### 2.2. First compile

Create `hello.ts`:

```ts
const message: string = "Hello, TypeScript!";
console.log(message);
```

Compile:

```bash
npx tsc hello.ts          # produces hello.js
node hello.js
```

### 2.3. Watch mode

Recompile on every save:

```bash
npx tsc --watch
# or
npx tsc -w
```

### 2.4. Where output goes

| Flag / config | Effect |
|---------------|--------|
| `tsc file.ts` | Emits `file.js` next to source |
| `"outDir": "./dist"` | All compiled files go to `dist/` |
| `"noEmit": true` | Type-check only (common with Vite/Next — bundler emits JS) |

Modern apps often use **Vite**, **Next.js**, or **esbuild** to transpile TS; `tsc` is still used for **type-checking** (`tsc --noEmit`).

---

## 3. `tsconfig.json` basics

Initialize a config file:

```bash
npx tsc --init
```

Minimal example:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 3.1. Important compiler options

| Option | Purpose |
|--------|---------|
| `target` | JS version emitted (`ES2020`, `ESNext`, …) |
| `module` | Module system for output (`CommonJS`, `ESNext`, …) |
| `strict` | Enables strict type-checking (recommended) |
| `noEmit` | Check types without writing `.js` files |
| `jsx` | How to handle `.tsx` (`react-jsx`, `preserve`, …) |
| `moduleResolution` | How TS resolves imports (`node`, `bundler`) |
| `baseUrl` / `paths` | Path aliases (File 09) |

### 3.2. `include` and `exclude`

```json
{
  "include": ["src"],
  "exclude": ["**/*.test.ts", "node_modules"]
}
```

Only files matched by `include` (and not `exclude`) are part of the project.

### 3.3. Strict mode

`"strict": true` turns on a bundle of checks. The most important:

- `strictNullChecks` — `null` / `undefined` must be handled explicitly
- `noImplicitAny` — variables need types or inferrable values
- `strictFunctionTypes` — safer function parameter checking

Start with **strict enabled** on new projects.

---

## 4. Type annotations on variables

A **type annotation** tells TypeScript what shape a value must have.

### 4.1. Basic annotations

```ts
let count: number = 0;
let title: string = "Notes";
let active: boolean = true;
```

### 4.2. Type inference

When you initialize a variable, TypeScript often **infers** the type — no annotation needed:

```ts
let count = 0;        // inferred as number
const title = "Notes"; // inferred as string (literal type "Notes" — widened to string on `let` reassignment context)
```

**Prefer inference** when the type is obvious; add annotations when it helps readability or when inference is too wide.

```ts
// Explicit when empty — TS cannot infer usefully
let items: string[] = [];

// Explicit for function boundaries (File 04)
function parseId(raw: string): number {
  return Number(raw);
}
```

### 4.3. `const` vs `let`

```ts
const PI = 3.14;           // number
let status = "pending";    // string

const user = { name: "Bharat" }; // { name: string }
user.name = "Alex";        // OK — object is mutable
// user = {}              // Error — cannot reassign const
```

### 4.4. Annotating arrays and objects (preview)

```ts
const tags: string[] = ["ts", "js"];
const scores: number[] = [90, 85];

const point: { x: number; y: number } = { x: 10, y: 20 };
```

Interfaces and type aliases (File 03) replace inline object shapes in larger codebases.

### 4.5. Comments that strip types (`@ts-check` in JS)

In plain `.js` files you can opt into checking:

```js
// @ts-check
/** @type {string} */
let name = "Bharat";
```

For new code, prefer `.ts` files.

---

## 5. Common questions

**5.1. Do I need to annotate every variable?**  
A: No. Use inference for locals; annotate function parameters, return types, and public APIs when it clarifies intent.

**5.2. Does TypeScript slow down my app?**  
A: No. Types are erased at compile time. Zero runtime cost from typing.

**5.3. What is the difference between `tsc` and Babel/esbuild?**  
A: `tsc` type-checks and can emit JS. Babel/esbuild/Vite transpile TS **fast** but often **ignore** type errors unless you run `tsc --noEmit` separately.

**5.4. Can I use TypeScript without `tsconfig.json`?**  
A: Yes for single files (`tsc file.ts`). Real projects should use `tsconfig.json` for consistent options.

**5.5. What file extensions does TypeScript use?**  
A: `.ts` for TypeScript; `.tsx` when JSX is included (React).

**5.6. What does `strict: true` do?**  
A: Enables the strictest family of checks. Recommended for new projects; migrating old code may require fixing many errors first.

**5.7. Is TypeScript a superset of JavaScript?**  
A: Practically yes — valid JS (with minor config caveats) is valid TS. TS adds types and a few extra syntax features (`enum`, `interface`, etc.).

**5.8. How do I run TypeScript directly without compiling manually?**  
A: Use `tsx` or `ts-node` for scripts; use a bundler/dev server (Vite, Next) for apps. Production still runs compiled JS.
