# 11. State Management

> File 11 — local vs global state, Zustand / Redux Toolkit basics.

React apps juggle **local UI state**, **shared client state**, and **server state** (File 10). This chapter clarifies **what goes where**, when **Context + useReducer** (File 08) is enough, and when to adopt **Zustand** or **Redux Toolkit** for global client state.

**Default rule today:** keep state **local** until sharing hurts; use **TanStack Query** for server data; use **Zustand** for moderate global client state or **Redux Toolkit** when you need strict patterns, middleware, and large-team tooling.

---

## 11.1. State placement decision tree

### 11.1.1. Where should this state live?

```
Is it from the API?
  → Yes → TanStack Query / RTK Query (File 10)

Do only one component and its children need it?
  → Yes → useState / useReducer locally (File 03)

Do siblings need it?
  → Lift to common parent (File 03)

Do many distant components need it?
  → Context (File 08) OR global store (this chapter)

Does it change very often across a huge tree?
  → Prefer Zustand/Redux with selectors, not one big Context
```

### 11.1.2. Avoid duplicating server data in global store

```jsx
// Avoid — user also lives in React Query cache
dispatch(setUser(fetchedUser));

// Prefer — single source for server copy
const { data: user } = useQuery({ queryKey: ["me"], queryFn: fetchMe });
```

Store **UI preferences**, **draft wizard state**, **cart** (if not server-owned) globally — not every GET response.

---

## 11.2. Context limits (recap)

File 08 — Context works for theme, auth snapshot, locale. Pain points at scale:

- One **Provider value** change re-renders many consumers
- No built-in **middleware**, **devtools**, or **time-travel**
- Easy to create a **god context** that everything subscribes to

Graduate to a store when profiling shows Context churn or logic sprawls across files.

---

## 11.3. Zustand

### 11.3.1. Minimal global store

```bash
npm install zustand
```

```jsx
import { create } from "zustand";

const useCartStore = create((set) => ({
  items: [],
  addItem: (item) =>
    set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter((i) => i.id !== id),
    })),
  clear: () => set({ items: [] }),
}));

function CartBadge() {
  const count = useCartStore((s) => s.items.length);
  return <span>{count}</span>;
}

function AddButton({ product }) {
  const addItem = useCartStore((s) => s.addItem);
  return <button onClick={() => addItem(product)}>Add</button>;
}
```

### 11.3.2. Selectors prevent extra re-renders

Subscribe to **slices** of state:

```jsx
const items = useCartStore((s) => s.items);        // re-render when items change
const addItem = useCartStore((s) => s.addItem);    // stable function reference
```

Only re-renders when **selected value** changes (shallow compare by default).

### 11.3.3. When Zustand fits

- Medium global state without boilerplate
- Quick prototypes and production apps alike
- Optional **persist** middleware (localStorage)
- Works outside React — `useCartStore.getState()`

### 11.3.4. immer middleware (optional)

```jsx
import { immer } from "zustand/middleware/immer";

const useStore = create(
  immer((set) => ({
    todos: [],
    toggle: (id) =>
      set((state) => {
        const todo = state.todos.find((t) => t.id === id);
        if (todo) todo.done = !todo.done;
      }),
  }))
);
```

---

## 11.4. Redux Toolkit

### 11.4.1. Core ideas

| Term | Meaning |
|------|---------|
| **Store** | Single app state tree |
| **Slice** | One feature's reducer + actions |
| **Dispatch** | Send action to update state |
| **Selector** | Read derived data from state |

Redux enforces **one direction**: UI → dispatch(action) → reducer → new state → UI.

### 11.4.2. createSlice

```jsx
import { configureStore, createSlice } from "@reduxjs/toolkit";

const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [] },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload); // Immer inside RTK
    },
    removeItem(state, action) {
      state.items = state.items.filter((i) => i.id !== action.payload);
    },
  },
});

export const { addItem, removeItem } = cartSlice.actions;

const store = configureStore({
  reducer: { cart: cartSlice.reducer },
});
```

