# 04. Control Flow

> File 04 — branching and loops.

**Control flow** decides which code runs and how many times. JavaScript gives you **conditionals** (`if`, ternary, `switch`) to branch on a condition, and **loops** (`for`, `while`, `for...of`, etc.) to repeat work.

Conditions use **truthy/falsy** coercion (see File 03, section 1.4). A block runs when its condition evaluates to a truthy value.

**Default rule today:** prefer **`if`/`else`** for clarity, **`===`** in conditions, **`for...of`** for arrays and iterables, and **`for...in`** only when you intentionally need object keys (not for arrays).

---

## 1.1. Conditionals

### 1.1.1. if / else if / else

Run one branch based on a condition. Only the **first** branch whose condition is truthy executes.

```js
const score = 85;

if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");   // runs
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("F");
}
```

Conditions are expressions coerced to boolean:

```js
const name = "Bharat";

if (name) {
  console.log(`Hello, ${name}`);  // runs — non-empty string is truthy
}

if (0) {
  // skipped — 0 is falsy
}
```

Optional braces for single statements (avoid in team code — easy to break during edits):

```js
if (x > 0) console.log("positive");  // works, but prefer braces
```

Block scope with `let`/`const` — each branch is its own block:

```js
if (true) {
  let message = "inside";
  console.log(message);
}
// message — ReferenceError outside the block
```

### 1.1.2. Ternary operator

Shorthand for simple **if/else** that **returns a value**:

```js
const age = 17;
const status = age >= 18 ? "adult" : "minor";

console.log(status);  // "minor"
```

Syntax: `condition ? valueIfTrue : valueIfFalse`

Nested ternaries work but hurt readability — use `if/else` for complex logic:

```js
const n = 0;
const label = n > 0 ? "positive" : n < 0 ? "negative" : "zero";
// Prefer if/else when logic grows beyond one line
```

Ternary vs `if`:

| Use ternary when | Use if/else when |
|------------------|------------------|
| Picking a value in an expression | Running multiple statements |
| Logic fits on one line | Side effects (logging, early return) |
| Assigning or returning | Complex branching |

```js
// Good ternary
const max = a > b ? a : b;

// Better as if/else
if (user.isAdmin) {
  logAccess(user);
  grantAdminPanel(user);
}
```

### 1.1.3. switch

Compare one **expression** against multiple **case** values. Uses **strict equality** (`===`) internally.

```js
const day = "Mon";

switch (day) {
  case "Mon":
  case "Tue":
  case "Wed":
  case "Thu":
  case "Fri":
    console.log("Weekday");
    break;
  case "Sat":
  case "Sun":
    console.log("Weekend");
    break;
  default:
    console.log("Unknown");
}
```

**`break`** exits the `switch`. Without it, execution **falls through** to the next case:

```js
const grade = "B";

switch (grade) {
  case "A":
  case "B":
    console.log("Pass");   // runs for B
    break;
  case "C":
    console.log("Marginal");
    break;
  default:
    console.log("Fail");
}
```

Intentional fall-through (rare — comment why):

```js
switch (n) {
  case 1:
    setup();
  // fall through
  case 2:
    run();
    break;
}
```

`switch` vs `if/else` chain:

| Prefer `switch` | Prefer `if/else` |
|-----------------|------------------|
| Many checks on **same value** | Range checks (`score >= 90`) |
| Discrete string/number cases | Mixed conditions |
| Readable case groups | Few branches (2–3) |

```js
// switch — good for discrete values
switch (httpStatus) {
  case 404: /* ... */ break;
  case 500: /* ... */ break;
}

// if/else — good for ranges
if (score >= 90) { /* A */ }
else if (score >= 80) { /* B */ }
```

---

## 1.2. Loops

### 1.2.1. for loop

Classic **C-style** loop: init, condition, update.

```js
for (let i = 0; i < 5; i++) {
  console.log(i);  // 0, 1, 2, 3, 4
}
```

All three parts are optional:

```js
let i = 0;
for (; i < 3; ) {
  console.log(i);
  i++;
}

for (;;) {
  // infinite loop — need break inside
}
```

Multiple variables in init/update (comma-separated):

```js
for (let i = 0, j = 10; i < j; i++, j--) {
  console.log(i, j);
}
```

`let` in the header creates a **block-scoped** binding per iteration (important for closures — see File 02):

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);  // 0, 1, 2
}
```

### 1.2.2. while

Test condition **before** each iteration. May run **zero** times.

```js
let count = 3;

while (count > 0) {
  console.log(count);
  count--;
}
// 3, 2, 1
```

Use when the number of iterations is **unknown** upfront:

```js
let input;
while (input !== "quit") {
  input = getNextInput();  // pseudo
  process(input);
}
```

Watch for **infinite loops** if the condition never becomes falsy.

### 1.2.3. do...while

Test condition **after** each iteration. Runs **at least once**.

```js
let n = 0;

do {
  console.log(n);
  n++;
} while (n < 3);
// 0, 1, 2
```

```js
let choice;
do {
  choice = promptUser();  // pseudo — always ask at least once
} while (choice !== "yes" && choice !== "no");
```

| Loop | Condition checked | Min iterations |
|------|-------------------|----------------|
| `while` | Before | 0 |
| `do...while` | After | 1 |

### 1.2.4. for...of

Iterate over **iterable** values (arrays, strings, Map, Set, NodeList, etc.). Gets **values**, not indices.

```js
const nums = [10, 20, 30];

for (const n of nums) {
  console.log(n);  // 10, 20, 30
}

