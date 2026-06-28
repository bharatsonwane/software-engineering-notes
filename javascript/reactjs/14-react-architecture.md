# 14. Architecture

> File 14 — folder structure, compound components, error boundaries.

As apps grow, **structure** matters as much as syntax. This chapter covers **feature folders**, **separation of concerns**, **compound components**, **error boundaries**, and patterns that keep React codebases maintainable.

Builds on composition (File 02), Context (File 08), routing (File 09), and testing (File 13).

**Default rule today:** organize by **feature**, not file type; keep **UI**, **hooks**, and **API** colocated per feature; use **error boundaries** for unexpected failures; prefer **composition** over prop explosion.

---

## 14.1. Folder structure

### 14.1.1. Feature-first layout

```
src/
  features/
    auth/
      components/
        LoginForm.jsx
        AuthGuard.jsx
      hooks/
        useAuth.js
      api/
        authApi.js
      index.js          # public exports
    products/
      components/
        ProductList.jsx
        ProductCard.jsx
      hooks/
        useProducts.js
      routes/
        ProductRoutes.jsx
  components/           # shared UI — Button, Modal, Layout
  lib/                  # utils, fetch client, constants
  app/
    App.jsx
    router.jsx
    providers.jsx
```

**Feature folders** group everything for one domain. **`components/`** at root holds **design system** pieces used everywhere.

### 14.1.2. Avoid type-only grouping at scale

```
# Hard to navigate in large apps
components/
hooks/
services/
```

Cross-cutting folders force jumping between directories for one feature. Hybrid works: shared **`components/`**, feature-local **`hooks/`**.

### 14.1.3. Public API via index.js

```jsx
// features/cart/index.js
export { CartProvider, useCart } from "./CartContext";
export { CartDrawer } from "./components/CartDrawer";
```

Import from **`@/features/cart`**, not deep paths into internals — freedom to refactor inside feature.

### 14.1.4. Colocate tests and styles

```
ProductCard/
  ProductCard.jsx
  ProductCard.test.jsx
  ProductCard.module.css
```

Tests next to source (File 13) — easier to find and maintain.

---

## 14.2. Separation of concerns

### 14.2.1. Presentational vs container (light touch)

| Layer | Responsibility |
|-------|----------------|
| **Page / container** | Data hooks, route params, wire handlers |
| **Presentational** | Props in, JSX out — minimal logic |
| **Hooks** | Reusable state + effects (File 07) |
| **API module** | fetch functions, no React |

Not a strict rule — avoid "container" files that only pass props through one level.

### 14.2.2. Example split

```jsx
// features/products/hooks/useProduct.js
export function useProduct(id) {
  return useQuery({ queryKey: ["product", id], queryFn: () => fetchProduct(id) });
}

// features/products/components/ProductPage.jsx
export function ProductPage() {
  const { id } = useParams();
  const { data, isLoading, error } = useProduct(id);
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <ProductDetails product={data} />;
}

// features/products/components/ProductDetails.jsx
export function ProductDetails({ product }) {
  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </article>
  );
}
```

File 10 — data in hook; UI stays readable.

### 14.2.3. providers.jsx

Centralize app-wide wrappers (File 08, File 10, File 09):

```jsx
export function AppProviders({ children }) {
  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        <AuthProvider>
          {children}
        </AuthProvider>
      </ThemeProvider>
    </QueryClientProvider>
  );
}
```

---

## 14.3. Compound components

### 14.3.1. Flexible API without prop soup

Instead of one mega-component with dozens of props:

```jsx
// Avoid
<Tabs tabs={items} activeIndex={0} onTabChange={...} renderPanel={...} />
```

Use **composition** — parent shares state with children via Context **inside the feature**:

```jsx
<Tabs defaultIndex={0}>
  <Tabs.List>
    <Tabs.Tab index={0}>Overview</Tabs.Tab>
    <Tabs.Tab index={1}>Reviews</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel index={0}><Overview /></Tabs.Panel>
    <Tabs.Panel index={1}><Reviews /></Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

### 14.3.2. Internal context for compound pattern

```jsx
const TabsContext = createContext(null);

