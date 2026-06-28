# 05. useEffect & Side Effects

> File 05 — useEffect, deps, cleanup, sync with external systems.

React components should be **pure during render**: given the same props and state, they return the same UI. Anything that **touches the outside world** — fetching data, subscribing to events, updating `document.title`, starting timers — is a **side effect**. **`useEffect`** runs that work **after** React paints the screen, and lets you **sync** component state with external systems.

Builds on state and events (File 03) and rendering patterns (File 04). File 10 goes deeper on **data fetching**; File 07 covers **custom hooks** that wrap effects.

**Default rule today:** put side effects in **`useEffect`**, not in the render body; list every value from the component that the effect **reads** in the **dependency array**; return a **cleanup** function when you subscribe, listen, or create resources that must be torn down.

---

## 5.1. Side effects vs render

### 5.1.1. What belongs outside render

**Render** should compute UI from props and state only:

```jsx
function Greeting({ name }) {
  // Good — pure calculation during render
  const label = name.trim() || "Guest";
  return <h1>Hello, {label}</h1>;
}
```

**Side effects** change something outside this render pass:

| Side effect | Example |
|-------------|---------|
| Network | `fetch("/api/user")` |
| DOM API | `document.title = "Home"` |
| Subscriptions | `socket.on("message", handler)` |
| Timers | `setInterval(tick, 1000)` |
| Third-party widgets | `new Chart(canvas, options)` |
| Local storage | `localStorage.setItem("theme", theme)` |

Running these **during render** causes bugs: they may run on every render, race with each other, and break React's predictable model (File 01 — pure components).

### 5.1.2. Where useEffect runs

```jsx
import { useState, useEffect } from "react";

function Page({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Runs AFTER React commits UI to the screen
    document.title = user ? `${user.name} — App` : "App";
  }, [user]);

  return user ? <Profile user={user} /> : <p>Loading...</p>;
}
```

**Order on mount/update:**

1. Component function runs (render).
2. React updates the DOM.
3. Browser paints.
4. **`useEffect`** callbacks run.

Effects do **not** block painting — good for responsiveness. For urgent DOM work before paint, React offers **`useLayoutEffect`** (File 12 — rare, mostly measurements).

---

## 5.2. Basic useEffect syntax

### 5.2.1. Effect function

```jsx
useEffect(() => {
  // effect body — side effect code here
});
```

The function you pass to `useEffect` is the **effect**. React calls it after render.

### 5.2.2. Optional cleanup return

```jsx
useEffect(() => {
  const id = setInterval(() => console.log("tick"), 1000);

  return () => {
    clearInterval(id);  // cleanup — runs before next effect and on unmount
  };
}, []);
```

Return a **cleanup function** when you need to undo what the effect set up.

### 5.2.3. Dependency array (third argument)

```jsx
useEffect(() => {
  // ...
}, [dep1, dep2]);  // re-run when dep1 or dep2 change
```

| Dependency array | When effect runs |
|------------------|------------------|
| **Omitted** | After **every** render (almost never what you want) |
| **`[]`** | Once after **mount** (plus Strict Mode extra run in dev — see 5.7) |
| **`[a, b]`** | After mount and whenever **`a` or `b`** changes |

---

## 5.3. Dependency array in depth

### 5.3.1. Include what you read

If the effect **uses** a value from the component, include it in deps:

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);  // roomId is read inside — must be listed
}
```

When `roomId` changes, React runs **cleanup** for the old room, then runs the effect for the new room.

### 5.3.2. Stale closures

Missing deps causes **stale** values inside the effect:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count);  // always logs initial count if [count] missing
    }, 1000);
    return () => clearInterval(id);
  }, []);  // Bug — count is stale

  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

**Fix options:**

```jsx
// Option A — add count to deps (effect re-subscribes when count changes)
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, [count]);

// Option B — functional update in event handler, not in interval (often better design)
// Option C — useRef for latest value without re-running effect (File 07)
```

### 5.3.3. Stable functions and deps

Functions defined in the component are **new references** every render:

```jsx
function Search({ query }) {
  function fetchResults() {
    return fetch(`/api?q=${query}`);
  }

  useEffect(() => {
    fetchResults();
  }, [fetchResults]);  // runs every render — fetchResults identity changes
}
```

**Fixes:**

```jsx
// Move fetch inline in effect — only list query
useEffect(() => {
  fetch(`/api?q=${query}`);
}, [query]);

