# 03. Interfaces & Type Aliases

> File 03 — interface, type, extending, when to use which.

Objects dominate application data. TypeScript describes object shapes with **`interface`** and **`type`** aliases.

---

## 1. `interface` syntax

An **interface** names the shape of an object:

```ts
interface User {
  id: number;
  name: string;
  email?: string; // optional
}

const user: User = {
  id: 1,
  name: "Bharat",
};
```

### 1.1. Readonly and index signatures

```ts
interface Config {
  readonly apiUrl: string;
  timeout: number;
}

interface StringDictionary {
  [key: string]: string;
}

interface NumberDictionary {
  [index: number]: string; // array-like
}
```

### 1.2. Call and construct signatures

```ts
interface SearchFn {
  (query: string, limit?: number): string[];
}

interface Timestamped {
  createdAt: Date;
}

interface UserConstructor {
  new (name: string): User;
}
```

---

## 2. `type` aliases

**`type`** creates an alias for any type — objects, unions, primitives, tuples:

```ts
type UserId = number;
type Point = { x: number; y: number };

type Result =
  | { success: true; data: string }
  | { success: false; error: string };
```

Object shapes with `type`:

```ts
type User = {
  id: number;
  name: string;
  email?: string;
};
```

For a plain object shape, `interface User` and `type User = { ... }` are often interchangeable.

---

## 3. Extending interfaces

Interfaces support **declaration merging** and **`extends`**:

```ts
interface Person {
  name: string;
}

interface Employee extends Person {
  employeeId: number;
  department: string;
}

const emp: Employee = {
  name: "Bharat",
  employeeId: 101,
  department: "Engineering",
};
```

Multiple inheritance:

```ts
interface Named {
  name: string;
}

interface Aged {
  age: number;
}

interface Person extends Named, Aged {}
```

### 3.1. Extending with `type` (intersection)

Types use **`&`** (intersection), not `extends`:

```ts
type Person = Named & Aged;

type Employee = Person & {
  employeeId: number;
};
```

---

## 4. Interface vs type — when to use which

| Prefer `interface` | Prefer `type` |
|------------------|---------------|
| Object / class contracts | Unions, intersections, tuples |
| Public API shapes you may extend | Mapped / conditional types |
| Declaration merging (e.g. augmenting modules) | Aliasing primitives (`type ID = string`) |
| React component props (team convention) | Utility type compositions |

### 4.1. Declaration merging (interfaces only)

```ts
interface Window {
  myAppVersion: string;
}

// Later in another file — merges into Window
interface Window {
  theme: "light" | "dark";
}
```

Types cannot be merged — duplicate `type Window` is an error.

### 4.2. Practical team guidelines

```ts
// Object shape for a domain model
interface Product {
  id: string;
  title: string;
  price: number;
}

// Discriminated union of API outcomes
type ApiResponse<T> =
  | { status: "ok"; data: T }
  | { status: "error"; message: string };

// Reusable primitive alias
type ProductId = string;
```

Many codebases use **`interface` for objects** and **`type` for everything else**. Consistency within a project matters more than the rule itself.

### 4.3. Implementing in classes

Both work with `implements`:

```ts
interface Serializable {
  serialize(): string;
}

class User implements Serializable {
  constructor(public name: string) {}
  serialize() {
    return JSON.stringify({ name: this.name });
  }
}
```

---

## 5. Common questions

**5.1. Can I use `interface` and `type` together?**  
A: Yes. An interface can extend a type alias, and a type can use an interface in an intersection.

**5.2. Which is better for React props?**  
A: Both work. `interface Props` is common; `type Props` is equally valid, especially with unions.

**5.3. What does `?` mean on a property?**  
A: Optional — the property may be missing or explicitly `undefined` (with `strictNullChecks`).

**5.4. Can interfaces describe functions?**  
A: Yes — via call signatures (`interface Fn { (x: number): number }`) or as a property type.

**5.5. What is an index signature?**  
A: Allows arbitrary keys of a given type — `[key: string]: number` for a string-keyed map.

**5.6. Do interfaces exist at runtime?**  
A: No. They are erased during compilation — zero runtime footprint.

**5.7. When would I augment an interface?**  
A: Extending third-party types (e.g. adding fields to `Express.Request`) via declaration merging in a `.d.ts` file.

**5.8. Interface vs type performance?**  
A: No runtime difference. Compile-time performance is similar; pick based on expressiveness and team style.
