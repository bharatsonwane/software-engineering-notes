# 08. Utility & Mapped Types

> File 08 — Partial, Pick, Omit, Record, ReturnType, mapped types, conditional intro.

TypeScript ships **built-in utility types** and lets you build your own with **mapped** and **conditional** types.

---

## 1. `Partial`, `Required`, `Pick`, `Omit`

### 1.1. `Partial<T>`

Makes all properties optional — useful for update payloads:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

function updateUser(id: number, changes: Partial<User>) {
  // apply changes
}

updateUser(1, { name: "Alex" }); // OK — only name required
```

### 1.2. `Required<T>`

Opposite of `Partial` — all properties required:

```ts
type RequiredUser = Required<Partial<User>>; // same as User if all keys were optional first
```

### 1.3. `Pick<T, K>` and `Omit<T, K>`

Select or remove properties:

```ts
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string }
```

Common for DTOs and form state:

```ts
type CreateUserInput = Omit<User, "id">;
type UserListItem = Pick<User, "id" | "name">;
```

---

## 2. `Record` and `ReturnType`

### 2.1. `Record<K, V>`

Object type with keys `K` and values `V`:

```ts
type Role = "admin" | "member" | "guest";
type Permissions = Record<Role, string[]>;

const perms: Permissions = {
  admin: ["read", "write", "delete"],
  member: ["read", "write"],
  guest: ["read"],
};
```

String-keyed dictionaries:

```ts
type StringMap = Record<string, number>;
```

### 2.2. `ReturnType<T>`

Extract function return type:

```ts
function createUser() {
  return { id: 1, name: "Bharat" };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string }
```

Related utilities:

```ts
type Params = Parameters<typeof createUser>; // tuple of arg types
type Constructor = ConstructorParameters<typeof Date>;
```

---

## 3. Mapped types

A **mapped type** transforms each property of an existing type:

```ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type Optional<T> = {
  [K in keyof T]?: T[K];
};
```

### 3.1. Key remapping (`as`)

```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<{ name: string; age: number }>;
// { getName: () => string; getAge: () => number }
```

### 3.2. Filtering keys

```ts
type RemoveId<T> = {
  [K in keyof T as K extends "id" ? never : K]: T[K];
};
```

### 3.3. `as const` and keyof patterns

```ts
const CONFIG = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} as const;

type ConfigKey = keyof typeof CONFIG; // "apiUrl" | "timeout"
type ConfigValues = typeof CONFIG[ConfigKey];
```

---

## 4. Conditional types (intro)

**Conditional types** pick types based on a condition: `T extends U ? X : Y`.

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false
```

### 4.1. Distributive conditionals

When `T` is a union, conditionals distribute:

```ts
type ToArray<T> = T extends unknown ? T[] : never;

type R = ToArray<string | number>;
// string[] | number[]
```

### 4.2. Practical example — nullable unwrap

```ts
type NonNullable<T> = T extends null | undefined ? never : T;

type Safe = NonNullable<string | null>; // string
```

### 4.3. `infer` preview (File 11)

Extract types inside conditionals:

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;

type Item = ElementType<string[]>; // string
```

Conditional + mapped types power most advanced TS libraries (File 11).

---

## 5. Common questions

**5.1. When use `Pick` vs `Omit`?**  
A: `Pick` when you need few fields; `Omit` when you need almost everything except a few.

**5.2. What does `keyof T` return?**  
A: Union of property names of `T` — `"id" | "name" | "email"` for `User`.

**5.3. Are utility types built into the compiler?**  
A: Yes — defined in `lib.es5.d.ts` and friends; no import needed.

**5.4. Can I combine utilities?**  
A: Yes — `Partial<Pick<User, "name" | "email">>` is common.

**5.5. What is the difference between mapped types and interfaces?**  
A: Mapped types **transform** existing types programmatically; interfaces declare shapes directly.

**5.6. What does `readonly [K in keyof T]` do?**  
A: Same keys as `T`, all marked readonly — like the built-in `Readonly<T>`.

**5.7. When do conditional types run?**  
A: At **compile time only** — they emit no JavaScript.

**5.8. How does `ReturnType` help with refactoring?**  
A: Derive types from implementation — change the function, derived types update automatically.
