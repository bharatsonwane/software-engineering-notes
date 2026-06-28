# 12. Performance & Patterns

> File 12 — memo, useMemo, useCallback, code splitting, Suspense.

React is fast by default. Optimize when **profiling** shows a problem — not preemptively everywhere. This chapter covers **`React.memo`**, **`useMemo`**, **`useCallback`**, **lazy loading**, and **Suspense** for splitting work and skipping unnecessary renders.

Builds on rendering (File 04), hooks (File 07), routing (File 09), and data fetching (File 10).

**Default rule today:** measure first; fix **wasted re-renders** and **large bundles** before micro-optimizing; use **`memo` + stable props** for expensive pure children; **lazy-load** routes and heavy components.

---

## 12.1. When to optimize

### 12.1.1. React is already efficient

Reconciliation updates only changed DOM nodes. Most slowness comes from:

- **Huge lists** without virtualization
- **Expensive calculations** every render
- **Large JavaScript bundles** slow initial load
- **Unnecessary re-renders** of heavy subtrees

### 12.1.2. Profile before memoizing

Use **React DevTools Profiler** and browser **Performance** tab. Avoid sprinkling `memo`/`useMemo` on every component — they add memory and comparison cost.

---

## 12.2. Re-renders recap

### 12.2.1. What triggers re-render?

- **State** change in this component
- **Context** value change (if consumer)
- **Parent** re-rendered (child re-renders too, unless optimized)

Props **equal by reference** for objects/functions — new reference every parent render → child thinks props changed.

### 12.2.2. Children render with parent

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <ExpensiveList items={items} />  {/* re-renders on every count click */}
    </>
  );
}
```

Fix by **lifting state down**, **memo**, or **splitting** state into smaller subtrees.

---

## 12.3. React.memo

### 12.3.1. Skip re-render if props unchanged

```jsx
const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log("ExpensiveList render");
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});
```

Shallow-compares **props** — if each prop is `===` previous, skip render.

### 12.3.2. Custom comparison

```jsx
const UserRow = memo(
  function UserRow({ user }) {
    return <div>{user.name}</div>;
  },
  (prev, next) => prev.user.id === next.user.id
);
```

Return **`true`** if props are **equal** (skip re-render).

### 12.3.3. When memo helps

- Component is **expensive** to render
- Same props **often** across parent re-renders
- Props are **stable** or primitives

Memo **does not help** if props are new objects/functions every time — pair with `useCallback` / `useMemo` or restructure.

---

## 12.4. useCallback

### 12.4.1. Stable function reference

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  const handleSave = useCallback(() => {
    save(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <MemoizedToolbar onSave={handleSave} />
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
    </>
  );
}
```

Without `useCallback`, `handleSave` is a **new function** each render → memoized child re-renders anyway.

### 12.4.2. Dependency array

Include everything **`handleSave`** closes over (`text`). Same rules as `useEffect` (File 05).

### 12.4.3. When not to use useCallback

- Child is not memoized
- Handler passed to native DOM (`onClick`) — usually fine without
- Premature optimization on cheap trees

---

## 12.5. useMemo

### 12.5.1. Cache expensive computation

```jsx
function ProductTable({ products, filter }) {
  const filtered = useMemo(() => {
    return products.filter((p) => p.name.includes(filter));
  }, [products, filter]);

  return <Table rows={filtered} />;
}
```

Recomputes only when **`products`** or **`filter`** change.

### 12.5.2. Stable object for Context or memo children

```jsx
const value = useMemo(
  () => ({ theme, toggleTheme }),
  [theme, toggleTheme]
);

return (
  <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
);
```

File 08 — avoid new context object every render.

### 12.5.3. useMemo vs useCallback

| Hook | Returns |
|------|---------|
| **`useMemo`** | **Value** (object, array, number, filtered list) |
| **`useCallback`** | **Function** (same as `useMemo(() => fn, deps)`) |

---

## 12.6. Code splitting and lazy

### 12.6.1. dynamic import()

Split heavy modules into separate **chunks** loaded on demand:

