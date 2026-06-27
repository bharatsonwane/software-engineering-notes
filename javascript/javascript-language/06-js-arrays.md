# 06. Arrays

> File 06 — working with lists of values.

An **array** is an ordered list of values. Arrays are **objects** (File 01) — they are copied and passed **by reference**, and you check them with `Array.isArray()`, not `typeof`.

Indexes are **zero-based**: first element is `[0]`, last is `[length - 1]`. Arrays are **mutable** — most methods below change the array in place unless noted.

**Default rule today:** use **`for...of`** or **`map`/`filter`/`reduce`** instead of manual index loops when possible; use **`spread`** or **`slice()`** to copy before mutating shared data; prefer **`find`/`some`/`every`** over manual search loops.

---

## 1.1. Creating & accessing

### 1.1.1. Creating arrays

```js
// Array literal (most common)
const nums = [1, 2, 3];

// Empty array
const empty = [];

// Mixed types — allowed but usually avoid in typed thinking
const mixed = [1, "hello", true, { x: 1 }];

// Array constructor (rare — can confuse)
const arr = new Array(3);     // [empty × 3] — sparse, holes
const withValues = new Array(1, 2, 3);  // [1, 2, 3]

// From iterable or array-like
const fromStr = Array.from("hi");   // ["h", "i"]
const copy = Array.from([1, 2]);    // [1, 2]

// Fill fixed length
const zeros = Array(5).fill(0);     // [0, 0, 0, 0, 0]
```

### 1.1.2. Index access

```js
const fruits = ["apple", "banana", "cherry"];

fruits[0];    // "apple"
fruits[2];    // "cherry"
fruits[10];   // undefined — no error, out of bounds

fruits.length;  // 3
```

**Negative index** — not built-in; use `at()` (ES2022+):

```js
fruits.at(-1);   // "cherry" — last
fruits.at(-2);   // "banana"
```

Assign by index — grows array if needed:

```js
const arr = [1, 2];
arr[5] = 99;
console.log(arr);       // [1, 2, empty × 3, 99]
console.log(arr.length);  // 6
```

Sparse arrays (holes) — avoid when possible; some methods skip holes:

```js
const sparse = [1, , 3];  // hole at index 1
sparse.length;            // 3
sparse[1];                // undefined
```

### 1.1.3. length property

`length` is the number of slots (including empty ones), not always "count of defined values":

```js
const arr = [10, 20, 30];
arr.length;        // 3

arr.length = 2;    // truncates — [10, 20]
arr.length = 5;    // extends with empty slots
```

Setting `length` shorter **removes** trailing elements:

```js
const nums = [1, 2, 3, 4, 5];
nums.length = 3;
console.log(nums);  // [1, 2, 3]
```

---

## 1.2. Mutating methods

These methods **change the original array**.

### 1.2.1. Add / remove at end — push & pop

```js
const stack = [1, 2];

stack.push(3);       // returns new length: 3
console.log(stack);  // [1, 2, 3]

stack.pop();         // returns removed: 3
console.log(stack);  // [1, 2]
```

`push` accepts multiple arguments:

```js
[1].push(2, 3, 4);   // [1, 2, 3, 4]
```

### 1.2.2. Add / remove at start — unshift & shift

```js
const queue = [2, 3];

queue.unshift(1);    // 3 — new length
console.log(queue);  // [1, 2, 3]

queue.shift();       // 1 — removed value
console.log(queue);  // [2, 3]
```

`shift` and `unshift` are **O(n)** — slower on large arrays than `push`/`pop`.

### 1.2.3. splice — insert, remove, replace

`arr.splice(start, deleteCount, ...items)` — versatile in-place editor:

```js
const arr = [1, 2, 3, 4, 5];

// Remove 2 elements starting at index 1
arr.splice(1, 2);           // returns [2, 3]; arr is [1, 4, 5]

// Insert at index 1
arr.splice(1, 0, 2, 3);     // arr is [1, 2, 3, 4, 5]

// Replace 1 element at index 2
arr.splice(2, 1, 99);       // arr is [1, 2, 99, 4, 5]
```

### 1.2.4. Other mutators

| Method | Effect |
|--------|--------|
| `reverse()` | Reverses order in place |
| `sort(compareFn?)` | Sorts in place (default: string Unicode order) |
| `fill(value, start?, end?)` | Sets range to same value |

