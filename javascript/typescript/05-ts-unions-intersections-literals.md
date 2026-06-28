# 05. Unions, Intersections & Literals

> File 05 — union, intersection, discriminated unions, exhaustiveness.

TypeScript combines types to express **either/or** (unions), **and** (intersections), and **exact values** (literals).

---

## 1. Union types

A **union** means a value can be one of several types:

```ts
type Id = string | number;

function printId(id: Id) {
  console.log(id);
}

printId(101);
printId("abc-101");
```

Union members share only operations valid for **all** members unless you narrow:

```ts
function padLeft(value: string | number) {
  // value.toUpperCase(); // Error — number has no toUpperCase
  if (typeof value === "string") {
    return value.padStart(5);
  }
  return String(value).padStart(5);
}
```

### 1.1. Common union patterns

```ts
type Status = "pending" | "active" | "done";
type Maybe<T> = T | null | undefined;
type Result<T> = { ok: true; value: T } | { ok: false; error: string };
```

---

## 2. Intersection types

An **intersection** combines multiple types — value must satisfy **all**:

```ts
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

const person: Person = { name: "Bharat", age: 30 };
```

Useful for mixing interfaces:

```ts
type AdminUser = User & { permissions: string[] };
```

Conflicting property types produce `never` on that property:

```ts
type Bad = { x: number } & { x: string }; // x: never
```

---

## 3. Literal and discriminated unions

### 3.1. Literal types

A **literal type** is an exact value:

```ts
type Direction = "north" | "south" | "east" | "west";
type Answer = 42 | true | "yes";
```

```ts
let dir: Direction = "north";
// dir = "up"; // Error
```

### 3.2. Discriminated unions (tagged unions)

Add a common **discriminant** field so TypeScript narrows safely:

```ts
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };
type Shape = Circle | Square;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
  }
}
```

After checking `shape.kind`, TS knows which fields exist.

### 3.3. Real-world example — API states

```ts
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function render<T>(state: RequestState<T>) {
  switch (state.status) {
    case "idle":
      return "Click to load";
    case "loading":
      return "Loading…";
    case "success":
      return state.data; // data exists here
    case "error":
      return state.message;
  }
}
```

---

## 4. Exhaustiveness checking

Ensure every union member is handled — use `never` in the default branch:

```ts
type Shape = Circle | Square | { kind: "triangle"; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default: {
      const _exhaustive: never = shape;
      return _exhaustive;
    }
  }
}
```

If you add a new variant to `Shape` but forget a `case`, assigning to `never` fails at compile time.

### 4.1. Helper: `assertNever`

```ts
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(x)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      return assertNever(shape);
  }
}
```

### 4.2. Union narrowing with `in`

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };
type Pet = Cat | Dog;

function speak(pet: Pet) {
  if ("meow" in pet) {
    pet.meow();
  } else {
    pet.bark();
  }
}
```

More narrowing techniques in File 07.

---

## 5. Common questions

**5.1. Union vs enum for status strings?**  
A: String literal unions are lightweight and idiomatic. Enums add a runtime object (File 02).

**5.2. What is the difference between `|` and `&`?**  
A: `|` is **or** (one of). `&` is **and** (must satisfy all).

**5.3. Why use a discriminant field?**  
A: Gives TypeScript a reliable switch/narrowing key — usually `kind`, `type`, or `status`.

**5.4. Can unions include `null`?**  
A: Yes — `string | null`. With strict null checks, handle both explicitly.

**5.5. What happens if intersection types conflict?**  
A: Overlapping incompatible properties become `never`, often surfacing as errors when you use them.

**5.6. How do I make exhaustiveness work with `if/else`?**  
A: Use `else` with `never` assignment, or a final `assertNever` call after all branches.

**5.7. Are literal types widened?**  
A: Yes on `let` reassignment contexts. Use `as const` to preserve literals (File 08 / 11).

**5.8. Union vs optional property?**  
A: `{ email?: string }` allows missing key. `{ email: string | undefined }` requires the key to exist (value may be undefined).
