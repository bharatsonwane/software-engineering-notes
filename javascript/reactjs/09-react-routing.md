# 09. Routing

> File 09 — React Router, nested routes, URL params, loaders.

Single-page apps (SPAs) still need **multiple screens**. **React Router** maps **URLs** to **components**, supports **nested layouts**, **dynamic segments**, and (in v6.4+) **loaders** for data before render.

Builds on components and composition (File 02). File 10 covers fetching inside routes; File 14 covers app structure around routes.

**Default rule today:** use **React Router v6+** with **`createBrowserRouter`** / **`<RouterProvider>`** or **`<Routes>` + `<Route>`**; prefer **route-level data loading** (loaders) over fetch-on-mount in every page when using the data APIs.

---

## 9.1. Why client-side routing

### 9.1.1. URL as UI state

The **address bar** should reflect where the user is:

| URL | Screen |
|-----|--------|
| `/` | Home |
| `/products` | Product list |
| `/products/42` | Product detail |
| `/settings` | Settings |

Users can bookmark, share links, and use back/forward. React Router syncs URL ↔ rendered components **without** full page reload.

### 9.1.2. React Router role

- **Match** path to a route definition
- **Render** the matching component(s)
- Provide **navigation** (`Link`, `useNavigate`)
- Expose **URL params** and **query strings**

Install: `npm install react-router-dom`

---

## 9.2. Basic setup

### 9.2.1. Browser router with Routes

```jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**`<Link to="...">`** — client navigation (no full reload).  
**`path="*"`** — catch-all for 404.

### 9.2.2. createBrowserRouter (recommended for data APIs)

```jsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  { path: "/", element: <Home /> },
  { path: "/about", element: <About /> },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

Data loaders and actions integrate cleanly with this style (9.5).

---

## 9.3. Navigation

### 9.3.1. Link and NavLink

```jsx
<Link to="/products">Products</Link>

<NavLink
  to="/products"
  className={({ isActive }) => (isActive ? "active" : "")}
>
  Products
</NavLink>
```

**`NavLink`** knows if its route is **active** — ideal for nav bars.

### 9.3.2. useNavigate (programmatic)

```jsx
import { useNavigate } from "react-router-dom";

function LoginForm() {
  const navigate = useNavigate();

  async function onSubmit(data) {
    await login(data);
    navigate("/dashboard", { replace: true });
  }
}
```

**`replace: true`** — no extra history entry (like redirect after login).

### 9.3.3. Relative links

Inside nested routes, **`to="details"`** resolves relative to the current route path.

---

## 9.4. Dynamic routes and params

### 9.4.1. URL parameters

```jsx
<Route path="/products/:productId" element={<ProductDetail />} />

function ProductDetail() {
  const { productId } = useParams();
  return <p>Product {productId}</p>;
}
```

**`:productId`** is a **dynamic segment**. `useParams()` returns `{ productId: "42" }` for `/products/42`.

### 9.4.2. Optional and splat segments

```jsx
<Route path="/files/*" element={<Files />} />
// useParams().["*"] captures rest of path
```

Consult React Router docs for optional params (`:id?`) in your version.

### 9.4.3. Search params (query string)

```jsx
import { useSearchParams } from "react-router-dom";

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const q = searchParams.get("q") ?? "";

  return (
    <input
      value={q}
      onChange={(e) => setSearchParams({ q: e.target.value })}
    />
  );
}
```

`/products?q=shirt` — **`useSearchParams`** reads and updates **`?q=`** without losing other params if you merge carefully.

---

## 9.5. Nested routes and layouts

### 9.5.1. Layout route with Outlet

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />
  <Route path="settings" element={<Settings />} />
  <Route path="profile" element={<Profile />} />
</Route>
```

```jsx
import { Outlet } from "react-router-dom";

function DashboardLayout() {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>
        <Outlet />  {/* child route renders here */}
      </main>
    </div>
  );
}
```

URLs:

| Path | Renders |
|------|---------|
| `/dashboard` | Layout + DashboardHome |
| `/dashboard/settings` | Layout + Settings |

**`index`** — default child when path matches parent exactly.

### 9.5.2. Shared chrome, independent pages

Nested routes avoid duplicating sidebar/header in every page component (File 02 — composition).

---

## 9.6. Loaders and actions (data router)

### 9.6.1. Loader — fetch before render

```jsx
const router = createBrowserRouter([
  {
    path: "/products/:productId",
    element: <ProductDetail />,
    loader: async ({ params }) => {
      const res = await fetch(`/api/products/${params.productId}`);
      if (!res.ok) throw new Response("Not Found", { status: 404 });
      return res.json();
    },
  },
]);
```

```jsx
import { useLoaderData } from "react-router-dom";

