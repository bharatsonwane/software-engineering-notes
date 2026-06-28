# 10. TypeScript with React

> File 10 — component props, useState, events, hooks, context, @types.

React and TypeScript work together through **typed props**, **generic hooks**, and **event types** from `@types/react`.

**Prerequisite:** [React.js notes](../reactjs/) (files 01–06) before this chapter.

---

## 1. Typing components and props

### 1.1. Function components

```tsx
type ButtonProps = {
  label: string;
  onClick: () => void;
  disabled?: boolean;
};

function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button type="button" onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```

Or with `interface`:

```tsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <section>
      <h2>{title}</h2>
      {children}
    </section>
  );
}
```

### 1.2. `React.FC` / `React.FunctionComponent`

Older style — optional `children` was implicit. Modern codebases prefer **explicit props**:

```tsx
// Avoid unless team standard
const OldButton: React.FC<ButtonProps> = ({ label, onClick }) => (
  <button onClick={onClick}>{label}</button>
);
```

### 1.3. Extending HTML element props

```tsx
type InputProps = React.InputHTMLAttributes<HTMLInputElement> & {
  label: string;
};

function LabeledInput({ label, ...rest }: InputProps) {
  return (
    <label>
      {label}
      <input {...rest} />
    </label>
  );
}
```

Use `ComponentPropsWithoutRef<"button">` or `ComponentProps<typeof SomeComponent>` to inherit props.

### 1.4. Typing `children`

```tsx
type Props = {
  children: React.ReactNode; // string, number, element, fragment, null, array, etc.
};
```

Stricter alternatives: `React.ReactElement` (single element), `React.ReactNode` (most common).

---

## 2. `useState` and event types

### 2.1. `useState` inference and explicit types

```tsx
const [count, setCount] = useState(0); // number

const [user, setUser] = useState<User | null>(null);

const [ids, setIds] = useState<string[]>([]);
// Prefer useState<string[]>([]) when initial value is []
```

### 2.2. Event handlers

React events are **synthetic** — use types from `@types/react`:

```tsx
function SearchForm() {
  const [query, setQuery] = useState("");

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
    </form>
  );
}
```

Common event types:

| Event | Type |
|-------|------|
| Input change | `React.ChangeEvent<HTMLInputElement>` |
| Button click | `React.MouseEvent<HTMLButtonElement>` |
| Form submit | `React.FormEvent<HTMLFormElement>` |
| Keyboard | `React.KeyboardEvent<HTMLElement>` |

### 2.3. Callback props

```tsx
type ListProps = {
  items: string[];
  onSelect: (item: string, index: number) => void;
};
```

---

## 3. Typing hooks and context

### 3.1. Custom hooks

Return type is often inferred; annotate for public hooks:

```tsx
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount((c) => c + 1);
  return { count, increment, setCount };
}

type UseCounterReturn = ReturnType<typeof useCounter>;
```

Generic custom hook:

```tsx
function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const raw = localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as T) : initial;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

### 3.2. `useReducer`

```tsx
type State = { count: number };
type Action =
  | { type: "increment" }
  | { type: "decrement" }
  | { type: "reset" };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return { count: 0 };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  // ...
}
```

### 3.3. Context

```tsx
type Theme = "light" | "dark";

type ThemeContextValue = {
  theme: Theme;
  toggle: () => void;
};

const ThemeContext = createContext<ThemeContextValue | null>(null);

function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) {
    throw new Error("useTheme must be used within ThemeProvider");
  }
  return ctx;
}

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");
  const toggle = () => setTheme((t) => (t === "light" ? "dark" : "light"));

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

## 4. Third-party types (`@types/*`)

Many JS libraries ship types. Others need **DefinitelyTyped** packages:

```bash
npm install --save-dev @types/react @types/react-dom
npm install --save-dev @types/node
npm install --save-dev @types/lodash
```

### 4.1. Library includes its own types

Modern packages embed `.d.ts`:

```json
// package.json of a typed library
{
  "name": "some-lib",
  "types": "dist/index.d.ts"
}
```

No `@types/some-lib` needed.

### 4.2. No types available

Create a declaration file:

```ts
// types/untyped-lib.d.ts
declare module "untyped-lib" {
  export function doSomething(input: string): number;
}
```

Or use `// @ts-expect-error` temporarily — fix with proper types or wrapper.

### 4.3. `ref` typing

```tsx
const inputRef = useRef<HTMLInputElement>(null);

useEffect(() => {
  inputRef.current?.focus();
}, []);

return <input ref={inputRef} />;
```

---

## 5. Common questions

**5.1. Should I use `interface` or `type` for props?**  
A: Either. `type` needed for unions (`Props | DisabledProps`). Teams often pick one and stay consistent.

**5.2. Why `useState<string[]>([])` instead of `useState([])`?**  
A: Empty array infers `never[]` — explicit generic sets element type.

**5.3. What is `React.ReactNode` vs `React.ReactElement`?**  
A: `ReactNode` is anything renderable. `ReactElement` is a single JSX element — stricter.

**5.4. How do I type `forwardRef`?**  
A: `const Input = forwardRef<HTMLInputElement, InputProps>((props, ref) => ...)`.

**5.5. Do I need `@types/react` with React 18+?**  
A: Types are still published via `@types/react` — install as dev dependency.

**5.6. How to type generic table/list components?**  
A: `function List<T>({ items, render }: { items: T[]; render: (item: T) => React.ReactNode })`.

**5.7. What about `propTypes`?**  
A: TypeScript replaces PropTypes for TS projects. PropTypes optional for JS interop.

**5.8. Strict null and optional props?**  
A: `disabled?: boolean` allows `undefined`. With `exactOptionalPropertyTypes`, omitting vs passing `undefined` differs.
