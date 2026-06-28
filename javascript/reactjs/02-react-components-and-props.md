# 02. Components & Props

> File 02 — function components, props patterns, children, composition.

File 01 introduced components and basic props. This chapter goes deeper: how to **structure components**, pass props **flexibly**, use **`children`** for layouts, and **compose** UI without inheritance.

Props stay **read-only** — if a child needs to change data, the parent updates **state** and passes new props (File 03).

**Default rule today:** prefer **composition** over inheritance; keep **one main export** per file when possible; use **destructuring** and **spread** for props; lift shared UI into **layout/wrapper** components.

---

## 2.1. Function components in depth

### 2.1.1. Anatomy of a component file

Typical file layout:

```jsx
// UserCard.jsx
import { Avatar } from "./Avatar.jsx";
import styles from "./UserCard.module.css";

export function UserCard({ name, role, avatarUrl }) {
  return (
    <article className={styles.card}>
      <Avatar src={avatarUrl} alt={name} />
      <h2>{name}</h2>
      <p>{role}</p>
    </article>
  );
}
```

Conventions:

- **PascalCase** filename matching component (`UserCard.jsx`).
- **Named export** for reusable components; **default export** for page/route root (team choice — stay consistent).
- Keep **markup + minimal logic** in the component; extract helpers to separate files if they grow.

### 2.1.2. Named vs default export

```jsx
// Named — explicit imports, good for multiple exports
export function Button() { /* ... */ }
export function IconButton() { /* ... */ }

import { Button, IconButton } from "./Button.jsx";
```

```jsx
// Default — one primary component per file
export default function Dashboard() { /* ... */ }

import Dashboard from "./Dashboard.jsx";
```

Avoid mixing many default exports in one file.

### 2.1.3. Components must return valid UI

Return JSX, `null`, or a **Fragment**. Do not return plain objects or undefined accidentally:

```jsx
function MaybeShow({ show, content }) {
  if (!show) return null;   // render nothing
  return <p>{content}</p>;
}
```

Early returns keep JSX readable (File 04 — conditional rendering).

### 2.1.4. Single responsibility

Split when a component:

- Exceeds ~100–150 lines (guideline, not a rule).
- Handles unrelated concerns (fetch + form + table).
- Is reused with small variations → extract props or `children`.

```jsx
// Before — one big component
function UserPage() {
  /* header + profile + posts + footer */
}

// After — composed
function UserPage() {
  return (
    <PageLayout>
      <UserHeader />
      <UserProfile />
      <UserPosts />
    </PageLayout>
  );
}
```

---

## 2.2. Props patterns

### 2.2.1. Destructuring and renaming

```jsx
function UserCard({ name: displayName, age, isActive = false }) {
  return (
    <div>
      <h2>{displayName}</h2>
      <p>{age}</p>
      {isActive && <span>Online</span>}
    </div>
  );
}
```

Rest props — forward unknown props to a DOM element:

```jsx
function PrimaryButton({ label, ...rest }) {
  return (
    <button type="button" className="btn-primary" {...rest}>
      {label}
    </button>
  );
}

<PrimaryButton label="Save" onClick={handleSave} disabled={loading} />
```

Useful for native attributes (`aria-*`, `data-*`, `className`).

### 2.2.2. Passing objects and spreading

Parent bundles related props:

```jsx
const user = { name: "Bharat", age: 30, role: "Dev" };

<UserCard {...user} />
// same as <UserCard name={user.name} age={user.age} role={user.role} />
```

Child picks what it needs — extra keys on DOM spread can warn in React; only spread onto DOM what belongs there.

Explicit is often clearer for public APIs:

```jsx
<UserCard name={user.name} role={user.role} />
```

### 2.2.3. Boolean and optional props

**Boolean props** — shorthand when `true`:

```jsx
function Modal({ open, dismissible }) {
  if (!open) return null;
  return (
    <div className="modal">
      {dismissible && <button>Close</button>}
    </div>
  );
}

<Modal open dismissible />   // open={true} dismissible={true}
<Modal open={false} />
```