```jsx
import { lazy, Suspense } from "react";

const AdminDashboard = lazy(() => import("./AdminDashboard"));
```

Webpack/Vite creates a separate file fetched when the component first renders.

### 12.6.2. Suspense fallback

```jsx
function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/admin" element={<AdminDashboard />} />
      </Routes>
    </Suspense>
  );
}
```

While chunk loads, **`fallback`** shows (File 09 — route-level splitting).

### 12.6.3. What to lazy-load

- Admin / settings routes (rare visits)
- Rich editors, charts, maps
- Not tiny components — overhead of async boundary

---

## 12.7. Suspense and data (overview)

### 12.7.1. Suspense for lazy components

Primary use today: **waiting for JS bundle**, not yet universal for all data fetching.

### 12.7.2. Suspense with TanStack Query / React 19

Frameworks increasingly support **Suspense boundaries** for queries — `useSuspenseQuery` throws promise until data ready; nearest Suspense shows fallback.

```jsx
const { data } = useSuspenseQuery({ queryKey: ["user", id], queryFn: fetchUser });
```

File 10 — opt-in when your stack supports it.

---

## 12.8. List performance

### 12.8.1. Virtualization

Long lists (1000+ rows) — render only **visible** rows:

- **`@tanstack/react-virtual`**
- **`react-window`**

Mapping 10k `<li>` elements slows DOM and reconciliation.

### 12.8.2. Keys and stable item components

Stable **`key={item.id}`** (File 04). **`memo`** on row component when parent state changes often.

---

## 12.9. useLayoutEffect (brief)

Runs **after DOM update, before paint** — use for measuring layout or synchronous DOM writes that must happen before user sees frame. Rare; default to **`useEffect`** (File 05).

---

## 12.10. Putting it together

### 12.10.1. Optimized list row

```jsx
const ProductRow = memo(function ProductRow({ product, onSelect }) {
  return (
    <tr onClick={() => onSelect(product.id)}>
      <td>{product.name}</td>
      <td>{product.price}</td>
    </tr>
  );
});

function ProductList({ products }) {
  const [filter, setFilter] = useState("");
  const [selectedId, setSelectedId] = useState(null);

  const visible = useMemo(
    () => products.filter((p) => p.name.includes(filter)),
    [products, filter]
  );

  const handleSelect = useCallback((id) => {
    setSelectedId(id);
  }, []);

  return (
    <>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} />
      <table>
        <tbody>
          {visible.map((p) => (
            <ProductRow key={p.id} product={p} onSelect={handleSelect} />
          ))}
        </tbody>
      </table>
    </>
  );
}
```

---

## 12.11. Common questions

**12.11.1. Should I wrap every component in memo?**  
A: **No.** Only when Profiler shows **expensive** wasted renders and props can be stable.

**12.11.2. useMemo vs recalculating every render?**  
A: **`useMemo`** when computation is **measurably expensive** or you need **referential stability** for deps/memo children.

**12.11.3. Why does memo not work with inline objects?**  
A: **`{ foo: 1 }`** is a **new reference** each render — shallow compare fails. Memoize the object or pass primitives.

**12.11.4. useCallback vs defining function in render?**  
A: Inline is fine unless a **memoized child** or **effect dep** needs stable identity.

**12.11.5. What is code splitting?**  
A: Loading parts of the app **on demand** via dynamic `import()` — smaller initial bundle.

**12.11.6. What does Suspense do?**  
A: Catches **lazy components** (and some async data patterns) while waiting — shows **fallback** UI.

**12.11.7. How do I speed up a 10,000 item list?**  
A: **Virtualize** — do not render all DOM nodes. Consider pagination.

**12.11.8. Does Context hurt performance?**  
A: **Any consumer** re-renders when value changes. Split contexts, memoize value, or use **Zustand selectors** (File 11).

**12.11.9. React Compiler / automatic memoization?**  
A: Emerging tooling may reduce manual memo — still understand concepts for debugging.

**12.11.10. What should I read next?**  
A: File 13 (testing memoized components), File 14 (architecture boundaries), File 10 (query suspense).
