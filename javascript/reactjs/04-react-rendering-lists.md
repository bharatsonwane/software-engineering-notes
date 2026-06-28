# 04. Rendering Lists & Conditionals

> File 04 — conditional UI, lists, keys, fragments.

Real UIs rarely show the same thing every time. You **conditionally** render loading spinners, errors, and empty states. You **map** arrays into lists of components. **Keys** help React track items; **fragments** group nodes without extra DOM.

Builds on state and events (File 03) and props (File 02).

**Default rule today:** use **`&&` / ternary** for simple conditionals; **`map`** for lists; **stable unique `key`** (prefer `id`, not array index when order changes); **fragments** instead of wrapper `<div>`s when layout allows.

---

## 4.1. Conditional rendering

### 4.1.1. Early return

Guard clauses keep JSX flat:

```jsx
function UserProfile({ user }) {
  if (!user) {
    return <p>No user selected.</p>;
  }

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

Loading and error states:

```jsx
function UserPanel({ loading, error, user }) {
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
  if (!user) return null;

  return <UserCard user={user} />;
}
```

### 4.1.2. Ternary operator

Inline **either/or** inside JSX:

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <p>Welcome back!</p>
      ) : (
        <p>Please sign in.</p>
      )}
    </div>
  );
}
```

Small UI toggles:

```jsx
<button>{isOn ? "ON" : "OFF"}</button>
```

Avoid deep nested ternaries — use early return or extract a component.

### 4.1.3. Logical AND (`&&`)

Render when condition is **true**; render nothing when **false**:

```jsx
function Cart({ itemCount }) {
  return (
    <div>
      <h1>Shop</h1>
      {itemCount > 0 && (
        <p>You have {itemCount} items.</p>
      )}
    </div>
  );
}
```

**Caution** — left side must be boolean. `count && <p>...</p>` fails when `count` is `0` (renders `0`):

```jsx
// Bug — shows "0" on screen when count is 0
{count && <p>{count} items</p>}

// Fix
{count > 0 && <p>{count} items</p>}
```

### 4.1.4. null and undefined

Return **`null`** from a component to render nothing:

```jsx
function AdminBanner({ isAdmin }) {
  if (!isAdmin) return null;
  return <div className="banner">Admin tools enabled</div>;
}
```

Do not confuse with **`undefined`** — prefer explicit `null` for "no UI".

### 4.1.5. Storing JSX in variables

Complex branches — assign to a variable first:

```jsx
function StatusMessage({ status }) {
  let content;

  if (status === "loading") {
    content = <Spinner />;
  } else if (status === "error") {
    content = <ErrorAlert />;
  } else {
    content = <SuccessMessage />;
  }

  return <div className="status">{content}</div>;
}
```

---

## 4.2. Rendering lists

### 4.2.1. map() to JSX

Transform array data into elements:

```jsx
const fruits = ["apple", "banana", "cherry"];

function FruitList() {
  return (
    <ul>
      {fruits.map((fruit) => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  );
}
```

Each callback returns JSX; **`map`** produces an array of elements React can render.

### 4.2.2. List item component

Extract for clarity and reuse:

```jsx
function TodoItem({ todo, onToggle }) {
  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={todo.done}
          onChange={() => onToggle(todo.id)}
        />
        {todo.text}
      </label>
    </li>
  );
}

function TodoList({ todos, onToggle }) {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
        />
      ))}
    </ul>
  );
}
```

**Key** goes on the outermost element in the `map` — often the list item component (section 4.3).

### 4.2.3. Empty lists

Always handle **zero items**:

```jsx
function TodoList({ todos }) {
  if (todos.length === 0) {
    return <p>No todos yet. Add one above.</p>;
  }

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

Or inline:

```jsx
{todos.length === 0 ? (
  <p>No todos yet.</p>
) : (
  <ul>{/* map */}</ul>
)}
```

### 4.2.4. Filter, sort, then map

Derive during render — don't mutate props/state arrays:

```jsx
function ActiveUsers({ users }) {
  const active = users
    .filter((u) => u.isActive)
    .sort((a, b) => a.name.localeCompare(b.name));

  return (
    <ul>
      {active.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

For expensive lists, memoize (File 12).

### 4.2.5. Lists of numbers and fragments

```jsx
function NumberRow({ numbers }) {
  return (
    <>
      {numbers.map((n) => (
        <span key={n} className="pill">{n}</span>
      ))}
    </>
  );
}
```

---

## 4.3. Keys

### 4.3.1. Why keys matter

React uses **keys** to match list items across re-renders — which DOM node corresponds to which data item.

Without stable keys, React may **reuse wrong DOM** → bugs in controlled inputs, animation, focus.

```jsx
{todos.map((todo) => (
  <TodoItem key={todo.id} todo={todo} />
))}
```

Keys are **not** passed as props — React consumes them. Use `id` from data if you need it inside the child.

### 4.3.2. Rules for keys

- **Unique among siblings** — not globally unique.
- **Stable** — same item → same key across renders.
- **Prefer data ids** — database id, UUID, slug.

```jsx
// Good
key={user.id}
key={product.sku}

// OK for static lists that never reorder
key={fruit}

// Avoid when list changes order or items insert/delete
key={index}
```

### 4.3.3. Index as key — when it breaks

```jsx
// Initial
["A", "B", "C"]  // keys 0, 1, 2

// After unshift "Z"
["Z", "A", "B", "C"]  // keys 0,1,2,3 — React thinks index 0 is still "A"
```

Symptoms:

- Wrong checkbox state after reorder/delete.
- Input values "stick" to wrong row.

Use **index** only if the list is **static**, never reordered, no item state in rows.

### 4.3.4. Generating keys

If no id exists, create one when adding items (File 03):

```jsx
setTodos((prev) => [
  ...prev,
  { id: crypto.randomUUID(), text: "New task", done: false },
]);
```

Do not generate new random keys **on every render**:

```jsx
// Bad — new key every render, full remount
key={Math.random()}
```

### 4.3.5. Keys in nested lists

Each **`map`** level needs keys on direct children:

```jsx
{categories.map((cat) => (
  <section key={cat.id}>
    <h2>{cat.name}</h2>
    <ul>
      {cat.items.map((item) => (
        <li key={item.id}>{item.label}</li>
      ))}
    </ul>
  </section>
))}
```

---

## 4.4. Fragments

### 4.4.1. Problem: extra wrappers

Component must return **one** root. Extra `<div>` can break layout (flex/grid, tables):

```jsx
// Extra div may break flex layout
function Row() {
  return (
    <div>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </div>
  );
}
```

### 4.4.2. Fragment syntax

**Short syntax** — no keys:

```jsx
function Row() {
  return (
    <>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </>
  );
}
```

**Explicit Fragment** — when you need a **key** in a list of fragments:

```jsx
import { Fragment } from "react";

function Glossary({ terms }) {
  return (
    <dl>
      {terms.map((term) => (
        <Fragment key={term.id}>
          <dt>{term.word}</dt>
          <dd>{term.definition}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

`<>` cannot take a `key` — use `<Fragment key={...}>`.

### 4.4.3. Fragment vs div

| Use Fragment | Use div (or semantic tag) |
|--------------|----------------------------|
| No styling/layout wrapper needed | Need className, flex child, click handler on wrapper |
| Table rows, definition lists | Containment for CSS hook |
| Avoid DOM bloat | Accessibility landmark (`<section>`, `<nav>`) |

---

## 4.5. Putting it together

### 4.5.1. Todo list pattern

Combines conditionals, map, keys, lifted state (File 03):

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState("");

  function addTodo(e) {
    e.preventDefault();
    if (!text.trim()) return;
    setTodos((prev) => [
      ...prev,
      { id: crypto.randomUUID(), text: text.trim(), done: false },
    ]);
    setText("");
  }

  function toggleTodo(id) {
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    );
  }

  const remaining = todos.filter((t) => !t.done).length;

  return (
    <div>
      <form onSubmit={addTodo}>
        <input value={text} onChange={(e) => setText(e.target.value)} />
        <button type="submit">Add</button>
      </form>

      {todos.length === 0 ? (
        <p>No todos yet.</p>
      ) : (
        <>
          <ul>
            {todos.map((todo) => (
              <li key={todo.id}>
                <label>
                  <input
                    type="checkbox"
                    checked={todo.done}
                    onChange={() => toggleTodo(todo.id)}
                  />
                  {todo.done ? <s>{todo.text}</s> : todo.text}
                </label>
              </li>
            ))}
          </ul>
          <p>{remaining} left</p>
        </>
      )}
    </div>
  );
}
```

### 4.5.2. Loading / empty / data pattern

Common async UI shape (File 10 expands):

```jsx
function UserList({ loading, error, users }) {
  if (loading) return <p>Loading users...</p>;
  if (error) return <p>Failed: {error}</p>;
  if (users.length === 0) return <p>No users found.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## 4.6. Common questions

**4.6.1. How do I conditionally render in JSX?**  
A: **Early return**, **ternary** (`cond ? a : b`), or **`&&`** (`cond && <UI />`). Pick the clearest for the case.

**4.6.2. Why does `{count && <p>…</p>}` show `0`?**  
A: When `count` is `0`, React renders **`0`** as text. Use `count > 0 &&` instead.

**4.6.3. How do I render a list in React?**  
A: Call **`.map()`** on the array inside JSX and return an element per item. Put a **`key`** on each top-level item in the list.

**4.6.4. What makes a good `key`?**  
A: A **stable, unique** id from your data (`user.id`, `todo.id`). Avoid **array index** when items reorder, insert, or delete.

**4.6.5. Can I use the index as key?**  
A: Only for **static** lists that never change order and hold no per-row state. Otherwise use real ids.

**4.6.6. What is a React Fragment?**  
A: A wrapper that **groups children without adding a DOM node**. Syntax: `<>...</>` or `<Fragment>`.

**4.6.7. When do I need `<Fragment key={...}>` instead of `<>`?**  
A: When mapping a list and the **fragment** is the direct child — fragments in lists need an explicit **key**.

**4.6.8. Do keys get passed as props to the component?**  
A: **No.** `key` is special — React uses it internally. Pass `id` separately if the child needs it.

**4.6.9. Should I filter before or inside map?**  
A: Usually **filter (or derive) before map** for clarity: `items.filter(...).map(...)`.

**4.6.10. What should I read next?**  
A: File 05 (useEffect), File 06 (forms), File 10 (fetching data into lists).
