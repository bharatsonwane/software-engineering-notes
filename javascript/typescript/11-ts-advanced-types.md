# 11. Advanced Types

> File 11 — infer, template literals, branded types, type-safe APIs.

Advanced TypeScript features model **string patterns**, **extract types from other types**, and **prevent mixing similar primitives** at compile time.

---

## 1. `infer` keyword

Inside a **conditional type**, `infer` declares a type variable to capture part of a matched type:

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;

type A = ElementType<string[]>;  // string
type B = ElementType<number>;    // number
```

### 1.1. Extract function return type (how `ReturnType` works)

```ts
type MyReturnType<T> = T extends (...args: never[]) => infer R ? R : never;

type R = MyReturnType<() => string>; // string
```

### 1.2. Extract promise inner type

```ts
type Awaited<T> = T extends Promise<infer U> ? U : T;

type Data = Awaited<Promise<{ id: number }>>; // { id: number }
```

Built-in `Awaited<T>` (TS 4.5+) handles nested promises.

### 1.3. Tuple element inference

```ts
type Head<T> = T extends [infer H, ...unknown[]] ? H : never;
type Tail<T> = T extends [unknown, ...infer Rest] ? Rest : never;

type H = Head<[string, number, boolean]>; // string
```

---

## 2. Template literal types

Combine string literal types like template strings:

```ts
type EventName = "click" | "focus" | "blur";
type HandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"
```

### 2.1. Intrinsic string manipulation types

| Utility | Example |
|---------|---------|
| `Uppercase<S>` | `"hello"` → `"HELLO"` |
| `Lowercase<S>` | `"HELLO"` → `"hello"` |
| `Capitalize<S>` | `"hello"` → `"Hello"` |
| `Uncapitalize<S>` | `"Hello"` → `"hello"` |

```ts
type HttpMethod = "get" | "post";
type Route = `/api/${string}`;
type GetRoute = `/api/${Lowercase<HttpMethod>}`;
```

### 2.2. CSS / API route safety

```ts
type Color = "red" | "green" | "blue";
type BgClass = `bg-${Color}`;
// "bg-red" | "bg-green" | "bg-blue"

function setBg(className: BgClass) {
  document.body.className = className;
}
```

### 2.3. Mapped + template combo

```ts
type PropEvents<T> = {
  [K in keyof T as `on${Capitalize<string & K>}Change`]: (value: T[K]) => void;
};

type UserEvents = PropEvents<{ name: string; age: number }>;
// { onNameChange: (value: string) => void; onAgeChange: (value: number) => void }
```

---

## 3. Branded types

**Branded** (nominal) types prevent accidentally mixing structurally identical types:

```ts
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId) {
  return { id, name: "Bharat" };
}

const userId = createUserId("u-123");
const orderId = "o-456" as OrderId;

getUser(userId);   // OK
// getUser(orderId); // Error — OrderId not assignable to UserId
```

### 3.1. Opaque aliases with helper

```ts
declare const __brand: unique symbol;

type Brand<T, B extends string> = T & { readonly [__brand]: B };

type Email = Brand<string, "Email">;
type Password = Brand<string, "Password">;

function register(email: Email, password: Password) {}
```

Forces explicit creation functions — raw strings cannot slip through.

### 3.2. Units of measure (pattern)

```ts
type Meters = number & { readonly unit: "m" };
type Feet = number & { readonly unit: "ft" };

function toMeters(feet: Feet): Meters {
  return (feet * 0.3048) as Meters;
}
```

---

## 4. Type-safe API patterns

### 4.1. End-to-end typed fetch

```ts
type ApiRoutes = {
  "/users": { GET: { response: User[] } };
  "/users/:id": { GET: { response: User } };
};

async function apiGet<Path extends keyof ApiRoutes>(
  path: Path
): Promise<ApiRoutes[Path]["GET"]["response"]> {
  const res = await fetch(path as string);
  return res.json();
}

const users = await apiGet("/users"); // User[]
```

Libraries like **tRPC**, **Hono RPC**, and **OpenAPI generators** automate this pattern.

### 4.2. Builder pattern with chained generics

```ts
class QueryBuilder<T extends Record<string, unknown>> {
  where<K extends keyof T>(key: K, value: T[K]): this {
    return this;
  }
}

const q = new QueryBuilder<{ name: string; age: number }>();
q.where("name", "Bharat"); // OK
// q.where("age", "thirty"); // Error
```

### 4.3. Zod / schema validation at boundaries

Types are compile-time only — validate external data at runtime:

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
});

type User = z.infer<typeof UserSchema>;

function parseUser(raw: unknown): User {
  return UserSchema.parse(raw);
}
```

Single source of truth: schema → runtime check + inferred TS type.

### 4.4. Const assertions for API contracts

```ts
const ROUTES = {
  home: "/",
  profile: "/profile",
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES];
// "/" | "/profile"
```

### 4.5. Checklist for type-safe boundaries

Advanced types shine at **system boundaries** — HTTP, env vars, config keys — where mistakes are costly:

1. **Parse** external input (`unknown` → validated type)
2. **Brand** IDs and domain primitives
3. **Discriminate** responses with tagged unions (File 05)
4. **Derive** types from schemas or const objects — avoid duplicating shapes

---

## 5. Common questions

**5.1. What does `infer` do?**  
A: Captures a type inside a conditional match — "if T looks like X, name this part U".

**5.2. Are template literal types only for strings?**  
A: Yes — they operate on string literal types and string manipulation utilities.

**5.3. Do branded types exist at runtime?**  
A: No — brands are compile-time only. Runtime validation still needed for untrusted input.

**5.4. When are advanced types worth it?**  
A: Libraries, design systems, API clients, large teams. Overkill for small scripts.

**5.5. What is the difference between `infer` and generics?**  
A: Generics declare parameters on types/functions. `infer` extracts types **inside** conditional type logic.

**5.6. Can template literals be too slow to compile?**  
A: Very large cartesian products of unions can slow `tsc`. Split types or simplify unions if builds lag.

**5.7. How does Zod relate to TypeScript?**  
A: Zod validates at runtime; `z.infer` gives static types — bridges the compile/runtime gap.

**5.8. What is `satisfies` (TS 4.9+)?**  
A: Checks value matches type while preserving narrow literal inference — great for config objects:

```ts
const palette = {
  red: "#f00",
  green: "#0f0",
} as const satisfies Record<string, string>;
```
