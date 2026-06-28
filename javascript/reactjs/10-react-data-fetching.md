# 10. Data Fetching

> File 10 — fetch, loading/error states, TanStack Query patterns.

Most apps load **remote data**. You need predictable **loading**, **error**, and **success** UI, plus rules for **when** to fetch, **caching**, and **refetching**. This chapter covers hand-rolled **`fetch`** with React state, then **TanStack Query** (React Query) as the modern default for **server state**.

Builds on `useEffect` (File 05), lists and conditionals (File 04), and routing loaders (File 09).

**Default rule today:** treat **server state** separately from **UI state**; use **TanStack Query** (or similar) for API data in production apps; hand-roll `fetch` + `useState` only for small cases or learning.

---

## 10.1. Server state vs client state

### 10.1.1. Two kinds of state

| Client (UI) state | Server state |
|-------------------|--------------|
| Form inputs, modals open, tab index | Users, products, comments from API |
| Owned by the browser session | Owned by the server; **cached copy** on client |
| `useState`, `useReducer` | fetch, TanStack Query, SWR |

Do not mirror entire API responses into global Redux unless you need to — **server cache libraries** handle sync, deduping, and staleness.

### 10.1.2. Async UI states

Every fetch UI should handle:

```jsx
if (loading) return <Spinner />;
if (error) return <ErrorMessage error={error} />;
if (!data) return null;
return <DataView data={data} />;
```

File 04 — early return pattern.

---

## 10.2. Fetch with useState and useEffect

### 10.2.1. Basic pattern

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
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
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
  return <div>{user.name}</div>;
}
```

### 10.2.2. Race conditions

When **`userId`** changes quickly, an older request may finish last. Fixes:

- **`AbortController`** abort on cleanup (File 05)
- **`cancelled` flag** (above)
- Query library handles this automatically

### 10.2.3. Extract to custom hook

```jsx
function useUser(userId) {
  const [state, setState] = useState({ data: null, loading: true, error: null });

  useEffect(() => {
    let cancelled = false;
    (async () => {
      try {
        const res = await fetch(`/api/users/${userId}`);
        if (!res.ok) throw new Error("Failed");
        const data = await res.json();
        if (!cancelled) setState({ data, loading: false, error: null });
      } catch (error) {
        if (!cancelled) setState({ data: null, loading: false, error });
      }
    })();
    return () => { cancelled = true; };
  }, [userId]);

  return state;
}
```

Good for learning; production apps benefit from a query library.

---

## 10.3. POST, PUT, DELETE (mutations)

### 10.3.1. Imperative fetch in event handler

```jsx
async function handleSubmit(e) {
  e.preventDefault();
  setSaving(true);
  try {
    const res = await fetch("/api/posts", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form),
    });
    if (!res.ok) throw new Error("Save failed");
    const post = await res.json();
    navigate(`/posts/${post.id}`);
  } catch (err) {
    setError(err.message);
  } finally {
    setSaving(false);
  }
}
```

Mutations often live in **handlers** (File 06), not `useEffect`.

### 10.3.2. Optimistic updates

Update UI **before** server confirms; roll back on error — TanStack Query **`onMutate`** / **`useOptimistic`** (React 19) simplify this.

---

## 10.4. TanStack Query basics

### 10.4.1. Setup

```bash
npm install @tanstack/react-query
```

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Routes />
    </QueryClientProvider>
  );
}
```

### 10.4.2. useQuery

```jsx
import { useQuery } from "@tanstack/react-query";

function UserProfile({ userId }) {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ["user", userId],
    queryFn: () =>
      fetch(`/api/users/${userId}`).then((r) => {
        if (!r.ok) throw new Error("Failed");
        return r.json();
      }),
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Error: {error.message}</p>;
  return <div>{data.name}</div>;
}
```

**`queryKey`** — unique cache identity. **`queryFn`** — returns a Promise of data.

### 10.4.3. What TanStack Query gives you

| Feature | Benefit |
|---------|---------|
| **Caching** | Same key → no redundant network |
| **Deduping** | Parallel mounts share one request |
| **Background refetch** | Stale data refreshes quietly |
| **Retry** | Configurable on failure |
| **DevTools** | Inspect cache and queries |