```js
[3, 1, 4, 1, 5].sort();              // [1, 1, 3, 4, 5] — string sort!
[3, 1, 4, 1, 5].sort((a, b) => a - b);  // numeric: [1, 1, 3, 4, 5]

[1, 2, 3].reverse();                 // [3, 2, 1]

[0, 0, 0].fill(7, 1, 3);             // [0, 7, 7]
```

**Sort pitfall** — default sort converts to strings:

```js
[10, 2, 1].sort();           // [1, 10, 2] — wrong for numbers
[10, 2, 1].sort((a, b) => a - b);  // [1, 2, 10]
```

### 1.2.5. Non-mutating alternatives

| Mutating | Non-mutating alternative |
|----------|--------------------------|
| `push` / `pop` | `concat`, spread `[...arr, x]` |
| `splice` | `slice`, `toSpliced()` (ES2023+) |
| `reverse` | `toReversed()` (ES2023+) |
| `sort` | `toSorted(compareFn)` (ES2023+) |

```js
const original = [3, 1, 2];
const sorted = [...original].sort((a, b) => a - b);
console.log(original);  // [3, 1, 2] — unchanged
console.log(sorted);    // [1, 2, 3]
```

---

## 1.3. Iteration & transformation

Most take a **callback** `(element, index, array) => ...` (File 05). They skip **holes** in sparse arrays unless noted.

### 1.3.1. forEach

Run a function for each element. Returns **`undefined`**. Cannot `break` — use `for...of` if you need early exit (File 04).

```js
[1, 2, 3].forEach((n, i) => {
  console.log(i, n);
});
```

### 1.3.2. map

Transform each element → **new array** of same length:

```js
const doubled = [1, 2, 3].map((n) => n * 2);
console.log(doubled);  // [2, 4, 6]
```

```js
const users = [{ name: "A" }, { name: "B" }];
const names = users.map((u) => u.name);  // ["A", "B"]
```

### 1.3.3. filter

Keep elements where callback returns **truthy** → **new array** (possibly shorter):

```js
const evens = [1, 2, 3, 4].filter((n) => n % 2 === 0);
console.log(evens);  // [2, 4]
```

### 1.3.4. reduce

Fold array into a **single value** (or object/array accumulator):

```js
const sum = [1, 2, 3, 4].reduce((acc, n) => acc + n, 0);
console.log(sum);  // 10
```

Without initial value, first element becomes accumulator (risky on empty arrays):

```js
[1, 2, 3].reduce((acc, n) => acc + n);  // 6 — works here
// [].reduce((acc, n) => acc + n);      // TypeError on empty
```

Build objects:

```js
const pairs = [["a", 1], ["b", 2]];
const obj = pairs.reduce((acc, [k, v]) => {
  acc[k] = v;
  return acc;
}, {});
// { a: 1, b: 2 }
```

### 1.3.5. find, findIndex, findLast

| Method | Returns |
|--------|---------|
| `find(predicate)` | First matching **element**, or `undefined` |
| `findIndex(predicate)` | First matching **index**, or `-1` |
| `findLast(predicate)` | Last matching element (ES2023+) |
| `findLastIndex(predicate)` | Last matching index (ES2023+) |

```js
const users = [
  { id: 1, active: false },
  { id: 2, active: true },
  { id: 3, active: true },
];

users.find((u) => u.active);       // { id: 2, active: true }
users.findIndex((u) => u.id === 3); // 2
users.findLast((u) => u.active);   // { id: 3, active: true }
```

Prefer `find` over `filter()[0]` — stops at first match.

### 1.3.6. some & every

Short-circuit boolean tests:

```js
[1, 2, 3].some((n) => n > 2);    // true — at least one
[1, 2, 3].every((n) => n > 0);   // true — all

[].some(() => true);   // false
[].every(() => false); // true — vacuous truth on empty
```

### 1.3.7. Other useful methods (non-mutating)

| Method | Purpose |
|--------|---------|
| `includes(value, fromIndex?)` | Strict equality search (`NaN` works) |
| `indexOf` / `lastIndexOf` | First/last index of value (`NaN` fails) |
| `slice(start?, end?)` | Copy portion — **new array** |
| `concat(...arrays)` | Merge into **new array** |
| `flat(depth?)` | Flatten nested arrays |
| `flatMap(fn)` | `map` then `flat(1)` |
| `join(separator?)` | String from elements |

