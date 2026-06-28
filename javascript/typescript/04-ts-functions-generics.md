# 04. Functions & Generics

> File 04 — parameters, return types, overloads, generics.

Functions are first-class in TypeScript. Annotate **inputs and outputs** so callers and implementers stay in sync.

---

## 1. Typed parameters and return types

```ts
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Function type alias
type BinaryOp = (a: number, b: number) => number;
const divide: BinaryOp = (a, b) => a / b;
```

Return type inference usually works; explicit return types help on **public APIs** and catch accidental changes:

```ts
function getUser(id: number) {
  // inferred return type from return statements
  return { id, name: "Bharat" };
}
```

---

## 2. Optional and default parameters

```ts
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}!`;
}

function createUser(name: string, role: string = "member"): User {
  return { id: 1, name, role };
}
```

Optional parameters must come **after** required ones. Default parameters make the argument optional at the call site.

```ts
greet("Bharat");           // "Hello, Bharat!"
greet("Bharat", "Hi");     // "Hi, Bharat!"
createUser("Alex");        // role = "member"
```

### 2.1. Rest parameters

```ts
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6
```

---

## 3. Function overloads

When one function behaves differently based on argument types, **overloads** document the contract:

```ts
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  if (typeof value === "number") {
    return value.toFixed(2);
  }
  return value.trim();
}

format("  hi  "); // string
format(3.14159);  // "3.14"
```

Rules:

- Overload **signatures** have no body — they declare allowed call shapes.
- One **implementation** signature must accept all overload cases (often wider types).

Use sparingly — unions and generics often replace overloads:

```ts
function identity<T>(value: T): T {
  return value;
}
```

---

## 4. Generic functions and constraints

**Generics** let you write reusable code while preserving type relationships.

### 4.1. Basic generic function

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}

first([1, 2, 3]);     // number | undefined
first(["a", "b"]);    // string | undefined
```

### 4.2. Multiple type parameters

```ts
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}

pair("id", 42); // [string, number]
```

### 4.3. Constraints with `extends`

Limit what `T` can be:

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Bharat", age: 30 };
getProperty(user, "name"); // string
// getProperty(user, "email"); // Error — "email" not in user
```

### 4.4. Generic interfaces and classes

```ts
interface ApiResult<T> {
  data: T;
  status: number;
}

class Stack<T> {
  private items: T[] = [];
  push(item: T) {
    this.items.push(item);
  }
  pop(): T | undefined {
    return this.items.pop();
  }
}
```

### 4.5. `const` type parameters (TS 5.0+)

Preserve literal types through generic calls:

```ts
function asConst<T extends readonly unknown[]>(items: T): T {
  return items;
}

const colors = asConst(["red", "green"] as const);
// type readonly ["red", "green"]
```

### 4.6. Defaults on generics

```ts
function createState<T = string>(initial: T): { value: T } {
  return { value: initial };
}
```

---

## 5. Common questions

**5.1. Should I always annotate return types?**  
A: Not for simple internal functions — inference is fine. Annotate exported functions and when you want to lock the contract.

**5.2. What is `void` vs omitting return type?**  
A: `void` means the caller should ignore the return value. Functions with no return implicitly return `undefined`.

**5.3. When are overloads better than unions?**  
A: When return type depends on input type in a way callers should see clearly (`document.getElementById` style). Prefer generics when one pattern fits all.

**5.4. What does `extends keyof T` mean?**  
A: `K` must be one of the keys of `T` — safe property access.

**5.5. Can arrow functions be generic?**  
A: Yes: `const fn = <T>(x: T): T => x;` (in `.tsx`, use `<T,>(x: T) => x` to avoid JSX ambiguity).

**5.6. What is the difference between `(x: number) => void` and `(x: number): void =>`?**  
A: First is a **type**; second is invalid syntax. Arrow function: `(x: number): void => { ... }`.

**5.7. Are generics erased at runtime?**  
A: Yes — like all types, generics do not exist in emitted JavaScript.

**5.8. How do generics relate to `any`?**  
A: Generics preserve type information through the call. `any` throws it away.
