# 07. Hooks Patterns

> File 07 — rules of hooks, custom hooks, useRef, useId.

**Hooks** are functions that let function components use state, effects, refs, and more. This chapter covers **rules** you must follow, **custom hooks** for reusable logic, **`useRef`** for values and DOM access, and **`useId`** for stable ids.

Builds on `useState` (File 03), `useEffect` (File 05), and forms (File 06). File 08 adds **Context** and **useReducer**.

**Default rule today:** only call hooks at the **top level** of React functions; prefix reusable hook logic with **`use`**; use **`useRef`** for mutable values that should not trigger re-renders; use **`useId`** for accessibility ids in SSR-safe apps.

---

## 7.1. Rules of Hooks

### 7.1.1. Only call hooks at the top level

Do **not** call hooks inside loops, conditions, or nested functions:

```jsx
// Wrong
function Bad({ show }) {
  if (show) {
    const [x, setX] = useState(0);  // breaks rules
  }
}

// Right
function Good({ show }) {
  const [x, setX] = useState(0);
  if (!show) return null;
  return <div>{x}</div>;
}
```

React relies on **call order** being identical every render to match state to the right hook.

### 7.1.2. Only call hooks from React functions

Call hooks from:

- **Function components**
- **Custom hooks** (functions whose names start with `use`)

Do **not** call hooks from regular JS functions, class components, or event handlers.

### 7.1.3. ESLint enforcement

**`eslint-plugin-react-hooks`** with rules **`rules-of-hooks`** and **`exhaustive-deps`** catches most mistakes. Enable them in every React project.

---

## 7.2. Custom hooks

### 7.2.1. What is a custom hook?

A **function** that **calls other hooks** and returns state, handlers, or refs for components to use:

```jsx
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);
  const reset = () => setCount(initial);
  return { count, increment, decrement, reset };
}

function App() {
  const { count, increment } = useCounter(0);
  return <button onClick={increment}>{count}</button>;
}
```

Each component calling `useCounter` gets **its own** independent state.

### 7.2.2. Naming convention

Name must start with **`use`** so React (and linters) know it may call hooks:

```jsx
function useLocalStorage(key, initial) { ... }
function useMediaQuery(query) { ... }
```

### 7.2.3. Extract when logic repeats

Extract a custom hook when:

- Same **state + effect + handlers** appear in multiple components
- A component grows too large and one concern can be isolated
- You want to **test** logic separately from UI

Keep hooks **focused** — one responsibility per hook when possible.

### 7.2.4. Custom hook wrapping useEffect

