# 13. Testing

> File 13 — React Testing Library, user events, mocking.

Tests give confidence that components **behave correctly for users** — not that implementation details match snapshots. **React Testing Library (RTL)** queries the DOM like a user and fires **real events**. Pair with **Vitest** or **Jest** as the test runner.

Builds on components, state, forms, and hooks (Files 02–07). File 14 covers error boundaries tested similarly.

**Default rule today:** test **behavior and accessibility**, not internal state; query by **role/label/text**; use **`userEvent`** over raw `fireEvent` when simulating typing and clicks; mock **network**, not child components, unless isolating a unit.

---

## 13.1. Testing philosophy

### 13.1.1. Test user-visible behavior

```jsx
// Good — what the user sees and does
expect(screen.getByRole("button", { name: /submit/i })).toBeEnabled();
await user.click(screen.getByRole("button", { name: /submit/i }));
expect(await screen.findByText(/success/i)).toBeInTheDocument();

// Avoid — testing implementation
expect(component.state.count).toBe(1);
```

Users do not see `state` or hook names — tests should not depend on them.

### 13.1.2. Testing Library guiding principle

> The more your tests resemble the way your software is used, the more confidence they can give you.

---

## 13.2. Setup

### 13.2.1. Typical stack (Vite + Vitest)

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

```js
// vitest.config.js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.js"],
  },
});
```

```js
// src/test/setup.js
import "@testing-library/jest-dom/vitest";
```

CRA/Jest projects use similar packages with `setupTests.js`.

### 13.2.2. Basic test

```jsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import Greeting from "./Greeting";

describe("Greeting", () => {
  it("shows the name", () => {
    render(<Greeting name="Ada" />);
    expect(screen.getByText("Hello, Ada")).toBeInTheDocument();
  });
});
```

---

## 13.3. Queries (priority order)

### 13.3.1. Preferred queries

| Priority | Query | Example |
|----------|-------|---------|
| 1 | **getByRole** | `getByRole("button", { name: /save/i })` |
| 2 | **getByLabelText** | `getByLabelText(/email/i)` |
| 3 | **getByPlaceholderText** | `getByPlaceholderText(/search/i)` |
| 4 | **getByText** | `getByText(/welcome/i)` |
| 5 | **getByDisplayValue** | `getByDisplayValue("Ada")` |
| Last resort | **getByTestId** | `getByTestId("custom-widget")` |

**Role + accessible name** encourages accessible markup (File 06 — labels).

### 13.3.2. getBy vs queryBy vs findBy

| Variant | Async | Empty result |
|---------|-------|--------------|
| **getBy** | No | **Throws** |
| **queryBy** | No | Returns `null` |
| **findBy** | Yes (waits) | Throws after timeout |

```jsx
expect(screen.queryByText(/error/i)).not.toBeInTheDocument();

await screen.findByText(/loaded/i); // waits for async UI
```

### 13.3.3. within — scope queries

```jsx
const row = screen.getByRole("row", { name: /ada/i });
within(row).getByRole("button", { name: /edit/i });
```

---

## 13.4. User events

### 13.4.1. userEvent.setup()

```jsx
import userEvent from "@testing-library/user-event";

it("updates input", async () => {
  const user = userEvent.setup();
  render(<SearchBox />);

  const input = screen.getByRole("textbox", { name: /search/i });
  await user.type(input, "react");
  expect(input).toHaveValue("react");
});
```

**`userEvent`** simulates realistic keyboard/mouse sequences — preferred over **`fireEvent.click`** for interactions.

### 13.4.2. Form submit

```jsx
await user.type(screen.getByLabelText(/email/i), "a@b.com");
await user.type(screen.getByLabelText(/password/i), "secret123");
await user.click(screen.getByRole("button", { name: /log in/i }));

expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
```

Matches File 06 form patterns.

### 13.4.3. Keyboard

```jsx
await user.keyboard("{Enter}");
await user.tab();
```

---

## 13.5. Testing async and data

### 13.5.1. waitFor

```jsx
import { waitFor } from "@testing-library/react";

await waitFor(() => {
  expect(screen.getByText("Ada Lovelace")).toBeInTheDocument();
});
```

Prefer **`findBy*`** when waiting for one element.

### 13.5.2. Mocking fetch

```jsx
import { vi } from "vitest";

beforeEach(() => {
  vi.stubGlobal(
    "fetch",
    vi.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({ name: "Ada" }),
      })
    )
  );
});

afterEach(() => {
  vi.unstubAllGlobals();
});
```

Or use **MSW (Mock Service Worker)** for realistic HTTP mocking across tests.

### 13.5.3. QueryClient wrapper

For TanStack Query components (File 10):

```jsx
function renderWithQuery(ui) {
  const client = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return render(
    <QueryClientProvider client={client}>{ui}</QueryClientProvider>
  );
}
```

Disable retries in tests for faster failures.