**Optional props** — defaults or conditional render:

```jsx
function Alert({ title, message, variant = "info" }) {
  return (
    <div className={`alert alert-${variant}`}>
      {title && <strong>{title}</strong>}
      <p>{message}</p>
    </div>
  );
}
```

### 2.2.4. Props are a snapshot

Props do not update inside the child by themselves — parent re-render passes **new** props:

```jsx
function Child({ count }) {
  // count is whatever parent passed for this render
  return <p>{count}</p>;
}

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <Child count={count} />
      <button onClick={() => setCount(count + 1)}>+</button>
    </>
  );
}
```

Never mutate props:

```jsx
// Bad
function Bad({ user }) {
  user.name = "Changed";  // mutates parent's object
}
```

### 2.2.5. Typing props (PropTypes / TypeScript)

JavaScript — optional **PropTypes** (runtime, dev only):

```jsx
import PropTypes from "prop-types";

function UserCard({ name, age }) {
  return <p>{name}, {age}</p>;
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};
```

**TypeScript** — preferred in many codebases:

```tsx
type UserCardProps = {
  name: string;
  age?: number;
};

function UserCard({ name, age }: UserCardProps) {
  return <p>{name}, {age ?? "N/A"}</p>;
}
```

Types document the public contract of a component.

### 2.2.6. Callback props

Parent passes **functions** so child can notify parent (events up, data down):

```jsx
function SearchInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}

function Parent() {
  const [query, setQuery] = useState("");
  return <SearchInput value={query} onChange={setQuery} />;
}
```

Naming: `on` + event for handlers (`onClick`, `onSubmit`, `onSelectUser`). Full event flow in File 03.

---

## 2.3. The `children` prop

### 2.3.1. Basic children

```jsx
function Panel({ title, children }) {
  return (
    <section>
      <h2>{title}</h2>
      <div className="panel-body">{children}</div>
    </section>
  );
}

<Panel title="Settings">
  <p>Content here</p>
  <button>Save</button>
</Panel>
```

`children` can be:

- Text: `<Button>Click</Button>`
- Element(s): nested JSX
- Expression: `{items.map(...)}` (File 04)

### 2.3.2. Multiple slots (explicit props)

When you need **named regions**, use extra props instead of only `children`:

```jsx
function Card({ header, footer, children }) {
  return (
    <article>
      <header>{header}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </article>
  );
}

<Card
  header={<h1>Title</h1>}
  footer={<button>OK</button>}
>
  <p>Body content</p>
</Card>
```

Alternative names: `sidebar`, `actions`, `leading`, `trailing`.

### 2.3.3. Layout components

**Layout** components wrap pages with shared chrome:

```jsx
function AppLayout({ children }) {
  return (
    <div className="app">
      <Navbar />
      <main className="content">{children}</main>
      <Footer />
    </div>
  );
}

function App() {
  return (
    <AppLayout>
      <Dashboard />
    </AppLayout>
  );
}
```

Same pattern for `Modal`, `Card`, `Sidebar`, `Stack`, `Grid` wrappers.

### 2.3.4. Children as function (render prop intro)

Pass a **function** as child to share data from parent wrapper:

```jsx
function MouseTracker({ children }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });

  return (
    <div
      onMouseMove={(e) => setPos({ x: e.clientX, y: e.clientY })}
    >
      {children(pos)}
    </div>
  );
}

<MouseTracker>
  {({ x, y }) => <p>Mouse: {x}, {y}</p>}
</MouseTracker>
```

Also called **render props**. Custom hooks often replace this pattern today (File 07).

---

## 2.4. Composition vs inheritance

React favors **composition** — building UI by nesting components and passing props/`children`.

### 2.4.1. Compose, do not extend

```jsx
// Prefer — composition
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <Dialog title="Welcome">
      <p>Thanks for signing up.</p>
    </Dialog>
  );
}
```