```js
[1, 2, 3].includes(2);           // true
[1, 2, NaN].includes(NaN);       // true
[1, 2, NaN].indexOf(NaN);        // -1

[1, 2, 3, 4].slice(1, 3);        // [2, 3]
[1, [2, 3]].flat();              // [1, 2, 3]

[1, 2, 3].join("-");             // "1-2-3"
```

### 1.3.8. Method cheat sheet

| Goal | Method |
|------|--------|
| Loop, no new array | `forEach`, `for...of` |
| Transform → new array | `map` |
| Subset → new array | `filter` |
| Single result | `reduce` |
| First match | `find` |
| Any / all test | `some` / `every` |
| Copy part | `slice` |
| Search by value | `includes`, `indexOf` |

---

## 1.4. Spread & copy

### 1.4.1. Shallow copy

Arrays hold **references** to objects (File 01). Copying duplicates the **array**, not nested objects:

```js
const original = [1, { x: 1 }];
const copy = [...original];       // or original.slice()

copy[0] = 99;
copy[1].x = 99;

console.log(original[0]);   // 1
console.log(original[1].x); // 99 — nested object shared!
```

Deep copy plain data: `structuredClone(original)` (File 01).

### 1.4.2. Spread in arrays

**Copy:**

```js
const a = [1, 2, 3];
const b = [...a];
```

**Merge:**

```js
const first = [1, 2];
const second = [3, 4];
const combined = [...first, ...second];  // [1, 2, 3, 4]
```

**Insert / prepend:**

```js
const arr = [2, 3];
const withHead = [1, ...arr];      // [1, 2, 3]
const withMid = [...arr, 4, 5];    // [2, 3, 4, 5]
```

Spread in function calls (File 05):

```js
Math.max(...[1, 5, 3]);   // 5
```

### 1.4.3. concat vs spread

Both produce new arrays:

```js
[1, 2].concat([3, 4]);     // [1, 2, 3, 4]
[1, 2, ...[3, 4]];         // [1, 2, 3, 4]
```

Spread is more flexible for interleaving values; `concat` accepts multiple arrays directly.

### 1.4.4. Destructuring arrays (brief)

Pull elements into variables (File 07 goes deeper):

```js
const [first, second, ...rest] = [1, 2, 3, 4];
// first=1, second=2, rest=[3, 4]

const [, , third] = [1, 2, 3];  // skip with commas
// third=3

const [head = "default", ...tail] = [];
// head="default", tail=[]
```

Swap without temp:

```js
let a = 1, b = 2;
[a, b] = [b, a];
```

---

## 1.5. Common questions

**1.5.1. How do you check if a value is an array?**  
A: Use **`Array.isArray(value)`**. `typeof []` returns `"object"`, which is not specific enough.

**1.5.2. What is the difference between `map` and `forEach`?**  
A: **`map`** returns a **new array** with transformed values. **`forEach`** runs side effects and returns **`undefined`**. Use `map` when you need a result array.

**1.5.3. What is the difference between `filter` and `find`?**  
A: **`filter`** returns **all** matches as an array. **`find`** returns the **first** match (or `undefined`) and stops early.

**1.5.4. Does `sort()` mutate the original array?**  
A: **Yes.** It sorts in place. Copy first: `[...arr].sort(compareFn)` or use `toSorted()` (ES2023+).

**1.5.5. Why does `[10, 2, 1].sort()` give `[1, 10, 2]`?**  
A: Default sort converts elements to **strings**. Use `(a, b) => a - b` for numeric sort.

**1.5.6. What is the difference between `slice` and `splice`?**  
A: **`slice`** returns a **shallow copy** of a portion — does not mutate. **`splice`** **mutates** the array (remove/insert/replace).

**1.5.7. Is `[...arr]` a deep copy?**  
A: **No.** It is a **shallow** copy. Top-level elements are copied; nested objects/arrays are still shared. Use `structuredClone()` for deep copies of plain data.

**1.5.8. What does `reduce` need as a second argument?**  
A: An **initial accumulator** value. Always provide one for empty arrays and when the result type differs from elements (e.g. summing to `0`, building `{}`).
