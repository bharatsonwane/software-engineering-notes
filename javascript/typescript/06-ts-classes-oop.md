# 06. Classes & OOP

> File 06 — classes, access modifiers, implements, extends, abstract.

TypeScript adds type information to JavaScript classes — visibility modifiers, abstract classes, and typed `implements` / `extends`.

---

## 1. Classes with types

```ts
class User {
  id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  greet(): string {
    return `Hello, ${this.name}`;
  }
}

const u = new User(1, "Bharat");
```

### 1.1. Parameter properties (shorthand)

```ts
class User {
  constructor(
    public id: number,
    public name: string,
    private passwordHash: string
  ) {}
}
```

TypeScript emits standard JS fields and constructor assignments.

### 1.2. Readonly fields

```ts
class Config {
  constructor(public readonly apiUrl: string) {}
}
```

---

## 2. `public`, `private`, `protected`

| Modifier | Class | Subclass | Outside |
|----------|-------|----------|---------|
| `public` (default) | Yes | Yes | Yes |
| `protected` | Yes | Yes | No |
| `private` | Yes | No | No |

```ts
class Animal {
  public name: string;
  protected age: number;
  private secret: string;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
    this.secret = "hidden";
  }
}

class Dog extends Animal {
  bark() {
    console.log(this.name);  // OK — public
    console.log(this.age);   // OK — protected
    // console.log(this.secret); // Error — private
  }
}
```

### 2.1. ECMAScript `#private` fields

JS native private fields work alongside TS modifiers:

```ts
class Account {
  #balance = 0;

  deposit(amount: number) {
    this.#balance += amount;
  }
}
```

`#balance` is **true runtime private**; TS `private` is compile-time only.

---

## 3. `implements` and `extends`

### 3.1. Implementing interfaces

```ts
interface Serializable {
  serialize(): string;
}

interface Timestamped {
  createdAt: Date;
}

class Note implements Serializable, Timestamped {
  createdAt = new Date();

  constructor(public body: string) {}

  serialize(): string {
    return JSON.stringify({ body: this.body, createdAt: this.createdAt });
  }
}
```

`implements` checks that the class satisfies the **shape** — no code is inherited.

### 3.2. Extending classes

```ts
class Employee extends User {
  constructor(
    id: number,
    name: string,
    public department: string
  ) {
    super(id, name);
  }

  override greet(): string {
    return `${super.greet()} from ${this.department}`;
  }
}
```

Use **`override`** (with `noImplicitOverride`) when replacing a base method.

---

## 4. Abstract classes

**Abstract** classes cannot be instantiated directly — they define a partial implementation for subclasses:

```ts
abstract class Shape {
  abstract area(): number;

  describe(): string {
    return `Area: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

// new Shape(); // Error — abstract
const c = new Circle(5);
```

### 4.1. Abstract class vs interface

| Abstract class | Interface |
|----------------|-----------|
| Can have implemented methods | Methods only as signatures (unless default in TS 5+) |
| Single inheritance (`extends`) | Multiple `implements` |
| Exists at runtime | Erased at compile time |
| Use when sharing base behavior | Use for pure contracts |

### 4.2. Static members

```ts
class MathUtil {
  static PI = 3.14159;

  static clamp(value: number, min: number, max: number): number {
    return Math.min(max, Math.max(min, value));
  }
}

MathUtil.clamp(10, 0, 5); // 5
```

---

## 5. Common questions

**5.1. Does TypeScript support multiple class inheritance?**  
A: No — one `extends` only. Use `implements` for multiple interfaces.

**5.2. Is `private` enforced at runtime?**  
A: TS `private`/`protected` are compile-time only. Use `#field` for runtime privacy.

**5.3. When use abstract class over interface?**  
A: When subclasses share concrete helper methods or fields; interfaces for pure shapes.

**5.4. Can classes implement type aliases?**  
A: Yes, if the alias resolves to an object type with members to satisfy.

**5.5. What is `override` for?**  
A: Ensures you intentionally replace a base method — catches typos in method names.

**5.6. Are classes the preferred pattern in TS?**  
A: Many TS/React codebases favor **functions + interfaces** over classes. Use classes when modeling stateful entities or extending frameworks (Angular, some ORMs).

**5.7. How do generics work with classes?**  
A: `class Box<T> { constructor(public value: T) {} }` — same as generic functions (File 04).

**5.8. Class fields vs constructor assignment — any difference?**  
A: Parameter properties are syntactic sugar. Both compile to similar JS depending on target.
