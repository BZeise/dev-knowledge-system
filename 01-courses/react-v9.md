# Complete Intro to React, v9

*Course by Brian Holt (Frontend Masters)*

## Course Resources

- [Course Website](https://react-v9.holt.courses/)
- [My Project Repository](https://github.com/BZeise/padre-ginos)
- [ESLint Setup Guide](https://react-v9.holt.courses/lessons/tools/linting)
- [Vite Configuration](https://react-v9.holt.courses/lessons/tools/vite) — *Want to incorporate into personal app template*

## Areas to Revisit

- [useEffect / Portals](https://react-v9.holt.courses/lessons/advanced-react/portals) — Still fuzzy on cleanup functions
- [Error Boundaries](https://react-v9.holt.courses/lessons/advanced-react/error-boundaries)
- [Preload and Preconnect](https://www.debugbear.com/blog/resource-hints-rel-preload-prefetch-preconnect) — Resource hints

---

## React Core Concepts

### Components

A React component is a function that returns renderable JSX. **Component names must start with a capital letter** — this is how React distinguishes them from native HTML elements.

```javascript
// Component — capital letter
function PizzaCard({ name, price }) {
  return <div>{name} — ${price}</div>;
}

// Not a component — camelCase for utilities
function formatPrice(cents) {
  return (cents / 100).toFixed(2);
}
```

Return multiple top-level elements with Fragments:
```javascript
<>
  <Header />
  <Main />
  <Footer />
</>
```

### Props

Props are **read-only data passed from parent to child**. A child cannot modify its own props.

Spread props to pass all through to a nested child:
```javascript
function Wrapper(props) {
  return <Inner {...props} />;
}
```

### State and useState

`useState` declares a piece of state local to a component. Returns the current value and a setter function. When the setter is called, React re-renders the component.

```javascript
const [count, setCount] = useState(0);

// Never mutate state directly
setCount(count + 1);

// For state derived from previous state, use functional form
setCount(prev => prev + 1);
```

### The Rules of Hooks

Hooks depend on being called in the same order on every render:

1. **Never call hooks inside loops, conditions, or nested functions**. Call them only at the top level.
2. **Only call hooks from React function components or custom hooks**, not regular JavaScript functions.

### useEffect

For side effects — anything reaching outside the component:
- Fetching data
- Subscribing to events
- Manipulating the DOM
- Setting timers

```javascript
// Runs after every render
useEffect(() => {
  console.log("Component rendered");
});

// Runs only once on mount
useEffect(() => {
  fetchData();
}, []);

// Runs when userId changes
useEffect(() => {
  fetchUser(userId);
}, [userId]);

// Cleanup function
useEffect(() => {
  const subscription = someAPI.subscribe();
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

**Important:** Don't make the useEffect callback itself async. Define an async function inside and call it immediately:

```javascript
useEffect(() => {
  async function loadData() {
    const data = await fetchSomething();
    setData(data);
  }
  loadData();
}, []);
```

### useRef

`useRef` gives you a mutable container (`.current`) that persists across renders without causing re-renders when changed.

Common uses:
- Holding a reference to a DOM node
- Storing a value you want to remember across renders

```javascript
const inputRef = useRef(null);

function focus() {
  inputRef.current.focus();
}

return <input ref={inputRef} />;
```

### useContext

Passes data through the component tree without threading props at every level. Useful for:
- Current user
- Theme
- Locale

```javascript
const ThemeContext = React.createContext('light');

// Provider
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

// Consumer
const theme = useContext(ThemeContext);
```

### useQuery (TanStack Query)

Manages server state — fetching, caching, and syncing data from a remote source.

```javascript
const { data, isLoading } = useQuery({
  queryKey: ['order', orderId],
  queryFn: () => fetchOrder(orderId),
  enabled: !!orderId,  // Don't run until orderId is defined
});
```

### Closing a Modal

Pass `undefined` (or `null`) to the state variable controlling modal visibility:

```javascript
setFocusedOrder(undefined);  // hides the modal
```

### Error Boundaries

Error boundaries are **class components** that catch JavaScript errors in their child tree and display a fallback UI. Cannot be written as function components.

To wrap a function component:

```javascript
function ErrorBoundaryWrappedPastOrderRoutes(props) {
  return (
    <ErrorBoundary>
      <PastOrdersRoute {...props} />
    </ErrorBoundary>
  );
}
```

---

## Testing with Vitest

Vitest mimics Jest's API (which itself mimicked Jasmine), so Jest knowledge transfers directly.

Designed for Vite projects and significantly faster than Jest for those setups.

---

## ESLint Setup

```javascript
/** @type {import('eslint').Linter.Config[]} */
```

This line removes the error requiring React import when it's no longer needed (React 17+):

```javascript
reactPlugin.configs.flat["jsx-runtime"],
```

---

## Resource Hints: Preload and Preconnect

Tell the browser about assets it'll need soon:

- **rel="preload"** — Fetch this resource now; I'll use it soon on this page
- **rel="preconnect"** — Establish a connection to this origin early (DNS + TCP + TLS)
- **rel="prefetch"** — Fetch this for likely future navigation (lower priority)

[Detailed Article](https://www.debugbear.com/blog/resource-hints-rel-preload-prefetch-preconnect)

---

## Deployment Notes

- `npm run build` generates the `dist/` folder — static assets ready for a CDN
- The API (server.js) must be deployed separately
- Currently on SQLite; switching to PostgreSQL requires:
  - Adding a connection string
  - Swapping the DB driver
  - Moving static files (styles, images) into the project

---

## Recommended Follow-Up Courses

- Intermediate React with Brian Holt
- React with TypeScript with Steve Kinney

---

*Course: Frontend Masters | Instructor: Brian Holt*
