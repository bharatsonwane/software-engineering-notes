# 03. Operators & Coercion

> File 03 — operators and how JavaScript converts types.

**Operators** act on values and produce a result. **Coercion** is when JavaScript converts a value from one type to another — sometimes automatically (implicit), sometimes because you asked (explicit).

These two ideas are tightly linked: many operators trigger coercion before comparing or combining values. Knowing which operators coerce and how helps you avoid bugs like `"5" + 1` vs `"5" - 1`.

**Default rule today:** use **`===`** for equality, prefer **explicit** conversions when you need them, and learn the small list of **falsy** values by heart.

---

## 1.1. Operators

### 1.1.1. Arithmetic operators

| Operator | Name | Example | Notes |
|----------|------|---------|-------|
| `+` | Addition | `2 + 3` → `5` | Also string concatenation if either side is a string |
| `-` | Subtraction | `5 - 2` → `3` | Coerces operands to numbers |
| `*` | Multiplication | `3 * 4` → `12` | |
| `/` | Division | `10 / 4` → `2.5` | |
| `%` | Remainder | `10 % 3` → `1` | Sign follows dividend |
| `**` | Exponent | `2 ** 3` → `8` | ES2016+ |
| `++` | Increment | `i++`, `++i` | Prefix vs postfix matters for expression value |
| `--` | Decrement | `i--`, `--i` | Same as increment |

```js
"5" + 1;    // "51"  — string concat (left operand is string)
"5" - 1;    // 4     — both coerced to numbers
"5" * "2";  // 10
10 % 3;     // 1

let n = 5;
n++;        // n is 6; expression evaluates to 5 (postfix)
++n;        // n is 7; expression evaluates to 7 (prefix)
```

### 1.1.2. Assignment operators

Assign a value to a variable. Compound forms combine an operation with assignment:

```js
let x = 10;

x += 5;   // x = x + 5  → 15
x -= 3;   // 12
x *= 2;   // 24
x /= 4;   // 6
x %= 5;   // 1
x **= 3;  // 1
```

Logical assignment (ES2021+):

```js
let a = null;
a ??= 10;   // assign only if null/undefined → a is 10
a ??= 20;   // no change → still 10

let b = 0;
b ||= 5;    // assign if falsy → b is 5
b &&= 0;    // assign if truthy → b is 0
```

### 1.1.3. Comparison operators (relational)

Compare two values; return a **boolean**. Operands are coerced to numbers when comparing numbers/strings (except with `===` / `!==`).

| Operator | Meaning | Example |
|----------|---------|---------|
| `>` | Greater than | `5 > 3` → `true` |
| `<` | Less than | `5 < 3` → `false` |
| `>=` | Greater or equal | `5 >= 5` → `true` |
| `<=` | Less or equal | `3 <= 5` → `true` |

```js
"10" > 9;     // true — "10" coerced to 10
"a" > "b";    // false — lexicographic string compare
null >= 0;    // true — null coerces to 0
null == 0;    // true
null === 0;   // false
```

### 1.1.4. Logical operators

Work with **boolean** logic. Operands are coerced to boolean unless using `&&` / `||` for short-circuit **value** return.

| Operator | Name | Behavior |
|----------|------|----------|
| `&&` | AND | Returns first **falsy** value, or last value if all truthy |
| `\|\|` | OR | Returns first **truthy** value, or last value if all falsy |
| `!` | NOT | Converts to boolean and inverts |

```js
true && "hello";   // "hello"
0 && "hello";      // 0
null || "default"; // "default"
"" || 42;          // 42

!true;             // false
!"hello";          // false
!0;                // true
```

Short-circuit evaluation — right side may never run:

```js
const user = null;
const name = user && user.name;   // null — no error, user.name not accessed

false && doSomething();  // doSomething never called
true || doSomething();   // doSomething never called
```

### 1.1.5. Unary and other operators

| Operator | Example | Notes |
|----------|---------|-------|
| `+` (unary) | `+"42"` → `42` | Coerces to number |
| `-` (unary) | `-"5"` → `-5` | Coerces to number, negates |
| `typeof` | `typeof 42` → `"number"` | Returns type string |
| `void` | `void 0` → `undefined` | Evaluates expr, returns `undefined` |
| `delete` | `delete obj.key` | Removes property; returns boolean |
| `in` | `"name" in obj` | Checks if key exists (own or inherited) |
| `?.` | `user?.address?.city` | Optional chaining — stops on `null`/`undefined` |
| `??` | `value ?? "default"` | Nullish coalescing — only for `null`/`undefined` |

```js
+"42";           // 42
+"hello";        // NaN

const obj = { a: 1 };
delete obj.a;    // true
delete obj.a;    // true (already gone, still "succeeds")

"toString" in {};  // true — inherited from Object.prototype
```

### 1.1.6. Operator precedence (brief)

When multiple operators appear in one expression, **precedence** decides evaluation order. Use parentheses when in doubt.

High → low (partial list):

