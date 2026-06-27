# 01 Types & Values

> File 01 — primitives, objects, and how values work in memory.

JavaScript is **dynamically typed**: a variable can hold any type, and the type is tied to the **value**, not the variable.

```js
let x = 10;      // number
x = "hello";     // now string — totally valid
```

There are two broad categories: **primitives** and **objects** (everything else).

---

## 1.1 Primitives

A **primitive** is a single, immutable value stored directly. JavaScript has **7 primitive types**:

| Type | Example | Notes |
|------|---------|-------|
| `string` | `"hello"`, `'world'`, `` `hi` `` | Text; template literals use backticks |
| `number` | `42`, `3.14`, `NaN`, `Infinity` | All numbers are 64-bit floats (no separate `int`) |
| `boolean` | `true`, `false` | Result of comparisons and logic |
| `undefined` | `undefined` | Variable declared but not assigned |
| `null` | `null` | Intentional absence of value |
| `symbol` | `Symbol("id")` | Unique identifier, ES6+ |
| `bigint` | `100n`, `BigInt(100)` | Arbitrarily large integers, ES2020+ |

### 1.1.1 Strings, numbers, booleans

```js
const name = "Bharat";
const age = 30;
const isActive = true;

// Template literals
const greeting = `Hello, ${name}!`;
```

### 1.1.2 `undefined` vs `null`

```js
let a;           // undefined — no value assigned
let b = null;    // null — explicitly empty

console.log(a);  // undefined
console.log(b);  // null
```

- **`undefined`** — JS sets this automatically (unassigned variable, missing function return, missing object property).
- **`null`** — you set this when you mean "no value on purpose".

### 1.1.3 `NaN` and `Infinity`

```js
typeof NaN;        // "number" (quirk!)
Number("abc");     // NaN
1 / 0;             // Infinity
```

`NaN` is the only value that is not equal to itself:

```js
NaN === NaN;       // false
Number.isNaN(NaN); // true — prefer this over global isNaN()
```

### 1.1.4 `symbol`

Used for unique property keys, often to avoid name collisions:

```js
const id = Symbol("id");
const user = { [id]: 123, name: "Bharat" };
```

Every `Symbol()` call creates a **new** unique value, even with the same description.

### 1.1.5 `bigint`

For integers larger than `Number.MAX_SAFE_INTEGER` (`2^53 - 1`):

```js
const big = 9007199254740991n;
typeof big;        // "bigint"

// Cannot mix bigint and number without conversion
// 1n + 1  → TypeError
```

### 1.1.6 Primitives are immutable

You cannot change a primitive in place — operations return **new** values:

```js
const s = "hello";
s.toUpperCase();   // "HELLO"
console.log(s);    // still "hello"
```

---

## 1.2 Objects & references

An **object** is a collection of key–value pairs. In JS, **arrays** and **functions** are also objects.

```js
const person = { name: "Bharat", age: 30 };
const nums = [1, 2, 3];
const greet = function () { return "hi"; };

typeof person;  // "object"
typeof nums;    // "object"
typeof greet;   // "function"
```

### 1.2.1 Value copy vs reference copy

**Primitives** are copied **by value**:

```js
let a = 10;
let b = a;   // b gets a copy of the value
b = 20;
console.log(a);  // 10
console.log(b);  // 20
```

**Objects** are copied **by reference** — the variable holds a pointer to the same object in memory:

```js
const obj1 = { x: 1 };
const obj2 = obj1;   // both point to the same object
obj2.x = 99;
console.log(obj1.x); // 99 — obj1 changed too!
```

### 1.2.2 Shallow copy

Spread and `Object.assign` create a **shallow** copy — top-level keys are copied, nested objects are still shared:

```js
const original = { a: 1, nested: { b: 2 } };
const copy = { ...original };

copy.a = 100;
copy.nested.b = 200;

console.log(original.a);          // 1
console.log(original.nested.b);   // 200 — nested object is shared!
```

For a deep copy of plain data, `structuredClone()` works in modern environments:

```js
const deep = structuredClone(original);
```

### 1.2.3 Passing to functions

Primitives are passed by value; objects are passed by reference (the reference is copied):

```js
function changePrimitive(n) {
  n = 100;
}
let num = 5;
changePrimitive(num);
console.log(num);  // 5

function changeObject(obj) {
  obj.x = 100;
}
const o = { x: 1 };
changeObject(o);
console.log(o.x);  // 100
```

### 1.2.4 Arrays are objects

```js
const arr = [1, 2, 3];
arr.push(4);
typeof arr;       // "object"
Array.isArray(arr); // true — use this to check for arrays
```

---

## 1.3 typeof & type checking

### 1.3.1 `typeof` operator

Returns a **string** naming the type:

```js
typeof "hello";     // "string"
typeof 42;          // "number"
typeof true;        // "boolean"
typeof undefined;   // "undefined"
typeof Symbol();    // "symbol"
typeof 10n;         // "bigint"
typeof function(){};// "function"
typeof {};          // "object"
typeof [];          // "object"
typeof null;        // "object"  ← famous bug, historical reason
```

### 1.3.2 Checking for `null`

`typeof null === "object"` is misleading. Use strict equality:

```js
const value = null;
value === null;  // true
```

### 1.3.3 `instanceof`

Checks if an object appears in another object's prototype chain:

```js
[] instanceof Array;        // true
[] instanceof Object;       // true
({}) instanceof Object;   // true

"hello" instanceof String; // false — primitive, not String object
```

### 1.3.4 Practical type checks

| Goal | How |
|------|-----|
| Is it an array? | `Array.isArray(x)` |
| Is it `NaN`? | `Number.isNaN(x)` |
| Is it a finite number? | `Number.isFinite(x)` |
| Is it `null` or `undefined`? | `x == null` or `x === null \|\| x === undefined` |
| Safe integer? | `Number.isSafeInteger(x)` |

### 1.3.5 Boxing (brief)

Primitives can be temporarily wrapped as objects when you access methods:

```js
"hello".toUpperCase();  // works — JS wraps in String object, then discards it
```

Avoid `new String("hello")` — creates an object wrapper, rarely needed.

---

## 1.4 Common questions

**1.4.1 How many data types does JavaScript have?**  
A: **7 primitives** + **objects** (which includes arrays, functions, dates, etc.).

**1.4.2 What is the difference between `undefined` and `null`?**  
A: `undefined` means "not assigned / missing". `null` means "intentionally empty".

**1.4.3 Why does `typeof null` return `"object"`?**  
A: A long-standing language bug kept for backward compatibility. Always use `value === null` to check for null.

**1.4.4 Primitive vs object — what is the key difference?**  
A: Primitives are immutable single values, stored and copied by value. Objects are mutable collections, copied by reference.

**1.4.5 Does `const` make objects immutable?**  
A: No. `const` prevents **reassigning** the variable. The object itself can still be mutated:

```js
const obj = { a: 1 };
obj.a = 2;       // OK
obj = { a: 3 };  // TypeError — cannot reassign
```

**1.4.6 What is `NaN` and how do you check for it?**  
A: `NaN` means "Not a Number" (result of invalid math). Use `Number.isNaN(value)`, not `value === NaN`.
