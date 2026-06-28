# React.js

Notes for React on the web.

Files are numbered in **learning order**: `01-react-topic.md`, `02-react-topic.md`, and so on.

**Prerequisite:** complete [JavaScript Language](../javascript-language/README.md) (at least files 01–07 and 10) before React hooks and async UI.

## Contents

| # | File | Tier | Topics |
|---|------|------|--------|
| 01 | [01-react-fundamental](./01-react-fundamental.md) | Fundamental | Setup, JSX, first components, props, state intro |
| 02 | [02-react-components-and-props](./02-react-components-and-props.md) | Fundamental | Function components, props, children, composition |
| 03 | [03-react-state-and-events](./03-react-state-and-events.md) | Fundamental | useState, events, lifting state up, one-way data flow |
| 04 | [04-react-rendering-lists](./04-react-rendering-lists.md) | Fundamental | Conditional UI, lists, keys, fragments |
| 05 | [05-react-use-effect](./05-react-use-effect.md) | Fundamental | useEffect, deps, cleanup, sync with external systems |
| 06 | [06-react-forms](./06-react-forms.md) | Fundamental | Controlled inputs, validation basics, form submit |
| 07 | [07-react-hooks-patterns](./07-react-hooks-patterns.md) | Fundamental | Rules of hooks, custom hooks, useRef, useId |
| 08 | [08-react-context-and-reducer](./08-react-context-and-reducer.md) | Intermediate | useContext, useReducer, avoiding prop drilling |
| 09 | [09-react-routing](./09-react-routing.md) | Intermediate | React Router, nested routes, URL params, loaders |
| 10 | [10-react-data-fetching](./10-react-data-fetching.md) | Intermediate | fetch, loading/error states, TanStack Query patterns |
| 11 | [11-react-state-management](./11-react-state-management.md) | Intermediate | Local vs global state, Zustand / Redux Toolkit basics |
| 12 | [12-react-performance-and-patterns](./12-react-performance-and-patterns.md) | Advanced | memo, useMemo, useCallback, code splitting, Suspense |
| 13 | [13-react-testing](./13-react-testing.md) | Advanced | React Testing Library, user events, mocking |
| 14 | [14-react-architecture](./14-react-architecture.md) | Advanced | Folder structure, compound components, error boundaries |



## Learning path

```
01–07  Fundamental  →  components, state, effects, forms, hooks
08–11  Intermediate →  context, routing, server state, global store
12–14  Advanced     →  performance, testing, architecture
```

Each file has its own numbered sections (`## 2.1.` in file 02, `## 3.1.` in file 03, etc.) and a **Common questions** block at the end — same convention as JavaScript Language notes (section prefix matches file number).

## Topic map (by area)

| Area | Files |
|------|-------|
| Core UI | 01, 02, 03, 04 |
| Side effects & forms | 05, 06 |
| Hooks depth | 07, 08 |
| App features | 09, 10, 11 |
| Production readiness | 12, 13, 14 |