1. `()` grouping
2. `++` `--` (postfix), `?.` `??` `?.`
3. `!` unary `+` unary `-` `typeof`
4. `**`
5. `*` `/` `%`
6. `+` `-`
7. `<` `<=` `>` `>=`
8. `==` `!=` `===` `!==`
9. `&&`
10. `||`
11. `??` (cannot mix with `&&`/`||` without parentheses)
12. `=` `+=` `-=` …

```js
2 + 3 * 4;       // 14, not 20
(2 + 3) * 4;     // 20

1 + 2 && 3;      // 3 — + before &&
```

---

## 1.2. Equality

### 1.2.1. `==` vs `===`

| Operator | Name | Coercion? |
|----------|------|-----------|
| `===` | Strict equality | No — types must match |
| `!==` | Strict inequality | No |
| `==` | Loose equality | Yes — converts then compares |
| `!=` | Loose inequality | Yes |

**Always prefer `===` and `!==`** unless you have a specific reason for loose equality.

```js
5 === 5;       // true
5 === "5";     // false — different types

5 == "5";      // true — string coerced to number
0 == false;    // true
"" == false;   // true
null == undefined;  // true (special case)
null === undefined; // false
```

Loose equality algorithm (simplified):

1. Same type → compare with `===` rules.
2. `null == undefined` → `true` (only each other).
3. Number vs string → string to number.
4. Boolean vs anything → boolean to number (`0` or `1`).
5. Object vs primitive → object to primitive (`valueOf` / `toString`).

### 1.2.2. `Object.is`

ES6 added **`Object.is`** for cases where `===` is not enough:

```js
Object.is(NaN, NaN);   // true  (NaN === NaN is false)
Object.is(+0, -0);     // false ( +0 === -0 is true)
Object.is(5, 5);       // true — same as === for normal values
```

Use `Object.is` when you explicitly care about `NaN` equality or signed zero. For everyday checks, `===` is fine.

### 1.2.3. Special equality cases

```js
// NaN
NaN == NaN;            // false
NaN === NaN;           // false
Object.is(NaN, NaN);   // true
Number.isNaN(NaN);     // true — best way to test for NaN

// null and undefined
null == undefined;     // true
null == 0;             // false (common interview trap)
undefined == 0;        // false

// Arrays and objects — compared by reference
[] === [];             // false
{} === {};             // false
const a = [];
const b = a;
a === b;               // true — same reference

// Empty string and false
"" == false;           // true
"" === false;          // false
```

---

## 1.3. Type coercion

**Coercion** is automatic or manual conversion between types. JavaScript is **weakly typed** — it will convert values when operators or contexts require it.

### 1.3.1. Implicit vs explicit coercion

| Kind | Who triggers it | Example |
|------|-----------------|---------|
| **Implicit** | JavaScript automatically | `"5" - 1`, `if ("hello")`, `5 == "5"` |
| **Explicit** | You call a function or operator | `Number("5")`, `String(42)`, `Boolean(0)` |

```js
// Implicit
if ("0") { /* runs — non-empty string is truthy */ }
"5" + 1;   // "51"

// Explicit
Number("5");    // 5
String(42);     // "42"
Boolean("");    // false
```

Prefer **explicit** conversion in application code — it documents intent and avoids surprises.

### 1.3.2. Abstract operations (ToNumber, ToString, ToBoolean)

The spec defines internal conversion rules. Practical summary:

**ToNumber:**

| Input | Result |
|-------|--------|
| `undefined` | `NaN` |
| `null` | `0` |
| `true` / `false` | `1` / `0` |
| `""` | `0` |
| `"123"` | `123` |
| `"hello"` | `NaN` |
| Objects | Call `valueOf` / `toString`, then convert again |

**ToString:**

| Input | Result |
|-------|--------|
| `null` | `"null"` |
| `undefined` | `"undefined"` |
| `true` / `false` | `"true"` / `"false"` |
| Numbers | Decimal string (`NaN` → `"NaN"`) |
| Objects | Call `toString` (often `"[object Object]"`) |

**ToBoolean:**

| Input | Result |
|-------|--------|
| Falsy values (see 1.4.1) | `false` |
| Everything else | `true` |

```js
Number(undefined);   // NaN
Number(null);        // 0
Number(true);        // 1
Number("");          // 0
Number("  42  ");    // 42 (whitespace trimmed)

String(null);        // "null"
String(undefined);   // "undefined"

Boolean(0);          // false
Boolean("hello");    // true
```

### 1.3.3. Common coercion pitfalls

```js
// + prefers string if either operand is string
[] + [];             // "" (empty string)
[] + {};             // "[object Object]"
{} + [];             // 0 in some engines — avoid ambiguous mixes

// Comparison traps with ==
0 == false;          // true
"" == 0;             // true
"0" == false;        // true
null == 0;           // false

// typeof null quirk (from File 01)
typeof null;         // "object"

// parseInt vs Number
parseInt("42px");    // 42 — parses until invalid char
Number("42px");      // NaN — whole string must be valid
```

