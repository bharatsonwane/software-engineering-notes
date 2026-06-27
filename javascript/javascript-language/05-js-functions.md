# 05. Functions

> File 05 — defining and using functions.

A **function** is a reusable block of code that can take **inputs** (parameters), perform work, and optionally **return** a value. In JavaScript, functions are **first-class** — you can assign them to variables, pass them as arguments, and return them from other functions.

Functions create their own **scope** (File 02). How you define a function affects **hoisting** and, for arrow functions, **`this`** binding (File 08).

**Default rule today:** use **function declarations** for top-level named helpers, **arrow functions** for short callbacks and when you want lexical `this`, and **default/rest parameters** instead of the legacy `arguments` object.

---

## 1.1. Function forms

### 1.1.1. Function declarations

Defined with the `function` keyword at the top level of a scope. **Hoisted** — you can call it before its line in the source (File 02, section 1.3.3).

```js
greet("Bharat");  // works — hoisted

function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet("Dev"));  // "Hello, Dev!"
```

Creates a **named binding** in the current scope (`greet` above).

### 1.1.2. Function expressions

A function assigned to a variable (or passed inline). The **variable** is hoisted, not the function value — same rules as `const`/`let`/`var` (File 02).

```js
// greet();  // ReferenceError or TypeError — depends on var vs const

const greet = function (name) {
  return `Hello, ${name}!`;
};

console.log(greet("Bharat"));
```

**Named function expression** — useful for stack traces and recursion inside the function:

```js
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);
};

factorial(5);  // 120
// fact(5);    // ReferenceError — name only visible inside
```

| Declaration | Hoisted as callable? | Typical use |
|-------------|----------------------|-------------|
| Function declaration | Yes | Top-level utilities, hoisting needed |
| `const` expression | No (TDZ until line) | Most assignments |
| `var` expression | No (`undefined` until line) | Legacy — avoid |

### 1.1.3. Arrow functions

Shorter syntax, introduced in ES6. **No own `this`**, `arguments`, or `prototype`. Cannot be used as constructors (`new`).

```js
const add = (a, b) => a + b;

const double = x => x * 2;           // one param — parens optional

const log = () => console.log("hi"); // no params

const buildUser = (name, age) => {
  return { name, age };              // block body needs explicit return
};
```

Implicit return with expression body:

```js
const sum = (a, b) => a + b;
const makePoint = (x, y) => ({ x, y });  // parens around object literal
```

Arrow vs regular — quick comparison:

| Feature | Regular function | Arrow function |
|---------|------------------|----------------|
| `this` | Dynamic (File 08) | Lexical — inherits from enclosing scope |
| `arguments` | Yes | No — use rest `...args` |
| Hoisting | Yes (declaration) | No — expression only |
| `new` / constructor | Yes | No |
| `prototype` property | Yes | No |

```js
const obj = {
  name: "Bharat",
  regular: function () {
    return this.name;
  },
  arrow: () => {
    // `this` is NOT obj — usually global/window or outer scope
    return this?.name;
  },
};

obj.regular();  // "Bharat"
obj.arrow();    // undefined (in strict module/script)
```

See File 08 for `this` in depth.

### 1.1.4. Methods (object and class shorthand)

When a function is a property of an object, it is often called a **method**:

```js
const calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  },
};

calculator.add(5, 3);  // 8
```

ES6 **method shorthand** — same as `add: function(a, b) { }` but cleaner. `this` inside refers to the object (when called as `obj.add()`).

### 1.1.5. When to use which

| Situation | Prefer |
|-----------|--------|
| Top-level named helper in a file | Function declaration or `const fn = () =>` |
| Callback in `map`, `filter`, `setTimeout` | Arrow function |
| Object method that needs `this` | Regular method shorthand |
| Generator function | `function*` (File 11) |
| Constructor / class | `class` or `function` + `new` (File 09) |

---

## 1.2. Parameters & arguments

### 1.2.1. Parameters vs arguments

- **Parameters** — names listed in the function **definition**.
- **Arguments** — actual **values** passed at **call time**.

```js
function multiply(a, b) {   // a, b are parameters
  return a * b;
}

multiply(3, 4);             // 3 and 4 are arguments
```

JavaScript does **not** enforce arity — you can pass fewer or more arguments than parameters:

```js
function log(a, b, c) {
  console.log(a, b, c);
}

log(1);           // 1, undefined, undefined
log(1, 2, 3, 4);  // 4 is ignored — no error
```

Extra arguments are available via **rest** or (legacy) **`arguments`**.

### 1.2.2. Default parameters

Default values apply when the argument is **`undefined`** (not when it is `null` or `0` unless you want that).

```js
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}

greet();              // "Hello, Guest!"
greet("Bharat");      // "Hello, Bharat!"
greet(undefined);     // "Hello, Guest!"
greet(null);          // "Hello, null!" — null is not undefined
```

Defaults can be **expressions** evaluated at call time:

```js
function createId(prefix = "id", suffix = Date.now()) {
  return `${prefix}-${suffix}`;
}
```

Later defaults can reference earlier parameters:

```js
function range(start, end = start + 10) {
  return { start, end };
}

range(5);  // { start: 5, end: 15 }
```

### 1.2.3. Rest parameters

**Rest** (`...name`) collects **remaining arguments** into a **real array**:

```js
function sum(...nums) {
  return nums.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4);  // 10
```

Combine fixed params with rest:

```js
function log(level, ...messages) {
  console.log(`[${level}]`, ...messages);
}

log("ERROR", "failed", "timeout");
```

