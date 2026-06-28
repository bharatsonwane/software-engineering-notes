# 08. Context & useReducer

> File 08 — useContext, useReducer, avoiding prop drilling.

When many components need the same data, passing props through every layer (**prop drilling**) gets painful. **Context** shares values across the tree without explicit props at each level. **`useReducer`** manages complex state updates with a predictable **action → state** flow — similar to Redux, built into React.

Builds on state (File 03) and hooks patterns (File 07). File 11 covers global stores (Zustand, Redux Toolkit) when Context is not enough.

**Default rule today:** use **Context** for **low-frequency, widely shared** data (theme, locale, auth user); colocate providers **as low as possible**; pair **`useReducer`** with Context for non-trivial shared state; split contexts to avoid unnecessary re-renders.

---

## 8.1. Prop drilling problem

### 8.1.1. Passing props through intermediates

```jsx
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}
```

`Layout` and `Sidebar` do not use `user` — they only forward props. Context (or a store) removes the middle passes.

### 8.1.2. When lifting state is still enough

If only **two or three** levels share state, **lifting state up** (File 03) may be simpler than Context. Reach for Context when the tree is deep or many branches need the same value.

---

## 8.2. Context API

### 8.2.1. createContext

```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext(null);

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme must be used within ThemeProvider");
  return ctx;
}
```

Default `null` (or a sentinel) lets custom hooks **fail fast** outside a provider.

### 8.2.2. Provider

```jsx
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  const value = { theme, setTheme };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Wrap the subtree that should access the context:

```jsx
function App() {
  return (
    <ThemeProvider>
      <Page />
    </ThemeProvider>
  );
}
```

### 8.2.3. Consuming with useContext

```jsx
function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      {theme}
    </button>
  );
}
```

Any descendant of `ThemeProvider` can call `useTheme()` without prop chains.

### 8.2.4. Colocate providers

Place providers **as close as needed** to consumers — not always at app root:

```jsx
function DashboardRoutes() {
  return (
    <DashboardProvider>
      <DashboardLayout />
    </DashboardProvider>
  );
}
```

Smaller subtrees = fewer components that re-render when context value changes.

---

## 8.3. Context performance

### 8.3.1. Re-renders when value changes

When **`Provider value={...}`** changes, **all consumers** of that context re-render (unless optimized — File 12).

Avoid recreating the value object every render without need:

```jsx
// New object every render — all consumers re-render
<ThemeContext.Provider value={{ theme, setTheme }}>

// Better — memoize value when dependencies stable
const value = useMemo(() => ({ theme, setTheme }), [theme]);
<ThemeContext.Provider value={value}>
```

`setTheme` from `useState` is stable — only `theme` needs to be in deps.

### 8.3.2. Split contexts

Separate **read-heavy** data from **actions**:

```jsx
const ThemeStateContext = createContext(null);
const ThemeDispatchContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeStateContext.Provider value={theme}>
      <ThemeDispatchContext.Provider value={setTheme}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  );
}
```

Components that only call `setTheme` subscribe to **dispatch** context and skip re-renders when `theme` changes (if they only use dispatch).

### 8.3.3. When Context is a poor fit

- **High-frequency updates** (mouse position, keystrokes in huge trees)
- **Large app state** with many selectors — use **Zustand / Redux** (File 11)
- **Server cache** — use **TanStack Query** (File 10)

---

## 8.4. useReducer

### 8.4.1. Reducer shape

```jsx
function counterReducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return { count: 0 };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </>
  );
}
```

**`dispatch`** is stable — safe to pass down without `useCallback`.

### 8.4.2. When to prefer useReducer over useState

| useState | useReducer |
|----------|------------|
| Few independent values | Many related fields |
| Simple updates | Next state depends on complex rules |
| Local component only | Shared update logic via Context |

### 8.4.3. Lazy initial state

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, initFunction);
// initFunction(initialArg) runs once for expensive setup
```

Same idea as lazy `useState` (File 03).

### 8.4.4. Immer (optional)

Libraries like **Immer** let you write "mutating" logic that produces immutable state — common in Redux Toolkit (File 11).