---

## 13.6. Providers and routing

### 13.6.1. Custom render helper

```jsx
function renderWithProviders(ui, { route = "/" } = {}) {
  window.history.pushState({}, "", route);
  return render(
    <QueryClientProvider client={testQueryClient}>
      <ThemeProvider>
        <MemoryRouter initialEntries={[route]}>
          {ui}
        </MemoryRouter>
      </ThemeProvider>
    </QueryClientProvider>
  );
}
```

Wrap **Context**, **Router**, and **Query** once — reuse in tests (File 08, File 09).

### 13.6.2. MemoryRouter

```jsx
import { MemoryRouter } from "react-router-dom";

render(
  <MemoryRouter initialEntries={["/products/42"]}>
    <ProductDetail />
  </MemoryRouter>
);
```

---

## 13.7. Testing hooks

### 13.7.1. renderHook

```jsx
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

it("increments", () => {
  const { result } = renderHook(() => useCounter(0));

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

File 07 — custom hooks tested in isolation.

### 13.7.2. Wrapper for hook context

```jsx
renderHook(() => useTheme(), {
  wrapper: ThemeProvider,
});
```

---

## 13.8. Mocking modules

### 13.8.1. vi.mock

```jsx
vi.mock("./api", () => ({
  fetchUser: vi.fn(() => Promise.resolve({ id: 1, name: "Ada" })),
}));
```

Mock **boundaries** (API module), not every child component.

### 13.8.2. Avoid mocking React itself

Do not mock `useState` — test through UI. Exception: error boundary tests with forced throws.

---

## 13.9. Accessibility checks

### 13.9.1. jest-axe (optional)

```jsx
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

it("has no a11y violations", async () => {
  const { container } = render(<LoginForm />);
  expect(await axe(container)).toHaveNoViolations();
});
```

RTL queries by role already push accessible markup.

---

## 13.10. What to test per feature

| Feature | Test ideas |
|---------|------------|
| Form (File 06) | Validation messages, submit disabled, success path |
| List (File 04) | Renders items, empty state, click row action |
| Modal | Opens/closes, focus trap optional |
| Auth route (File 09) | Redirect when logged out |
| Error UI | Error message when fetch fails |

Skip snapshot-only tests of large trees — brittle and low signal.

---

## 13.11. Putting it together

### 13.11.1. Login form test

```jsx
describe("LoginForm", () => {
  it("shows error on invalid email", async () => {
    const user = userEvent.setup();
    const onLogin = vi.fn();
    render(<LoginForm onLogin={onLogin} />);

    await user.type(screen.getByLabelText(/email/i), "bad");
    await user.click(screen.getByRole("button", { name: /log in/i }));

    expect(screen.getByRole("alert")).toHaveTextContent(/valid email/i);
    expect(onLogin).not.toHaveBeenCalled();
  });

  it("calls onLogin with credentials", async () => {
    const user = userEvent.setup();
    const onLogin = vi.fn();
    render(<LoginForm onLogin={onLogin} />);

    await user.type(screen.getByLabelText(/email/i), "a@b.com");
    await user.type(screen.getByLabelText(/password/i), "password1");
    await user.click(screen.getByRole("button", { name: /log in/i }));

    expect(onLogin).toHaveBeenCalledWith({
      email: "a@b.com",
      password: "password1",
    });
  });
});
```

---

## 13.12. Common questions

**13.12.1. RTL vs Enzyme?**  
A: **RTL** is the modern default — user-centric. Enzyme shallow-renders implementation details; avoid for new projects.

**13.12.2. getByRole vs getByTestId?**  
A: Prefer **role** — forces accessible UI. **testId** only when no semantic query works.

**13.12.3. fireEvent vs userEvent?**  
A: **`userEvent`** simulates full interaction (focus, keydown, input). **`fireEvent`** is lower level — use for edge cases.

**13.12.4. How do I test async fetch?**  
A: Mock **`fetch`** or **MSW**; use **`findBy*`** or **`waitFor`**. Disable query retries in tests.

**13.12.5. Should I test implementation details?**  
A: **No** — avoid asserting private state, hook call order, or child component props unless unit-testing a hook directly.

**13.12.6. How do I test React Router?**  
A: Wrap in **`MemoryRouter`** with **`initialEntries`**, assert navigation with **`screen`** or **`useNavigate` mock**.

**13.12.7. How do I test custom hooks?**  
A: **`renderHook`** + **`act`** for updates. Provide **wrapper** with required providers.

**13.12.8. Snapshot tests — yes or no?**  
A: Sparingly for **stable** small output. Prefer behavior assertions.

**13.12.9. Where do test files live?**  
A: Colocated **`Component.test.jsx`** next to source, or **`__tests__/`** folder — team convention.

**13.12.10. What should I read next?**  
A: File 14 (error boundaries, architecture), File 06/09 for features to test, MSW docs for API mocking.