From File 05 — online status:

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const on = () => setIsOnline(true);
    const off = () => setIsOnline(false);
    window.addEventListener("online", on);
    window.addEventListener("offline", off);
    return () => {
      window.removeEventListener("online", on);
      window.removeEventListener("offline", off);
    };
  }, []);

  return isOnline;
}
```

### 7.2.5. Returning stable APIs

Return an **object** or **tuple** — document whether functions are stable. For callbacks passed to effects, wrap in **`useCallback`** when needed (File 12).

---

## 7.3. useRef

### 7.3.1. Ref object persists across renders

```jsx
const ref = useRef(initialValue);
// ref.current — read/write without re-render
```

Updating **`ref.current`** does **not** trigger a re-render (unlike `setState`).

### 7.3.2. DOM refs

Attach to JSX to access the underlying DOM node:

```jsx
function FocusInput() {
  const inputRef = useRef(null);

  function focus() {
    inputRef.current?.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button type="button" onClick={focus}>Focus</button>
    </>
  );
}
```

Use for: focus, scroll, measuring size, integrating non-React libraries.

### 7.3.3. Mutable instance values

Store timers, previous props, or any value that should survive renders without causing them:

```jsx
function Timer() {
  const intervalRef = useRef(null);

  useEffect(() => {
    intervalRef.current = setInterval(tick, 1000);
    return () => clearInterval(intervalRef.current);
  }, []);
}
```

### 7.3.4. Storing previous value

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

### 7.3.5. Latest callback without re-subscribing

```jsx
function Chat({ onMessage }) {
  const onMessageRef = useRef(onMessage);
  onMessageRef.current = onMessage;

  useEffect(() => {
    const sub = subscribe((msg) => onMessageRef.current(msg));
    return () => sub.unsubscribe();
  }, []);  // empty deps — always calls latest onMessage
}
```

Pattern: keep ref synced each render; effect reads `ref.current`.

### 7.3.6. ref callback (advanced)

```jsx
<div ref={(node) => { /* node mounted or unmounted */ }} />
```

Use when you need setup/teardown per DOM node. Less common than object ref.

---

## 7.4. useId

### 7.4.1. Stable unique ids for accessibility

Link labels, inputs, and error messages with matching **`id`** / **`htmlFor`**:

```jsx
function EmailField({ label, error }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        aria-describedby={error ? errorId : undefined}
        aria-invalid={!!error}
      />
      {error && <p id={errorId} role="alert">{error}</p>}
    </div>
  );
}
```

### 7.4.2. SSR and hydration

**`useId`** generates ids that are **consistent** between server and client render — avoid `Math.random()` or increment counters for ids in SSR apps.

### 7.4.3. Do not use useId for list keys

**Keys** in lists need stable **data ids** (`item.id`), not `useId()` — `useId` is per component instance, not per list item logic.

---

## 7.5. Composing hooks

### 7.5.1. Hooks can call hooks

```jsx
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then((data) => { if (!cancelled) setUser(data); })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [userId]);

  return { user, loading };
}
```

File 10 covers server-state libraries that replace hand-rolled fetch hooks.

### 7.5.2. Split hooks by concern

```jsx
function ProfilePage({ userId }) {
  const { user, loading } = useUser(userId);
  const isOnline = useOnlineStatus();
  // render...
}
```

Smaller hooks compose into readable components.

---

## 7.6. Hook pitfalls

### 7.6.1. Over-abstracting too early

Not every `useState` needs a custom hook. Extract when **reuse** or **clarity** justifies it.

### 7.6.2. Hidden dependencies

Custom hooks should document required **props/args** and return values. Callers still need correct **effect deps** inside the hook.

### 7.6.3. Ref vs state

| Need | Use |
|------|-----|
| Value shown in UI | **state** |
| DOM node, timer id, previous value, latest callback | **ref** |

---

## 7.7. Putting it together

### 7.7.1. useDebouncedValue

```jsx
function useDebouncedValue(value, delayMs) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delayMs);
    return () => clearTimeout(id);
  }, [value, delayMs]);

  return debounced;
}

function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebouncedValue(query, 300);

  useEffect(() => {
    if (debouncedQuery) search(debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

---

## 7.8. Common questions

**7.8.1. What are the Rules of Hooks?**  
A: Call hooks only at the **top level** of React functions, and only from **components** or **custom hooks** — never inside conditions, loops, or plain functions.

**7.8.2. What is a custom hook?**  
A: A reusable function named **`useSomething`** that calls hooks and returns state, effects, or handlers for components.

**7.8.3. Does each component share custom hook state?**  
A: **No.** Each call site gets **independent** state, like `useState`.

**7.8.4. When do I use useRef vs useState?**  
A: **`useState`** when UI should update. **`useRef`** for DOM access or mutable values that should **not** re-render on change.

**7.8.5. What is useId for?**  
A: Generate **stable unique ids** for `label`/`input`/`aria-*` links, especially with **SSR**.

**7.8.6. Can I use useId as a list key?**  
A: **No.** Use stable ids from **data** (`item.id`).

**7.8.7. Why store callbacks in refs?**  
A: So **effects** with empty deps can call the **latest** callback without re-subscribing every render.

**7.8.8. Can custom hooks return JSX?**  
A: They **can**, but usually return **data and functions** — keep UI in components. Exception: small shared layout hooks are rare.

**7.8.9. How do I test custom hooks?**  
A: **`@testing-library/react`** **`renderHook`** (File 13) — render hook in isolation and assert return values.

**7.8.10. What should I read next?**  
A: File 08 (Context, useReducer), File 12 (useCallback/useMemo with hooks), File 13 (testing hooks).
