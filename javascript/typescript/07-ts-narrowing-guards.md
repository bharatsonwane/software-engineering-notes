# 07. Type Narrowing & Guards

> File 07 — typeof, instanceof, truthiness, type predicates, `in`.

**Narrowing** refines a broad type to a specific one inside a branch so TypeScript allows safe operations.

```ts
function print(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // string
  } else {
    console.log(value.toFixed(2));    // number
  }
}
```

---

## 1. `typeof` and `instanceof` narrowing

### 1.1. `typeof` (primitives)

Works well for: `string`, `number`, `boolean`, `bigint`, `symbol`, `undefined`, `function`.

```ts
function assertString(x: unknown) {
  if (typeof x === "string") {
    x.length; // OK
  }
}
```

**Caveat:** `typeof null === "object"` in JavaScript — combine with null check:

```ts
if (typeof value === "object" && value !== null) {
  // object (not null)
}
```

### 1.2. `instanceof` (objects / classes)

```ts
class ApiError extends Error {
  statusCode: number;
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
  }
}

function handle(err: unknown) {
  if (err instanceof ApiError) {
    console.log(err.statusCode);
  } else if (err instanceof Error) {
    console.log(err.message);
  }
}
```

`instanceof` checks the prototype chain — useful for class instances, not plain object shapes.

---

## 2. Truthiness narrowing

Falsy values in JS: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`.

```ts
function printName(name: string | null | undefined) {
  if (name) {
    console.log(name.toUpperCase());
  }
}
```

### 2.1. Equality narrowing

```ts
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // x and y are both string here
  }
}
```

### 2.2. `== null` / `!= null`

Checks both `null` and `undefined`:

```ts
if (value != null) {
  // value is neither null nor undefined
}
```

---

## 3. Type predicates (`is`)

A **type predicate** tells TypeScript a boolean function narrows the type:

```ts
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();
  } else {
    pet.fly();
  }
}
```

Custom guards are essential for **discriminated unions** without a single tag field:

```ts
function isSuccess<T>(
  result: { ok: true; data: T } | { ok: false; error: string }
): result is { ok: true; data: T } {
  return result.ok === true;
}
```

---

## 4. `in` operator narrowing

Check if a property exists on an object:

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function speak(pet: Cat | Dog) {
  if ("meow" in pet) {
    pet.meow();
  } else {
    pet.bark();
  }
}
```

Works with discriminated unions:

```ts
type Action =
  | { type: "increment"; amount: number }
  | { type: "reset" };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case "increment":
      return state + action.amount;
    case "reset":
      return 0;
  }
}
```

### 4.1. Assignment narrowing

```ts
let x: string | number = "hello";
x = 42; // now narrowed to number in following code
```

### 4.2. Control flow analysis

TypeScript tracks narrowing across `if`, `else`, `switch`, early `return`, and `throw`:

```ts
function process(value: string | null) {
  if (!value) {
    return;
  }
  // value is string here
  console.log(value.length);
}
```

---

## 5. Common questions

**5.1. What is narrowing?**  
A: Refining a union or `unknown` to a specific type inside a conditional branch.

**5.2. `typeof` vs `instanceof`?**  
A: `typeof` for primitives (and functions). `instanceof` for class instances.

**5.3. What is a type predicate?**  
A: Return type `arg is Type` on a boolean function — custom narrowing for TS.

**5.4. Why doesn't `typeof` narrow `null` correctly?**  
A: JS quirk — `typeof null === "object"`. Always add `value !== null`.

**5.5. Does narrowing work across function boundaries?**  
A: Generally **no** — TS does not track mutations on outer variables inside nested functions (unless const or proven immutable).

**5.6. How do I narrow `unknown` from `JSON.parse`?**  
A: Validate with `typeof`, `Array.isArray`, custom type guards, or libraries like Zod/io-ts.

**5.7. What is discriminant narrowing?**  
A: Switching on a shared field (`kind`, `type`, `status`) to narrow tagged unions (File 05).

**5.8. Can I assert a type without narrowing?**  
A: Type assertions (`as Type`) force TS to trust you — no runtime check. Prefer guards when data is external.