When reviewing code, treat **`==`** as a red flag unless comparing `null`/`undefined` with intent:

```js
// Idiomatic null/undefined check
if (value == null) { /* null OR undefined */ }

// Prefer explicit in strict codebases
if (value === null || value === undefined) { }
```

### 1.3.4. Explicit conversion functions

| Goal | Use | Avoid |
|------|-----|-------|
| To number | `Number(x)`, `parseInt(s, 10)`, `parseFloat(s)` | Unary `+` in complex expressions |
| To string | `String(x)`, template literals `` `${x}` `` | `.toString()` on `null`/`undefined` (throws) |
| To boolean | `Boolean(x)` | Double negation `!!x` is OK but less readable |

```js
Number("42");        // 42
parseInt("42px", 10); // 42
parseFloat("3.14");  // 3.14

String(42);          // "42"
String(null);        // "null"

Boolean(0);          // false
Boolean([]);         // true — empty array is truthy!
Boolean({});         // true
```

Safe string conversion:

```js
const value = null;
String(value);           // "null"
// value.toString();     // TypeError
```

---

## 1.4. Truthy & falsy

In **boolean contexts** (`if`, `while`, `&&`, `||`, `!`, ternary), values are coerced to `true` or `false`.

### 1.4.1. Falsy values (only 8)

There are exactly **8 falsy** values in JavaScript:

| Value | Notes |
|-------|-------|
| `false` | Boolean false |
| `0` | Number zero |
| `-0` | Negative zero (falsy like `0`) |
| `0n` | BigInt zero |
| `""` | Empty string |
| `null` | Intentional empty |
| `undefined` | Missing value |
| `NaN` | Not a Number |

**Everything else is truthy** — including `"0"`, `"false"`, `[]`, `{}`, `function(){}`.

```js
if ([]) {
  console.log("empty array is truthy");  // runs!
}
```

### 1.4.2. Truthy gotchas

```js
// Strings that look "false"
Boolean("false");    // true — non-empty string
Boolean("0");        // true

// Objects and arrays
Boolean([]);         // true
Boolean({});         // true

// document.all (legacy browser quirk — rare today)
// Special object that can be both truthy and falsy in very old code

// Checking for empty array/object — truthiness is not enough
const arr = [];
if (arr) {
  // true — but arr.length is 0
}
if (arr.length === 0) { /* actually empty */ }
```

### 1.4.3. `??` vs `||` vs `&&`

| Operator | Picks / returns when |
|----------|----------------------|
| `\|\|` | First **truthy** (or last if all falsy) |
| `??` | First **not null/undefined** (or last if both nullish) |
| `&&` | First **falsy** (or last if all truthy) |

```js
0 || 10;        // 10 — 0 is falsy, skipped
0 ?? 10;        // 0  — 0 is not nullish, kept

"" || "default";   // "default"
"" ?? "default";   // "" — empty string is not nullish

null ?? undefined ?? "fallback";  // "fallback"
```

Use **`??`** when `0`, `""`, or `false` are valid values you want to keep:

```js
function setCount(count) {
  const value = count ?? 0;   // only default if null/undefined
  // not: count || 0  — would turn -0 into 0 (OK) but also reject... 
  // actually 0 || 0 is 0, but if count could be explicitly passed as 0, ?? is safer
}
```

---

## 1.5. Common questions

**1.5.1. What is the difference between `==` and `===`?**  
A: `===` compares **type and value** without coercion. `==` coerces operands to the same type before comparing. Use `===` by default.

**1.5.2. Why does `"5" + 1` give `"51"` but `"5" - 1` gives `4`?**  
A: `+` does **string concatenation** if either operand is a string. `-` only works on numbers, so both operands are coerced to numbers first.

**1.5.3. What are all the falsy values in JavaScript?**  
A: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. All other values are truthy.

**1.5.4. Is an empty array `[]` truthy or falsy?**  
A: **Truthy.** Only the 8 falsy values above evaluate to `false`. To check emptiness, use `arr.length === 0`.

**1.5.5. When should I use `??` instead of `||`?**  
A: Use `??` when `0`, `""`, or `false` are valid values you must preserve. `||` treats them as missing and falls through to the default.

**1.5.6. What is the difference between `Object.is` and `===`?**  
A: `Object.is(NaN, NaN)` is `true`; `NaN === NaN` is `false`. `Object.is(+0, -0)` is `false`; `+0 === -0` is `true`. Otherwise they agree.

**1.5.7. What is type coercion?**  
A: Automatic or manual conversion between types. Implicit coercion happens in expressions like `"5" - 1` or `if (value)`. Explicit coercion uses `Number()`, `String()`, or `Boolean()`.

**1.5.8. How do I safely convert a string to a number?**  
A: Use `Number(str)` when the whole string must be numeric. Use `parseInt(str, 10)` or `parseFloat(str)` when parsing prefixes like `"42px"`. Always pass radix `10` to `parseInt`.