### 10.4.4. useMutation

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function CreatePostForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newPost) =>
      fetch("/api/posts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(newPost),
      }).then((r) => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: "Hello" })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? "Saving..." : "Create"}
    </button>
  );
}
```

**`invalidateQueries`** — refetch lists after create/update.

---

## 10.5. Query keys and cache rules

### 10.5.1. Key structure

```jsx
["users"]                    // all users list
["users", userId]            // one user
["users", userId, "posts"]   // nested resource
```

Keys should include **every variable** the queryFn uses.

### 10.5.2. enabled option

Skip fetch until ready:

```jsx
useQuery({
  queryKey: ["user", userId],
  queryFn: () => fetchUser(userId),
  enabled: !!userId,
});
```

### 10.5.3. staleTime and gcTime

- **`staleTime`** — how long data is "fresh" before background refetch
- **`gcTime`** (formerly cacheTime) — how long unused cache stays in memory

Tune per resource — static config vs live dashboard differ.

---

## 10.6. React Router loaders + Query

### 10.6.1. Prefetch in loader

```jsx
// In route loader — prefetch into query cache before render
loader: async ({ params }) => {
  await queryClient.ensureQueryData({
    queryKey: ["product", params.id],
    queryFn: () => fetchProduct(params.id),
  });
  return null;
};
```

Component uses same **`queryKey`** — instant cache hit (File 09).

### 10.6.2. When to use loader vs useQuery alone

| Loader prefetch | useQuery only |
|---------------|---------------|
| Critical path data for route | Client-only apps, secondary data |
| Avoid loading spinner on navigation | Simpler setup |

Both are valid; combine for best UX on important routes.

---

## 10.7. Error and loading UX

### 10.7.1. Granular loading

- **Initial load** — full-page or skeleton
- **Background refetch** — subtle indicator, keep showing stale data
- **`isFetching` vs `isLoading`** in TanStack Query — know the difference

### 10.7.2. Error boundaries vs query error

**Query `isError`** — expected API failures, retry buttons.  
**Error boundary** — unexpected render throws (File 14).

### 10.7.3. Empty states

Distinguish **loaded empty array** from **still loading**:

```jsx
if (isLoading) return <Spinner />;
if (data.length === 0) return <p>No results.</p>;
```

---

## 10.8. Alternatives and related tools

| Tool | Notes |
|------|-------|
| **SWR** | Similar hooks API from Vercel |
| **RTK Query** | Redux Toolkit integrated queries (File 11) |
| **Apollo Client** | GraphQL-focused |
| **fetch in Server Components** | Next.js — data on server, not client effect |

Pick one **server cache** strategy per app; avoid mixing many overlapping libraries.

---

## 10.9. Putting it together

### 10.9.1. Products list + detail

```jsx
function ProductList() {
  const { data: products, isLoading, error } = useQuery({
    queryKey: ["products"],
    queryFn: fetchProducts,
  });

  if (isLoading) return <p>Loading products...</p>;
  if (error) return <p>Failed to load products.</p>;

  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>
          <Link to={`/products/${p.id}`}>{p.name}</Link>
        </li>
      ))}
    </ul>
  );
}

function ProductDetail({ id }) {
  const { data: product, isLoading, error } = useQuery({
    queryKey: ["products", id],
    queryFn: () => fetchProduct(id),
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Not found.</p>;
  return <h1>{product.name}</h1>;
}
```

---

## 10.10. Common questions

**10.10.1. Where should I fetch — useEffect or event handler?**  
A: **Load data** — `useEffect`, route **loader**, or **`useQuery`**. **Mutations** — **event handlers** / **`useMutation`**, not effect on mount.

**10.10.2. What is server state?**  
A: Data that **lives on the server** and you **cache** on the client — users, lists, permissions. Different from UI toggles and form drafts.

**10.10.3. Why TanStack Query over raw fetch?**  
A: **Caching**, deduping, retries, background refresh, and less boilerplate for loading/error.

**10.10.4. What is a query key?**  
A: A **serializable array** identifying a query in the cache — must include all variables (`userId`, filters).

**10.10.5. How do I refetch after a mutation?**  
A: **`queryClient.invalidateQueries({ queryKey: [...] })`** or update cache directly with **`setQueryData`**.

**10.10.6. How do I avoid fetch races?**  
A: **AbortController**, cancelled flag, or use a library that cancels outdated queries by key.

**10.10.7. isLoading vs isFetching?**  
A: **`isLoading`** — no cached data yet and fetching. **`isFetching`** — any in-flight request (including background).

**10.10.8. Should I store API data in useContext?**  
A: Usually **no** — use TanStack Query. Context fits **auth session**, theme, not full API cache.

**10.10.9. Loader vs useQuery?**  
A: **Loader** prefetches before route render. **useQuery** fetches when component mounts. Combine with shared **`queryKey`**.

**10.10.10. What should I read next?**  
A: File 11 (global client state), File 12 (Suspense + queries), File 09 (loaders).