function ProductDetail() {
  const product = useLoaderData();
  return <h1>{product.name}</h1>;
}
```

Data is available on first render of the route — reduces loading flash vs `useEffect` fetch (File 05, File 10).

### 9.6.2. Error boundaries on routes

```jsx
{
  path: "/products/:productId",
  element: <ProductDetail />,
  loader: productLoader,
  errorElement: <ProductError />,
}
```

Loader throws → **`errorElement`** renders (File 14 — error boundaries).

### 9.6.3. Actions — form mutations

```jsx
{
  path: "/products/new",
  element: <NewProduct />,
  action: async ({ request }) => {
    const formData = await request.formData();
    await createProduct(Object.fromEntries(formData));
    return redirect("/products");
  },
}
```

Use **`<Form>`** from React Router to POST to the route **action** (File 06 — forms).

---

## 9.7. Protected routes

### 9.7.1. Wrapper component

```jsx
function ProtectedRoute({ children }) {
  const { user } = useAuth(); // from Context — File 08
  const location = useLocation();

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

<Route
  path="/dashboard/*"
  element={
    <ProtectedRoute>
      <DashboardLayout />
    </ProtectedRoute>
  }
/>
```

After login, **`navigate(location.state?.from ?? "/")`** sends user back.

### 9.7.2. Loader auth check

In data routers, throw **`redirect("/login")`** from loader if session missing — centralizes gate before render.

---

## 9.8. Code splitting routes (preview)

```jsx
import { lazy, Suspense } from "react";

const Settings = lazy(() => import("./Settings"));

<Route
  path="/settings"
  element={
    <Suspense fallback={<p>Loading...</p>}>
      <Settings />
    </Suspense>
  }
/>
```

File 12 expands **lazy** and **Suspense**.

---

## 9.9. Putting it together

### 9.9.1. Small app route table

```jsx
const router = createBrowserRouter([
  {
    element: <RootLayout />,
    children: [
      { index: true, element: <Home /> },
      { path: "products", element: <ProductList /> },
      {
        path: "products/:id",
        element: <ProductDetail />,
        loader: productLoader,
        errorElement: <RouteError />,
      },
      {
        path: "account",
        element: <ProtectedRoute><Account /></ProtectedRoute>,
      },
    ],
  },
  { path: "login", element: <Login /> },
  { path: "*", element: <NotFound /> },
]);
```

---

## 9.10. Common questions

**9.10.1. Link vs anchor `<a href>`?**  
A: **`<Link>`** navigates inside the SPA without reload. Use **`<a href>`** for external URLs or downloads.

**9.10.2. How do I read URL params?**  
A: **`useParams()`** for path segments (`:id`). **`useSearchParams()`** for query string.

**9.10.3. What is Outlet?**  
A: Placeholder in a **layout route** where **child routes** render.

**9.10.4. Loader vs useEffect fetch?**  
A: **Loader** runs as part of routing — data ready before route component paints. **useEffect** is fine for client-only or non-router apps (File 10).

**9.10.5. How do nested routes build full paths?**  
A: Child paths **append** to parent unless they start with `/` (absolute).

**9.10.6. How do I redirect?**  
A: **`<Navigate to="..." />`**, **`navigate()`**, or **`redirect()`** in loader/action.

**9.10.7. Where does BrowserRouter go?**  
A: Near app root (or use **`RouterProvider`** with **`createBrowserRouter`**). Only one router per app.

**9.10.8. How do I pass state between routes?**  
A: **`navigate("/path", { state: { ... } })`** and **`useLocation().state`** — not in URL; lost on refresh.

**9.10.9. React Router vs Next.js routing?**  
A: **React Router** — client SPA in Vite/CRA. **Next.js** — file-based routing with server components. Concepts overlap; APIs differ.

**9.10.10. What should I read next?**  
A: File 10 (data fetching, TanStack Query), File 12 (lazy routes), File 14 (app folder structure).