### 11.4.3. React bindings

```jsx
import { Provider, useSelector, useDispatch } from "react-redux";

function App() {
  return (
    <Provider store={store}>
      <CartPage />
    </Provider>
  );
}

function CartPage() {
  const items = useSelector((state) => state.cart.items);
  const dispatch = useDispatch();
  return (
    <button onClick={() => dispatch(addItem({ id: 1, name: "Hat" }))}>
      Add ({items.length})
    </button>
  );
}
```

### 11.4.4. RTK Query (server state)

Redux Toolkit includes **RTK Query** — define API slices with generated hooks (parallel to TanStack Query). Pick **one** primary server cache per app.

### 11.4.5. When Redux Toolkit fits

- Large teams wanting **strict conventions**
- Complex async middleware (logging, analytics)
- **Redux DevTools** time-travel debugging
- Existing Redux ecosystem investment

---

## 11.5. Zustand vs Redux Toolkit

| | Zustand | Redux Toolkit |
|---|---------|---------------|
| Boilerplate | Low | Medium |
| Learning curve | Gentle | Steeper |
| DevTools | Optional plugin | First-class |
| Selectors | Per-hook selector | `useSelector` + memoized selectors |
| Middleware | Minimal | Rich (thunks, listeners) |
| Best for | Most apps' client global state | Large apps, strict architecture |

Neither replaces **TanStack Query** for normalized server cache unless you standardize on RTK Query.

---

## 11.6. Patterns across libraries

### 11.6.1. Colocate feature state

Keep cart logic in `cartSlice` or `useCartStore` — not scattered `useState` in ten files.

### 11.6.2. Derived state

Compute totals in **selectors** or `useMemo`, do not duplicate in store unless needed for perf:

```jsx
const total = useCartStore((s) =>
  s.items.reduce((sum, i) => sum + i.price, 0)
);
```

### 11.6.3. URL as state

Filters, pagination, selected tab — prefer **URL search params** (File 09) over global store when users should share/bookmark state.

---

## 11.7. Putting it together

### 11.7.1. Small app stack

| Concern | Tool |
|---------|------|
| Form inputs | `useState` (File 06) |
| Theme / auth user | Context (File 08) |
| API lists and details | TanStack Query (File 10) |
| Shopping cart UI | Zustand store |
| Routes | React Router (File 09) |

Avoid Redux until complexity or team standards require it.

---

## 11.8. Common questions

**11.8.1. Do I need Redux for every React app?**  
A: **No.** Most apps get by with **local state**, **Context**, **TanStack Query**, and optionally **Zustand**.

**11.8.2. Zustand vs Context?**  
A: **Context** for low-frequency shared values. **Zustand** when many components need **selective** subscriptions without prop drilling or giant re-renders.

**11.8.3. Should API data live in Redux?**  
A: Prefer **TanStack Query** or **RTK Query**. Duplicating server cache in manual Redux slices adds sync bugs.

**11.8.4. What is a selector?**  
A: Function that **reads** a slice of store state — enables fine-grained subscriptions.

**11.8.5. Can I use multiple Zustand stores?**  
A: **Yes** — one store per feature domain is common (`useCartStore`, `useUIStore`).

**11.8.6. Is dispatch stable in Redux?**  
A: **Yes** — like `useReducer` dispatch (File 08).

**11.8.7. How do I persist cart to localStorage?**  
A: Zustand **`persist`** middleware, or save in effect (File 05) — handle SSR carefully.

**11.8.8. When is Redux Toolkit worth the overhead?**  
A: Large codebases, many async workflows, team already on Redux, need **standardized** patterns and DevTools.

**11.8.9. Can Zustand and Redux coexist?**  
A: Technically yes, but **avoid** — pick one global client state approach.

**11.8.10. What should I read next?**  
A: File 12 (performance — selectors vs re-renders), File 10 (server cache), File 14 (feature folders).