function Tabs({ defaultIndex = 0, children }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  const value = useMemo(
    () => ({ activeIndex, setActiveIndex }),
    [activeIndex]
  );
  return (
    <TabsContext.Provider value={value}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  return (
    <button
      role="tab"
      aria-selected={activeIndex === index}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

Tabs.Tab = Tab;
Tabs.List = TabList;
// attach subcomponents
```

Consumers compose markup; library owns **behavior** and **accessibility** wiring.

### 14.3.3. When to use

- Design systems (`Select`, `Menu`, `Accordion`)
- Repeated layout patterns with slots
- File 02 — `children` and composition extended

---

## 14.4. Error boundaries

### 14.4.1. What they catch

**Error boundaries** catch **render errors** in child tree — JavaScript exceptions during render, lifecycle, or constructors of child components.

They do **not** catch:

- Event handler errors (use try/catch)
- Async errors inside `useEffect` (handle in promise)
- Server-side rendering errors (framework-specific)

### 14.4.2. Class component boundary (current API)

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    console.error("Boundary caught:", error, info.componentStack);
    // report to Sentry, etc.
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div role="alert">
          <h2>Something went wrong.</h2>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

React may add **`use` / error boundary hooks** — check your React version; class boundary remains widely used.

### 14.4.3. Granular placement

```jsx
<AppProviders>
  <ErrorBoundary fallback={<AppCrashScreen />}>
    <Header />
    <ErrorBoundary fallback={<SectionError />}>
      <ProductRoutes />
    </ErrorBoundary>
    <Footer />
  </ErrorBoundary>
</AppProviders>
```

Isolate failures — one broken widget should not white-screen the entire app.

### 14.4.4. React Router errorElement

File 09 — route-level **`errorElement`** for loader/render failures in a section.

---

## 14.5. Boundaries and layers

### 14.5.1. Unidirectional data flow (recap)

Props down, events up (File 03). Global client state via Context or store (Files 08, 11). Server state via query library (File 10).

### 14.5.2. Avoid circular imports

Features import from **`components/`** and **`lib/`**, not from each other. Shared types in **`lib/types`** or feature-agnostic module.

### 14.5.3. Environment and config

```jsx
// lib/config.js
export const API_BASE = import.meta.env.VITE_API_URL;
```

Never hardcode secrets in client bundle — env vars prefixed for exposure (Vite `VITE_`).

---

## 14.6. Scaling patterns

### 14.6.1. Route-based code splitting

File 12 — lazy routes per feature folder reduce initial bundle.

### 14.6.2. Design system layer

Shared **`Button`**, **`Input`**, **`Modal`** — consistent a11y and styling. Feature components compose design system pieces.

### 14.6.3. Documentation and Storybook

Optional **Storybook** for presentational components — visual catalog isolated from data fetching.

---

## 14.7. Anti-patterns to avoid

| Anti-pattern | Better approach |
|--------------|-----------------|
| God **`App.jsx`** with all state | Feature modules + providers |
| Fetch in every leaf component | Route loader or shared query hooks |
| Copy-paste Context for each feature | Feature-local state or Zustand slice |
| Prop drilling 6 levels | Context or composition |
| Catching errors only at root | Nested error boundaries |

---

## 14.8. Putting it together

### 14.8.1. Minimal production-shaped app

```
src/
  app/
    App.jsx
    router.jsx
    providers.jsx
  features/
    auth/
    products/
    cart/
  components/
    ui/
      Button.jsx
      ErrorBoundary.jsx
  lib/
    apiClient.js
    queryClient.js
```

```jsx
// app/router.jsx
const router = createBrowserRouter([
  {
    element: (
      <ErrorBoundary fallback={<AppError />}>
        <RootLayout />
      </ErrorBoundary>
    ),
    children: [
      { index: true, element: <Home /> },
      { path: "products/*", element: <ProductRoutes /> },
    ],
  },
]);
```

Each **`features/*`** owns routes segment, pages, hooks, and API for that domain.

---

## 14.9. Common questions

**14.9.1. Feature folders vs pages folders?**  
A: **Feature-first** scales better — everything for "products" lives together. **`pages/`** can alias route entry components inside features.

**14.9.2. Where do shared hooks go?**  
A: **`hooks/`** at root if truly generic; otherwise **inside the feature** that owns them.

**14.9.3. What are compound components?**  
A: A set of components (**`Tabs`**, **`Tabs.Tab`**) that **share implicit state** via internal Context for a flexible public API.

**14.9.4. Do error boundaries catch useEffect errors?**  
A: **Not async errors** inside effects unless re-thrown during render. Handle fetch errors in UI state (File 10).

**14.9.5. Can I use error boundary with function components?**  
A: Boundaries are **class components** today (or library wrappers). Handlers use **try/catch**.

**14.9.6. How many providers is too many?**  
A: Combine in **`AppProviders`**. Split contexts by concern (File 08) — not one mega-provider.

**14.9.7. Monorepo?**  
A: Shared **`ui`** package + app packages per product — same feature principles at package level.

**14.9.8. TypeScript?**  
A: Same structure — add **`types.ts`** per feature for API shapes and props.

**14.9.9. When to split a feature?**  
A: When folder grows hard to navigate (~15+ files) or two unrelated domains share a folder — extract sub-features.

**14.9.10. End of React track — what next?**  
A: Build a small full-stack app applying Files 01–14. Deepen **File 10** (TanStack Query), **File 12** (perf), or explore **Next.js** for SSR/file routing.