// Or wrap fetchResults in useCallback when shared (File 12)
```

**Setters from `useState`** and **`dispatch` from `useReducer`** are stable — safe to omit from deps (React guarantees identity).

### 5.3.4. Objects and arrays as deps

```jsx
useEffect(() => {
  saveSettings(settings);
}, [settings]);  // new object reference every render → effect runs every time
```

If `settings` is recreated each render, the effect fires too often. **Derive** stable deps (primitives) or **memoize** the object (File 12).

### 5.3.5. ESLint `react-hooks/exhaustive-deps`

The **`eslint-plugin-react-hooks`** rule warns when deps are missing or unnecessary. Treat warnings seriously — they catch stale closure bugs. Suppress only with a comment and a clear reason.

---

## 5.4. Cleanup

### 5.4.1. When cleanup runs

Cleanup runs:

1. **Before** the effect runs again (when deps change).
2. When the component **unmounts**.

```jsx
useEffect(() => {
  console.log("subscribe", roomId);
  return () => console.log("unsubscribe", roomId);
}, [roomId]);

// Mount room A:  subscribe A
// Change to B:   unsubscribe A → subscribe B
// Unmount:       unsubscribe B
```

### 5.4.2. Subscriptions and listeners

```jsx
useEffect(() => {
  function onResize() {
    setWidth(window.innerWidth);
  }

  window.addEventListener("resize", onResize);
  return () => window.removeEventListener("resize", onResize);
}, []);
```

Always remove listeners, close connections, and abort fetches in cleanup.

### 5.4.3. Aborting fetch

```jsx
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const res = await fetch(`/api/users/${id}`, {
        signal: controller.signal,
      });
      const data = await res.json();
      setUser(data);
    } catch (err) {
      if (err.name !== "AbortError") setError(err);
    }
  }

  load();
  return () => controller.abort();
}, [id]);
```

Without abort, a slow response for **old** `id` can overwrite state after **new** `id` loaded (**race condition**). File 10 expands on loading/error UI.

### 5.4.4. Timers

```jsx
useEffect(() => {
  const id = setTimeout(() => setVisible(true), 300);
  return () => clearTimeout(id);
}, []);
```

---

## 5.5. Common effect patterns

### 5.5.1. Syncing document title

```jsx
function ProductPage({ product }) {
  useEffect(() => {
    document.title = product ? product.name : "Shop";
  }, [product]);

  return <ProductDetails product={product} />;
}
```

Small, self-contained — good first useEffect.

### 5.5.2. Fetch on mount / when id changes

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function load() {
      setLoading(true);
      setError(null);
      try {
        const res = await fetch(`/api/users/${userId}`);
        if (!res.ok) throw new Error("Failed to load");
        const data = await res.json();
        if (!cancelled) setUser(data);
      } catch (e) {
        if (!cancelled) setError(e);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    load();
    return () => { cancelled = true; };
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
  if (!user) return null;

  return <Profile user={user} />;
}
```

For production apps, prefer **TanStack Query** or similar (File 10) instead of hand-rolled fetch effects.

### 5.5.3. Syncing to localStorage

```jsx
function usePersistedState(key, initial) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initial;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

**Note:** reading `localStorage` in lazy `useState` initializer is fine; **writing** on every `value` change belongs in an effect (or event handler if you only save on explicit save).

### 5.5.4. External non-React library

```jsx
useEffect(() => {
  const chart = new Chart(canvasRef.current, { /* options */ });
  return () => chart.destroy();
}, [data]);
```

Create / destroy imperatively in effect + cleanup when the library is not React-aware.

---

## 5.6. When not to use useEffect

### 5.6.1. Deriving state during render

Don't sync state with props via effect when you can **compute during render**:

```jsx
// Avoid
function FullName({ first, last }) {
  const [full, setFull] = useState("");
  useEffect(() => {
    setFull(`${first} ${last}`);
  }, [first, last]);
}

// Prefer
function FullName({ first, last }) {
  const full = `${first} ${last}`;
  return <span>{full}</span>;
}
```

Use **`useMemo`** only when the computation is expensive (File 12).

### 5.6.2. User events belong in handlers

```jsx
// Avoid — effect on every query change to POST analytics
useEffect(() => {
  trackSearch(query);
}, [query]);

// Prefer — track when user submits
function onSubmit(e) {
  e.preventDefault();
  trackSearch(query);
  runSearch(query);
}
```

Effects are for **syncing with external systems**, not for chaining **event → state → event**.

### 5.6.3. Initial data that could be server-provided

In frameworks with **server components** or **loaders** (File 09), fetch on the server or in the route loader instead of `useEffect` on mount when possible.

### 5.6.4. Resetting state when a prop changes

```jsx
// Avoid extra effect + flash
useEffect(() => {
  setSelectedItem(null);
}, [userId]);

