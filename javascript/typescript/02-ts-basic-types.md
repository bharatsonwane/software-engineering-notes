# 02. Basic Types

> File 02 — primitives, arrays, tuples, enum, any vs unknown.

TypeScript models JavaScript values with types. Most map directly to JS runtime types, plus a few TS-only constructs (`enum`, `tuple`, `void`, `never`).

---

## 1. Primitives and `any`

### 1.1. JavaScript primitives in TypeScript

| TS type | JS equivalent | Example |
|---------|---------------|---------|
| `string` | string | `"`hello`` |
| `number` | number | `42`, `3.14`, `NaN` |
| `boolean` | boolean | `true`, `false` |
| `bigint` | bigint | `100n` |
| `symbol` | symbol | `Symbol("id")` |
| `null` | null | `null` |
| `undefined` | undefined | `undefined` |

```ts
let name: string = "Bharat";
let count: number = 0;
let ok: boolean = false;
```

### 1.2. `void` and `never`

**`void`** — function returns nothing meaningful:

```ts
function log(msg: string): void {
  console.log(msg);
}
```

**`never`** — function never returns (throws or infinite loop):

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

### 1.3. `any` — escape hatch (avoid when possible)

`any` disables type checking for that value:

```ts
let value: any = 10;
value = "hello";
value.foo.bar; // No compile error — dangerous
```

Use `any` only for gradual migration or truly dynamic data. Prefer `unknown` (below).

### 1.4. Type widening and literals

```ts
let x = "hello";  // string
const y = "hello"; // type "hello" (literal) — const cannot be reassigned
```

Literal types are covered in depth in File 05.

---

## 2. Arrays and tuples

### 2.1. Array types

Two equivalent syntaxes:

```ts
const nums: number[] = [1, 2, 3];
const tags: Array<string> = ["ts", "js"];
```

Read-only arrays:

```ts
const frozen: readonly number[] = [1, 2, 3];
// frozen.push(4); // Error
```

### 2.2. Tuples — fixed-length, typed arrays

```ts
type Point = [number, number];
const origin: Point = [0, 0];

type UserEntry = [string, number]; // [name, age]
const row: UserEntry = ["Bharat", 30];
```

Optional and rest elements:

```ts
type StringsAndOptionalNumber = [string, string, number?];
type StringNumberBooleans = [string, number, ...boolean[]];
```

### 2.3. Tuple vs array

| | Array | Tuple |
|---|-------|-------|
| Length | Variable | Fixed (by type) |
| Use case | Homogeneous lists | CSV row, coordinates, `[key, value]` pairs |

```ts
// React useState return type is a tuple
const [count, setCount]: [number, (n: number) => void] = useState(0);
```

---

## 3. `enum`

TypeScript `enum` creates a set of named constants. Prefer **`const enum`** or plain unions in modern code; enums remain common in legacy codebases.

### 3.1. Numeric enum (default)

```ts
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}

const move = Direction.Up;
```

### 3.2. String enum

```ts
enum Status {
  Pending = "PENDING",
  Active = "ACTIVE",
  Done = "DONE",
}
```

### 3.3. Alternatives (often preferred)

```ts
// Union of string literals — no runtime object emitted
type Status = "PENDING" | "ACTIVE" | "DONE";

const STATUS = {
  Pending: "PENDING",
  Active: "ACTIVE",
} as const;
type StatusFromConst = typeof STATUS[keyof typeof STATUS];
```

**Rule of thumb:** use string unions or `as const` objects unless you need enum-specific features or existing API compatibility.

---

## 4. `unknown` vs `any`

Both accept any value. The difference is **what you can do with them**:

```ts
let a: any = getValue();
a.toFixed(); // OK — no check

let u: unknown = getValue();
// u.toFixed(); // Error — must narrow first

if (typeof u === "number") {
  u.toFixed(2); // OK after narrowing
}
```

| | `any` | `unknown` |
|---|-------|-----------|
| Assign anything to it | Yes | Yes |
| Assign to stricter types | Yes (unsafe) | No — must narrow |
| Call methods / access props | Yes | No until narrowed |
| Recommended | Last resort | Default for "don't know yet" |

### 4.1. Narrowing `unknown`

```ts
function parseJson(raw: string): unknown {
  return JSON.parse(raw);
}

const data = parseJson('{"name":"Bharat"}');

if (
  typeof data === "object" &&
  data !== null &&
  "name" in data &&
  typeof (data as { name: unknown }).name === "string"
) {
  console.log(data.name);
}
```

Type guards and narrowing patterns are covered in File 07.

### 4.2. `never` in exhaustiveness

When narrowing unions, `never` proves you handled every case:

```ts
type Shape = { kind: "circle"; r: number } | { kind: "square"; s: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.r ** 2;
    case "square":
      return shape.s ** 2;
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

---

## 5. Common questions

**5.1. What is the difference between `null` and `undefined` in TypeScript?**  
A: Both exist in JS. With `strictNullChecks`, you must handle them explicitly. Optional properties use `?:` which allows `undefined`.

**5.2. When should I use `any`?**  
A: Rarely — migration, third-party JS without types, or prototyping. Replace with proper types or `unknown` as soon as possible.

**5.3. Tuple vs array — when to use tuples?**  
A: When position and type at each index matter — `[string, number]`, hook return pairs, fixed CSV columns.

**5.4. Are TypeScript enums compiled to JavaScript?**  
A: Yes — they emit a runtime object (unless `const enum`, which inlines). String literal unions emit nothing extra.

**5.5. What is `readonly` for arrays?**  
A: Prevents mutating methods (`push`, `pop`) and reassignment of elements at the type level.

**5.6. Can I mix types in an array?**  
A: Yes — `(string | number)[]` or use a union. For fixed positions, prefer a tuple.

**5.7. Why prefer `unknown` over `any`?**  
A: `unknown` forces you to validate before use, keeping type safety at boundaries (API responses, `JSON.parse`).

**5.8. What is `never` used for besides throwing functions?**  
A: Exhaustiveness checks in `switch`, impossible branches after narrowing, and advanced conditional types (File 11).
