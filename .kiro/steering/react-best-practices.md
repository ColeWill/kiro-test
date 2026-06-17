---
inclusion: always
---

# React Best Practices & Conventions

## Folder Structure — Feature-Based

Organize code by feature, not by type. Each feature is self-contained with its own components, hooks, state, services, and tests colocated in one directory.

```
src/
├── features/
│   ├── navbar/
│   │   ├── Navbar.tsx
│   │   ├── NavItem.tsx
│   │   ├── useNavbar.ts          # feature-specific hook
│   │   ├── navbarSlice.ts        # Redux slice
│   │   ├── navbarService.ts      # API calls / business logic
│   │   ├── navbar.types.ts       # types specific to this feature
│   │   ├── navbar.test.tsx
│   │   └── index.ts              # public API barrel export
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── SignupForm.tsx
│   │   ├── useAuth.ts
│   │   ├── authSlice.ts
│   │   ├── authService.ts
│   │   ├── auth.types.ts
│   │   └── index.ts
│   └── dashboard/
│       ├── Dashboard.tsx
│       ├── DashboardWidget.tsx
│       ├── useDashboard.ts
│       ├── dashboardSlice.ts
│       └── index.ts
├── shared/
│   ├── components/               # truly reusable UI primitives (Button, Modal, Input)
│   ├── hooks/                    # generic hooks used across features (useDebounce, useMediaQuery)
│   ├── utils/                    # pure utility functions
│   └── types/                    # shared/global types
├── app/
│   ├── App.tsx
│   ├── store.ts                  # Redux store configuration
│   ├── router.tsx                # route definitions
│   └── providers.tsx             # context providers wrapper
└── index.tsx
```

### Rules

- A feature folder owns everything related to that feature.
- If a component is used in only one feature, it lives in that feature's folder.
- Only move something to `shared/` when it's genuinely used by 2+ features.
- Each feature exports its public API through an `index.ts` barrel file. Other features import from the barrel, never from internal files.
- Avoid deep nesting. One level of feature folders is usually enough.

## Component Patterns

### Functional components only

Never use class components. Always use function components with hooks.

### Keep components small and focused

A component should do one thing. If it grows past ~150 lines, break it up.

### Separate logic from presentation

Extract business logic and side effects into custom hooks. Components should primarily handle rendering.

```tsx
// useNavbar.ts — logic
export function useNavbar() {
  const items = useSelector(selectNavItems);
  const dispatch = useDispatch();

  const handleItemClick = (id: string) => {
    dispatch(setActiveItem(id));
  };

  return { items, handleItemClick };
}

// Navbar.tsx — presentation
export function Navbar() {
  const { items, handleItemClick } = useNavbar();

  return (
    <nav>
      {items.map((item) => (
        <NavItem key={item.id} item={item} onClick={handleItemClick} />
      ))}
    </nav>
  );
}
```

### Props

- Use TypeScript interfaces for props, named `<Component>Props`.
- Destructure props in the function signature.
- Avoid passing more than 5-6 props. If you need more, consider composition or context.
- Use `children` and composition over deep prop drilling.

```tsx
interface NavItemProps {
  item: NavItem;
  onClick: (id: string) => void;
  isActive?: boolean;
}

export function NavItem({ item, onClick, isActive = false }: NavItemProps) {
  // ...
}
```

## Hooks

### Custom hook naming

Always prefix with `use`. Name them after what they provide: `useAuth`, `useNavbar`, `useDebounce`.

### Rules of Hooks

- Only call hooks at the top level. Never inside conditions, loops, or nested functions.
- Only call hooks from React function components or other custom hooks.

### Avoid excessive useEffect

`useEffect` is for synchronizing with external systems, not for derived state or transformations.

```tsx
// BAD — derived state in useEffect
const [fullName, setFullName] = useState("");
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// GOOD — compute during render
const fullName = `${firstName} ${lastName}`;
```

### Common useEffect traps to avoid

- Don't use useEffect to update state based on other state (use derived values or useMemo).
- Don't use useEffect for event handlers (attach handlers directly).
- Don't forget cleanup functions for subscriptions and timers.
- Always include correct dependencies — don't lie to the dependency array.

## State Management

### Local vs Global

- Use `useState` / `useReducer` for UI state that only one component cares about.
- Use Redux (via Redux Toolkit) for state that multiple features need or that persists across routes.
- Use context sparingly — it's for dependency injection (theme, auth), not general state management.

### Redux Toolkit conventions

- One slice per feature, colocated in the feature folder.
- Use `createSlice` — never write reducers or action creators manually.
- Use `createAsyncThunk` for async operations.
- Keep selectors in the slice file. Prefix with `select`.
- Normalize complex data with `createEntityAdapter` when appropriate.

```tsx
// navbarSlice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface NavbarState {
  activeItemId: string | null;
}

const initialState: NavbarState = {
  activeItemId: null,
};

const navbarSlice = createSlice({
  name: "navbar",
  initialState,
  reducers: {
    setActiveItem(state, action: PayloadAction<string>) {
      state.activeItemId = action.payload;
    },
  },
});

export const { setActiveItem } = navbarSlice.actions;
export const selectActiveItemId = (state: RootState) =>
  state.navbar.activeItemId;
export default navbarSlice.reducer;
```

## Performance

### Avoid unnecessary re-renders

- Use `React.memo` only when profiling shows a problem — don't sprinkle it everywhere.
- Use `useCallback` for functions passed as props to memoized children.
- Use `useMemo` for expensive computations, not for every variable.
- Never create objects or arrays inline in JSX props (creates new references each render).

```tsx
// BAD — new object every render
<NavItem style={{ color: "red" }} />;

// GOOD — stable reference
const itemStyle = { color: "red" };
<NavItem style={itemStyle} />;
```

### Lazy loading

Use `React.lazy` + `Suspense` for route-level code splitting.

```tsx
const Dashboard = React.lazy(() => import("./features/dashboard"));
```

## Common Pitfalls to Avoid

1. **Prop drilling** — Use composition, context, or Redux instead of passing props through 4+ layers.
2. **God components** — Break up components that do too much.
3. **Index files that re-export everything** — Only export the public API.
4. **Putting everything in global state** — Most state is local. Start local, elevate only when needed.
5. **Forgetting keys in lists** — Always use stable, unique keys. Never use array index as key if the list can reorder.
6. **Direct DOM manipulation** — Use refs only when React can't handle it (focus, measurements, third-party libs).
7. **Async state updates without cleanup** — Always handle component unmount in async operations.
8. **Circular imports** — Barrel files help, but watch for features importing from each other. If two features need shared logic, move it to `shared/`.

## TypeScript Conventions

- Prefer `interface` for object shapes, `type` for unions and intersections.
- Export types alongside the code that uses them.
- Avoid `any`. Use `unknown` when the type is genuinely not known, then narrow it.
- Use generics to make reusable hooks and utilities type-safe.
- Define return types on exported functions for better documentation and error messages.

## Naming Conventions

- Components: `PascalCase` — `Navbar.tsx`, `LoginForm.tsx`
- Hooks: `camelCase` with `use` prefix — `useAuth.ts`, `useNavbar.ts`
- Slices: `camelCase` with `Slice` suffix — `navbarSlice.ts`
- Services: `camelCase` with `Service` suffix — `authService.ts`
- Types files: `camelCase` with `.types.ts` — `navbar.types.ts`
- Constants: `UPPER_SNAKE_CASE`
- Utility functions: `camelCase`