for (const char of "hi") {
  console.log(char);  // "h", "i"
}
```

With **`entries()`** for index + value:

```js
for (const [index, value] of nums.entries()) {
  console.log(index, value);  // 0 10, 1 20, 2 30
}
```

`for...of` does **not** work on plain objects (not iterable by default):

```js
const obj = { a: 1, b: 2 };
// for (const x of obj) { }  // TypeError

for (const [key, value] of Object.entries(obj)) {
  console.log(key, value);
}
```

Prefer `for...of` over indexed `for` when you do not need the index:

```js
// Prefer
for (const item of list) { /* use item */ }

// When index matters
for (const [i, item] of list.entries()) { }
```

### 1.2.5. for...in

Iterate over **enumerable property keys** of an object (including inherited keys unless filtered).

```js
const user = { name: "Bharat", age: 30 };

for (const key in user) {
  console.log(key, user[key]);
  // name Bharat, age 30
}
```

**Do not use `for...in` on arrays** — order and extra keys (e.g. custom properties) cause bugs:

```js
const arr = [1, 2, 3];
arr.custom = "oops";

for (const key in arr) {
  console.log(key);  // "0", "1", "2", "custom" — often wrong
}
```

Use **`hasOwnProperty`** or **`Object.hasOwn`** when iterating object keys:

```js
for (const key in user) {
  if (Object.hasOwn(user, key)) {
    console.log(key, user[key]);  // skips inherited enumerable keys
  }
}
```

| Loop | Iterates over | Best for |
|------|---------------|----------|
| `for...of` | Iterable **values** | Arrays, strings, Map, Set |
| `for...in` | Object **keys** (enumerable) | Plain objects (own keys only) |
| Classic `for` | Index counter | When you need full index control |

### 1.2.6. Loop comparison summary

```js
const arr = ["a", "b", "c"];

// Index loop
for (let i = 0; i < arr.length; i++) {
  console.log(i, arr[i]);
}

// Values (preferred for arrays)
for (const val of arr) {
  console.log(val);
}

// Keys (avoid on arrays)
for (const key in arr) {
  console.log(key);
}

// Functional style (File 06 covers in depth)
arr.forEach((val, i) => console.log(i, val));
```

---

## 1.3. break & continue

### 1.3.1. break

**`break`** exits the **innermost** loop or `switch` immediately.

```js
for (let i = 0; i < 10; i++) {
  if (i === 5) break;
  console.log(i);  // 0, 1, 2, 3, 4
}
```

```js
switch (n) {
  case 1:
    doSomething();
    break;   // required unless fall-through is intentional
  case 2:
    doOther();
    break;
}
```

Search pattern — break when found:

```js
const users = [{ id: 1 }, { id: 2 }, { id: 3 }];
let target = null;

for (const user of users) {
  if (user.id === 2) {
    target = user;
    break;
  }
}
```

Often **`find()`** or **`some()`** is clearer than `break` in arrays (File 06).

### 1.3.2. continue

**`continue`** skips the **rest of the current iteration** and moves to the next one.

```js
for (let i = 0; i < 5; i++) {
  if (i % 2 === 0) continue;  // skip evens
  console.log(i);  // 1, 3
}
```

```js
for (const user of users) {
  if (!user.isActive) continue;
  sendEmail(user);  // only active users
}
```

`continue` in `while` / `do...while` — jumps to the **condition check**:

```js
let i = 0;
while (i < 5) {
  i++;
  if (i === 3) continue;
  console.log(i);  // 1, 2, 4, 5
}
```

### 1.3.3. Labeled break & continue (brief)

**Labels** name a loop so `break` / `continue` can target an **outer** loop. Rare in everyday code.

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer;
    console.log(i, j);
  }
}
// stops entirely when i=1, j=1
```

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 0) continue outer;  // skip rest of inner when j=0
    console.log(i, j);
  }
}
```

Prefer **functions** or **early return** over labeled loops when logic gets nested:

```js
function findPair(matrix, target) {
  for (const row of matrix) {
    for (const cell of row) {
      if (cell === target) return cell;  // clearer than break outer
    }
  }
  return null;
}
```

---

## 1.4. Common questions

**1.4.1. What is the difference between `if` and ternary?**  
A: Both branch on a condition. `if` runs **statements**; ternary **returns a value** as an expression. Use ternary for simple assignments; use `if` for multi-line logic and side effects.

**1.4.2. Does `switch` use `==` or `===`?**  
A: **`===`**. Type and value must match the case. `"1"` does not match case `1`.

**1.4.3. What happens if I forget `break` in a `switch`?**  
A: Execution **falls through** to the next case until a `break` or the end of `switch`. Sometimes intentional; usually a bug.

**1.4.4. What is the difference between `while` and `do...while`?**  
A: `while` checks the condition **before** the first iteration (may run 0 times). `do...while` checks **after** (always runs at least once).

**1.4.5. When should I use `for...of` vs classic `for`?**  
A: Use **`for...of`** when you need each **value** from an iterable. Use classic **`for`** when you need the index with full control, or when iterating backwards / by steps.

**1.4.6. What is the difference between `for...in` and `for...of`?**  
A: `for...in` loops over **enumerable property keys** (objects). `for...of` loops over **iterable values** (arrays, strings, Map, Set). Do not use `for...in` on arrays.

**1.4.7. What do `break` and `continue` do?**  
A: `break` exits the innermost loop or `switch`. `continue` skips the rest of the **current** loop iteration and starts the next one.

**1.4.8. Why does `for (let i = 0; ...)` behave differently from `for (var i = 0; ...)` in closures?**  
A: `let` creates a **new binding per iteration**. `var` is function-scoped, so all callbacks share one `i`. See File 02, section 1.2.3.