Rest must be the **last** parameter:

```js
// function bad(a, ...rest, b) { }  // SyntaxError
```

Use rest instead of **`arguments`** in modern code.

### 1.2.4. The `arguments` object (legacy)

Non-arrow functions have an array-like **`arguments`** object with all passed values:

```js
function oldSum() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}

oldSum(1, 2, 3);  // 6
```

Limitations:

- Not a real array (no `map`/`filter` without converting)
- Not available in **arrow functions**
- Confusing with rest parameters

Prefer **`...rest`**:

```js
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}
```

### 1.2.5. Destructuring parameters

Unpack properties or array elements directly in the parameter list:

```js
function printUser({ name, age }) {
  console.log(name, age);
}

printUser({ name: "Bharat", age: 30 });
```

With defaults:

```js
function connect({ host = "localhost", port = 3000 } = {}) {
  console.log(`${host}:${port}`);
}

connect();                    // localhost:3000
connect({ port: 8080 });      // localhost:8080
```

Array destructuring:

```js
function firstAndSecond([a, b]) {
  return a + b;
}

firstAndSecond([10, 20]);  // 30
```

File 07 covers destructuring in more detail.

---

## 1.3. Return & first-class functions

### 1.3.1. return

**`return`** sends a value back to the caller and **stops** the function.

```js
function double(n) {
  return n * 2;
}

const result = double(5);  // 10
```

Without `return`, or with bare `return;`, the function returns **`undefined`**:

```js
function noop() {
  // implicit return undefined
}

function early() {
  return;
  console.log("never runs");
}
```

**Early return** — guard clauses keep logic flat:

```js
function processUser(user) {
  if (!user) return null;
  if (!user.isActive) return null;

  return { id: user.id, name: user.name };
}
```

Only one value per `return` — return multiple values with an **object** or **array**:

```js
function minMax(nums) {
  return { min: Math.min(...nums), max: Math.max(...nums) };
}

const { min, max } = minMax([3, 1, 4, 1, 5]);
```

### 1.3.2. Functions as values

Assign, store, and pass functions like any other value:

```js
const fn = function (x) { return x * 2; };

const ops = {
  add: (a, b) => a + b,
  sub: (a, b) => a - b,
};

function apply(op, a, b) {
  return op(a, b);
}

apply(ops.add, 5, 3);  // 8
```

Functions passed as arguments are **callbacks**:

```js
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, (i) => console.log(i));  // 0, 1, 2
```

File 10 covers async callbacks, Promises, and event handlers.

### 1.3.3. Higher-order functions

A **higher-order function** either **takes** a function as an argument or **returns** a function (or both).

```js
// Takes a function
function filter(arr, predicate) {
  const result = [];
  for (const item of arr) {
    if (predicate(item)) result.push(item);
  }
  return result;
}

filter([1, 2, 3, 4], (n) => n % 2 === 0);  // [2, 4]
```

```js
// Returns a function
function multiplyBy(factor) {
  return function (n) {
    return n * factor;
  };
}

const triple = multiplyBy(3);
triple(5);  // 15
```

Built-in higher-order array methods (`map`, `filter`, `reduce`) are covered in File 06.

### 1.3.4. Returning functions and closures (intro)

Inner functions **close over** variables from the outer scope — a **closure**:

```js
function makeCounter(start = 0) {
  let count = start;

  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter(10);
counter();  // 11
counter();  // 12
```

The returned function keeps access to `count` even after `makeCounter` finishes. File 08 covers closures, IIFEs, and common patterns in depth.

### 1.3.5. Pure vs impure functions (brief)

| Pure function | Impure function |
|---------------|-----------------|
| Same inputs → same output | Depends on or mutates external state |
| No side effects | Logs, DOM updates, network calls, mutates args |
| Easier to test and reason about | Often necessary in apps |

```js
// Pure
function add(a, b) {
  return a + b;
}

// Impure — mutates external array
function pushItem(arr, item) {
  arr.push(item);
  return arr;
}
```

Prefer pure functions where practical; isolate side effects at boundaries (API calls, UI updates).

---

## 1.4. Common questions

**1.4.1. What is the difference between a function declaration and a function expression?**  
A: Declarations are hoisted and create a named binding you can call before the line. Expressions assign a function to a variable — the variable follows `let`/`const`/`var` hoisting rules, not full function hoisting.

**1.4.2. When should I use arrow functions?**  
A: Short callbacks, when you want lexical `this`, and when you do not need `arguments`, `new`, or a `prototype`. Use regular functions or method shorthand when `this` must refer to the call site or object.

**1.4.3. What is the difference between parameters and arguments?**  
A: **Parameters** are in the function definition; **arguments** are the values passed when calling. JS allows any number of arguments regardless of parameter count.

**1.4.4. How do default parameters work with `null`?**  
A: Defaults apply only when the argument is **`undefined`**, not `null`. `fn(null)` uses `null`, not the default.

**1.4.5. What is the rest parameter and how is it different from `arguments`?**  
A: Rest (`...args`) collects remaining arguments into a **real array** and works in all modern function forms. `arguments` is array-like, legacy, and unavailable in arrow functions.

**1.4.6. What does a function return if there is no `return`?**  
A: **`undefined`**.

**1.4.7. What is a first-class function?**  
A: Functions are values — you can assign them to variables, pass them as arguments, return them, and store them in data structures.

**1.4.8. What is a higher-order function?**  
A: A function that takes another function as an argument, returns a function, or both. Examples: `map`, `filter`, `setTimeout`, and custom helpers like `multiplyBy(factor)`.