// Prefer — key on child resets state when userId changes
<UserItems key={userId} userId={userId} />
```

Or pass `userId` and derive selection during render.

---

## 5.7. Strict Mode and development behavior

In **React Strict Mode** (dev only), React **mounts → unmounts → remounts** components to surface missing cleanup:

```jsx
useEffect(() => {
  console.log("effect");
  return () => console.log("cleanup");
}, []);
// Dev logs: effect, cleanup, effect
// Prod logs: effect
```

**Implications:**

- Effects must be **idempotent** — safe to run twice in dev.
- Cleanup must fully undo the effect — subscriptions must not leak.
- Do not rely on "run once ever" in dev without understanding Strict Mode.

---

## 5.8. Multiple effects

Split unrelated side effects into **separate** `useEffect` calls:

```jsx
function Dashboard({ userId }) {
  useEffect(() => {
    document.title = "Dashboard";
  }, []);

  useEffect(() => {
    const conn = connectAnalytics(userId);
    return () => conn.disconnect();
  }, [userId]);

  useEffect(() => {
    // fetch widgets...
  }, [userId]);

  return <div>...</div>;
}
```

Each effect has its own **deps** and **cleanup** — easier to reason about than one large effect.

---

## 5.9. Putting it together

### 5.9.1. Online status hook (preview of File 07)

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(
    typeof navigator !== "undefined" ? navigator.onLine : true
  );

  useEffect(() => {
    function goOnline() { setIsOnline(true); }
    function goOffline() { setIsOnline(false); }

    window.addEventListener("online", goOnline);
    window.addEventListener("offline", goOffline);

    return () => {
      window.removeEventListener("online", goOnline);
      window.removeEventListener("offline", goOffline);
    };
  }, []);

  return isOnline;
}

function StatusBar() {
  const online = useOnlineStatus();
  return <span>{online ? "Online" : "Offline"}</span>;
}
```

Custom hooks are just functions that call `useEffect` — File 07 formalizes the pattern.

### 5.9.2. Debounced search (effect + cleanup)

```jsx
function SearchUsers({ onResults }) {
  const [query, setQuery] = useState("");

  useEffect(() => {
    if (!query.trim()) {
      onResults([]);
      return;
    }

    const id = setTimeout(async () => {
      const res = await fetch(`/api/users?q=${encodeURIComponent(query)}`);
      const data = await res.json();
      onResults(data);
    }, 300);

    return () => clearTimeout(id);
  }, [query, onResults]);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search users"
    />
  );
}
```

Debounce cleanup cancels the pending fetch when `query` changes before 300ms.

---

## 5.10. Common questions

**5.10.1. What is a side effect in React?**  
A: Anything that touches **outside** the render result — network, DOM APIs, timers, subscriptions, storage. Run these in **`useEffect`**, not during render.

**5.10.2. When does useEffect run?**  
A: **After** React commits the update to the DOM and the browser paints. On mount and after re-renders where **dependencies** changed (if you pass a dependency array).

**5.10.3. What does an empty dependency array `[]` mean?**  
A: Run the effect **once after mount** (and run cleanup on unmount). In Strict Mode dev, mount/cleanup/mount runs once extra to test cleanup.

**5.10.4. What happens if I omit the dependency array?**  
A: The effect runs after **every** render. Rarely correct — almost always specify `[]` or `[deps]`.

**5.10.5. Why do I need a cleanup function?**  
A: To **undo** subscriptions, listeners, timers, and in-flight requests when deps change or the component unmounts. Prevents leaks and stale updates.

**5.10.6. How do I fix stale closure bugs in effects?**  
A: Add missing values to the **dependency array**, use **functional state updates**, or store latest values in **`useRef`** (File 07). Follow **`exhaustive-deps`** lint warnings.

**5.10.7. Can I fetch data in useEffect?**  
A: **Yes** for client-only loading. Use **abort/cancel** on cleanup to avoid races. For real apps, prefer **TanStack Query** or route **loaders** (Files 09–10).

**5.10.8. Should I use useEffect to update state when props change?**  
A: Usually **no** — compute derived values during **render**, reset with **`key`**, or adjust design. Effects that only mirror props into state often cause extra renders.

**5.10.9. Why does my effect run twice in development?**  
A: **Strict Mode** intentionally double-invokes effects in dev to verify cleanup. Production runs once per dependency change.

**5.10.10. What should I read next?**  
A: File 06 (forms — controlled inputs and submit), File 07 (custom hooks, useRef), File 10 (data fetching patterns).