```jsx
// Avoid — class inheritance (legacy pattern)
// class WelcomeDialog extends Dialog { ... }
```

Configure behavior with **props**, not subclass overrides.

### 2.4.2. Specialization via props

```jsx
function Button({ variant = "primary", size = "md", ...rest }) {
  const className = `btn btn-${variant} btn-${size}`;
  return <button className={className} {...rest} />;
}

function DangerButton(props) {
  return <Button variant="danger" {...props} />;
}
```

**Wrapper components** reuse base UI with preset props.

### 2.4.3. Containment vs specialization

| Pattern | Example |
|---------|---------|
| **Containment** | `Card` wraps arbitrary `children` |
| **Specialization** | `PrimaryButton` sets `variant="primary"` on `Button` |

Both are composition — no class hierarchy.

---

## 2.5. Component composition patterns

### 2.5.1. Small presentational components

Split **data** (File 03, 10) from **presentation** when helpful:

```jsx
function UserAvatar({ user, size = 40 }) {
  return (
    <img
      src={user.avatarUrl}
      alt={user.name}
      width={size}
      height={size}
      style={{ borderRadius: "50%" }}
    />
  );
}

function UserRow({ user }) {
  return (
    <li className="user-row">
      <UserAvatar user={user} />
      <span>{user.name}</span>
      <span>{user.role}</span>
    </li>
  );
}
```

Not a strict rule — use when it improves reuse and tests.

### 2.5.2. Config-driven UI

Pass arrays of props to map into components (File 04):

```jsx
const links = [
  { href: "/", label: "Home" },
  { href: "/about", label: "About" },
];

function Nav() {
  return (
    <nav>
      {links.map((link) => (
        <a key={link.href} href={link.href}>{link.label}</a>
      ))}
    </nav>
  );
}
```

### 2.5.3. Prop drilling (preview)

Passing props through many layers:

```jsx
function App() {
  const user = { name: "Bharat" };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserMenu user={user} />;
}
```

Works for shallow trees. Deep drilling → **Context** (File 08) or state library (File 11).

### 2.5.4. Fragment in composition

Avoid extra DOM nodes when wrapping siblings for export:

```jsx
function TableRow({ cells }) {
  return (
    <>
      {cells.map((cell, i) => (
        <td key={i}>{cell}</td>
      ))}
    </>
  );
}
```

Long form: `<React.Fragment key={id}>`. Details in File 04.

---

## 2.6. Common questions

**2.6.1. Can a component update its own props?**  
A: **No.** Props are read-only. The **parent** owns the data and passes new props on re-render.

**2.6.2. What is the difference between props and state?**  
A: **Props** are external inputs; **state** is internal (File 01, File 03). Components react to both by re-rendering.

**2.6.3. When should I use `children` vs a named prop like `footer`?**  
A: Use **`children`** for a single main slot (typical wrapper). Use **named props** for multiple distinct regions (header, footer, sidebar).

**2.6.4. What does spreading props (`{...props}`) do?**  
A: Copies all properties onto the target element or component. Useful for forwarding HTML attributes; avoid spreading unknown objects onto DOM nodes.

**2.6.5. Why compose instead of inherit in React?**  
A: Composition is **flexible** and avoids fragile class hierarchies. React's model is built around nesting components and passing props.

**2.6.6. What is a callback prop?**  
A: A function passed from **parent to child** so the child can report events or data upward (`onChange`, `onSelect`).

**2.6.7. What is prop drilling?**  
A: Passing props through **intermediate** components that do not use them, only to reach a deep child. Fix with Context or global state when it hurts maintainability.

**2.6.8. Named or default export?**  
A: Either works. **Named** exports scale well for component libraries; **default** exports are common for page-level components. Pick one style per project.

**2.6.9. What is a render prop?**  
A: A prop (often `children`) that is a **function** receiving data from the parent component and returning UI. Custom hooks are often a simpler alternative (File 07).

**2.6.10. What should I read next?**  
A: File 03 (state & events), File 04 (lists & conditional rendering).