---

## 8.5. Context + useReducer together

### 8.5.1. Global state pattern

```jsx
const CartStateContext = createContext(null);
const CartDispatchContext = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case "add":
      return { items: [...state.items, action.item] };
    case "remove":
      return { items: state.items.filter((i) => i.id !== action.id) };
    default:
      return state;
  }
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    <CartStateContext.Provider value={state}>
      <CartDispatchContext.Provider value={dispatch}>
        {children}
      </CartDispatchContext.Provider>
    </CartStateContext.Provider>
  );
}

export function useCartState() {
  const ctx = useContext(CartStateContext);
  if (!ctx) throw new Error("useCartState requires CartProvider");
  return ctx;
}

export function useCartDispatch() {
  const ctx = useContext(CartDispatchContext);
  if (!ctx) throw new Error("useCartDispatch requires CartProvider");
  return ctx;
}
```

### 8.5.2. Dispatching actions from components

```jsx
function AddToCartButton({ product }) {
  const dispatch = useCartDispatch();
  return (
    <button onClick={() => dispatch({ type: "add", item: product })}>
      Add to cart
    </button>
  );
}

function CartSummary() {
  const { items } = useCartState();
  return <p>{items.length} items</p>;
}
```

---

## 8.6. Common Context use cases

### 8.6.1. Theme and locale

Theme, language, density — read by many components, changes infrequently.

### 8.6.2. Auth session

Current user and login/logout — often Context + fetch on mount (File 10). Keep **token** in httpOnly cookies when possible (security — not React-specific).

### 8.6.3. Dependency injection (testing)

Provide mock services via Context in tests without rewiring imports.

---

## 8.7. Putting it together

### 8.7.1. Todo app with Context + reducer

```jsx
function todosReducer(state, action) {
  switch (action.type) {
    case "add":
      return [...state, { id: crypto.randomUUID(), text: action.text, done: false }];
    case "toggle":
      return state.map((t) =>
        t.id === action.id ? { ...t, done: !t.done } : t
      );
    case "remove":
      return state.filter((t) => t.id !== action.id);
    default:
      return state;
  }
}

const TodosContext = createContext(null);
const TodosDispatchContext = createContext(null);

export function TodosProvider({ children }) {
  const [todos, dispatch] = useReducer(todosReducer, []);
  return (
    <TodosContext.Provider value={todos}>
      <TodosDispatchContext.Provider value={dispatch}>
        {children}
      </TodosDispatchContext.Provider>
    </TodosContext.Provider>
  );
}
```

List UI from File 04 consumes `todos` and dispatches actions — no prop drilling through layout chrome.

---

## 8.8. Common questions

**8.8.1. What is prop drilling?**  
A: Passing props through **many intermediate** components that do not use them, only forward them down.

**8.8.2. When should I use Context?**  
A: When **many components** in a subtree need the same value and prop drilling hurts. Not for every shared bit of state.

**8.8.3. Does Context replace Redux?**  
A: **Context + useReducer** covers moderate global state. **Redux / Zustand** (File 11) scale better for large apps, devtools, and middleware.

**8.8.4. Why split state and dispatch contexts?**  
A: So components that only **dispatch** do not re-render when **state** changes.

**8.8.5. useReducer vs useState?**  
A: **useState** for simple local state. **useReducer** when updates are **event-driven**, related, or complex.

**8.8.6. Is dispatch stable?**  
A: **Yes** — `dispatch` identity from `useReducer` does not change between renders.

**8.8.7. Can I put functions in context value?**  
A: **Yes**, but a **new object** `{ user, logout }` each render re-renders all consumers. **Memoize** the value object.

**8.8.8. Where should Provider live?**  
A: As **low** in the tree as still covers all consumers — avoid wrapping the entire app if only one section needs it.

**8.8.9. How do I type Context in TypeScript?**  
A: Type `createContext<T | null>(null)` and narrow in custom hooks after null check.

**8.8.10. What should I read next?**  
A: File 09 (routing), File 11 (Zustand / Redux Toolkit), File 12 (memoizing context consumers).
