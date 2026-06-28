# 03. State & Events

> File 03 — useState, events, lifting state up, one-way data flow.

**State** lets components remember and update data over time. **Events** let users interact with the UI. React combines them with **one-way data flow**: data moves **down** via props; changes move **up** via **callback** props.

File 01 introduced `useState` briefly. This chapter goes deeper on **updating state correctly**, **handling events**, and **lifting state** to a common parent.

**Default rule today:** treat state as **immutable** (copy, don't mutate); use **functional updates** when the next value depends on the previous; lift state to the **lowest common ancestor** of components that need it.

---

## 3.1. useState in depth

### 3.1.1. State and re-renders

When you call a state **setter**, React:

1. Schedules a **re-render** of that component.
2. Runs the component function again with the **new** state.
3. Updates the DOM where output changed (reconciliation).

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  console.log("render", count);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

Each click logs a new render with an incremented count.

### 3.1.2. Initial state and lazy initialization

```jsx
const [count, setCount] = useState(0);
const [user, setUser] = useState({ name: "", age: 0 });
```

**Expensive initial value** — pass a **function** so it runs only once:

```jsx
function ExpensiveInit() {
  const [data, setData] = useState(() => {
    return computeHeavyInitialValue();
  });
}
```

Without the function, `computeHeavyInitialValue()` would run on **every** render.

### 3.1.3. Functional updates

When the next state depends on the **previous** state, use the updater form:

```jsx
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);  // safe when batched — both see correct prev
```

Avoid stale closures in async handlers:

```jsx
// Risky if multiple updates batch
setCount(count + 1);

// Safer
setCount((prev) => prev + 1);
```

### 3.1.4. Object and array state (immutability)

**Do not mutate** state objects or arrays — replace with a **new** copy:

```jsx
const [user, setUser] = useState({ name: "Bharat", age: 30 });

// Bad — mutates same object
// user.age = 31;
// setUser(user);

// Good — new object
setUser({ ...user, age: 31 });
```

Arrays:

```jsx
const [items, setItems] = useState(["a", "b"]);

setItems([...items, "c"]);           // add
setItems(items.filter((x) => x !== "a"));  // remove
setItems(items.map((x) => x === "b" ? "B" : x));  // update one
```

Same idea as JavaScript immutability (File 12) — React compares by reference for change detection.

### 3.1.5. Multiple state variables

Split unrelated state — easier to reason about:

```jsx
function Form() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [agreed, setAgreed] = useState(false);
}
```

Combine into one object when fields always update together (File 06 — forms):

```jsx
const [form, setForm] = useState({ name: "", email: "" });

setForm((prev) => ({ ...prev, name: "Bharat" }));
```

### 3.1.6. Batching (brief)

React **batches** multiple setters in the same event handler into **one re-render**:

```jsx
function handleClick() {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // one re-render, not two
}
```

React 18+ batches more cases (including promises/timeouts in many setups). File 05 for async interaction.

---

## 3.2. Events in React

### 3.2.1. Synthetic events

React wraps native DOM events in **SyntheticEvent** — same interface across browsers, pooled for performance (details rarely matter in app code):

```jsx
function Button() {
  function handleClick(event) {
    console.log(event.type);   // "click"
    console.log(event.target); // DOM element
  }

  return <button onClick={handleClick}>Click</button>;
}
```

Use **camelCase** event names: `onClick`, `onChange`, `onSubmit`, `onKeyDown`.

### 3.2.2. Common handlers

**Click:**

```jsx
<button onClick={() => console.log("clicked")}>Go</button>
```

**Input change:**

```jsx
function SearchBox() {
  const [query, setQuery] = useState("");

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**Form submit:**

```jsx
function LoginForm() {
  function handleSubmit(e) {
    e.preventDefault();  // don't full-page reload
    console.log("submit");
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Log in</button>
    </form>
  );
}
```

Full forms in File 06.

### 3.2.3. Passing arguments to handlers

```jsx
function ItemList({ items }) {
  function handleDelete(id) {
    console.log("delete", id);
  }

  return items.map((item) => (
    <button key={item.id} onClick={() => handleDelete(item.id)}>
      Delete {item.name}
    </button>
  ));
}
```

Wrap in arrow function — `onClick={handleDelete(item.id)}` would **call immediately** on render.

### 3.2.4. preventDefault and stopPropagation

```jsx
function Link({ href, onNavigate }) {
  function handleClick(e) {
    e.preventDefault();        // cancel default anchor navigation
    onNavigate(href);
  }

  return <a href={href} onClick={handleClick}>Go</a>;
}
```

```jsx
function handleInnerClick(e) {
  e.stopPropagation();  // don't bubble to parent handlers
}
```

---

## 3.3. One-way data flow

### 3.3.1. Data down, events up

Parent owns **state**; child receives **props** and **callbacks**:

```jsx
function Parent() {
  const [text, setText] = useState("");

  return (
    <Child
      value={text}
      onChange={setText}
    />
  );
}

function Child({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}
```

- **Down:** `value={text}`
- **Up:** `onChange` calls parent's `setText`

Child does not own the text state — parent does (**controlled** pattern, File 06).

### 3.3.2. Unidirectional flow benefits

- Easier to trace bugs — state lives in one place.
- Predictable UI — render is a function of state.
- Siblings stay in sync when they share parent state.

### 3.3.3. Controlled vs uncontrolled (preview)

| | Controlled | Uncontrolled |
|---|------------|--------------|
| Value source | React state | DOM |
| Read value | From state | Ref to DOM (File 07) |
| Typical | Forms, inputs | Simple file inputs, legacy |

Prefer **controlled** for most React forms.

---

## 3.4. Lifting state up

When **two or more components** need the same state, move it to their **closest common parent**.

### 3.4.1. Problem: siblings need shared data

```jsx
// Won't stay in sync — separate state each
function BrokenApp() {
  return (
    <>
      <TemperatureInput scale="c" />
      <TemperatureInput scale="f" />
    </>
  );
}
```

Each `TemperatureInput` would keep its own temperature.

### 3.4.2. Solution: lift to parent

```jsx
function App() {
  const [temperature, setTemperature] = useState("");
  const [scale, setScale] = useState("c");

  return (
    <>
      <TemperatureInput
        scale="c"
        temperature={scale === "c" ? temperature : toCelsius(temperature, scale)}
        onTemperatureChange={setTemperature}
      />
      <TemperatureInput
        scale="f"
        temperature={scale === "f" ? temperature : toFahrenheit(temperature, scale)}
        onTemperatureChange={setTemperature}
      />
      <BoilingVerdict celsius={parseFloat(temperature)} />
    </>
  );
}

function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  return (
    <fieldset>
      <legend>Enter temperature in {scale.toUpperCase()}:</legend>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}

function BoilingVerdict({ celsius }) {
  if (celsius >= 100) return <p>Water boils.</p>;
  return <p>Water does not boil.</p>;
}
```

Parent holds **single source of truth**; children are controlled.

### 3.4.3. Lifting callback-only state

Child triggers action; parent updates list:

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);

  function addTodo(text) {
    setTodos((prev) => [...prev, { id: Date.now(), text, done: false }]);
  }

  function toggleTodo(id) {
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    );
  }

  return (
    <>
      <TodoInput onAdd={addTodo} />
      <TodoList todos={todos} onToggle={toggleTodo} />
    </>
  );
}
```

List rendering details in File 04.

### 3.4.4. How far to lift

Lift to the **lowest** ancestor that:

- Needs the state for its own render, or
- Has both children that need read/write access.

Lift too high → prop drilling (File 02, File 08). Keep state local when only one component cares.

---

## 3.5. State design basics

### 3.5.1. Single source of truth

Avoid duplicating the same fact in two state variables:

```jsx
// Bad — name and fullName can drift
const [first, setFirst] = useState("");
const [last, setLast] = useState("");
const [fullName, setFullName] = useState("");

// Good — derive during render
const fullName = `${first} ${last}`.trim();
```

**Derived values** — compute from state/props in render; don't store unless expensive (File 12 — `useMemo`).

### 3.5.2. Minimal state

Ask: "What is the minimum state needed?" Remove anything computable:

```jsx
const [items, setItems] = useState([]);
const [filter, setFilter] = useState("");

const visibleItems = items.filter((item) =>
  item.name.toLowerCase().includes(filter.toLowerCase())
);
// visibleItems is derived — not separate useState
```

### 3.5.3. State structure

| Prefer | Avoid |
|--------|-------|
| Flat, normalized lists (`id` keys) | Deeply nested duplicate objects |
| Separate booleans for independent flags | One big `status: "loading" \| "error" \| ...` when unrelated |
| Lift shared state | Copy same data into sibling state |

Complex global state → File 08, File 11.

---

## 3.6. Common questions

**3.6.1. Why doesn't updating a variable re-render the component?**  
A: React only re-renders when **state** (via setter) or **parent** re-renders with new props. Plain `let` variables are not tracked.

**3.6.2. Can I mutate state directly?**  
A: **No.** Mutating objects/arrays in place may skip re-renders and breaks predictability. Always pass a **new** reference to the setter.

**3.6.3. When should I use the functional form of setState?**  
A: When the new state **depends on the previous** state, especially with batched updates or inside timeouts (`setCount(prev => prev + 1)`).

**3.6.4. What is lifting state up?**  
A: Moving shared state from child components to their **common parent**, then passing it down as props and updating via callbacks.

**3.6.5. What is one-way data flow?**  
A: Data flows **down** (props); events/callbacks flow **up**. Parents own state; children request changes through handlers.

**3.6.6. What is a SyntheticEvent?**  
A: React's cross-browser wrapper around native DOM events, used by `onClick`, `onChange`, etc.

**3.6.7. Why use `e.preventDefault()` on forms?**  
A: To stop the browser's default **full page reload** on submit and handle submission in JavaScript instead.

**3.6.8. What is the difference between controlled and uncontrolled components?**  
A: **Controlled** — React state is the source of truth for input value. **Uncontrolled** — DOM holds value; read via ref (File 07).

**3.6.9. Should every input have its own useState?**  
A: Often **yes** for simple forms, or **one object** for many related fields. Lift to parent when siblings or submit handler need the data.

**3.6.10. What should I read next?**  
A: File 04 (lists, keys, conditional UI), File 06 (forms), File 05 (useEffect for side effects).
