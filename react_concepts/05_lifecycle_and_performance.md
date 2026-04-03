# Part 5 — Lifecycle, Error Boundaries & Performance

> **Understanding how components are born, updated, and destroyed — and how to keep them fast.**

---

## 1. Component Lifecycle — The 4 Phases

Every React component goes through these phases:

```
┌─────────────────────────────────────────────────────────┐
│  1. INITIALIZATION  →  setup props & state              │
│  2. MOUNTING        →  component added to DOM           │
│  3. UPDATING        →  state/props change               │
│  4. UNMOUNTING      →  component removed from DOM       │
└─────────────────────────────────────────────────────────┘
```

### Lifecycle Methods → Hooks Mapping

| Phase | Class Method | Hook Equivalent |
|-------|-------------|-----------------|
| Mount | `componentDidMount()` | `useEffect(() => {}, [])` |
| Update | `componentDidUpdate()` | `useEffect(() => {}, [dep])` |
| Unmount | `componentWillUnmount()` | `useEffect(() => { return cleanup }, [])` |
| Should update? | `shouldComponentUpdate()` | `React.memo()` |
| Catch errors | `componentDidCatch()` | ❌ No hook yet (class only) |

### 🛒 Real App: Product Page Lifecycle

```jsx
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);

  // MOUNT + UPDATE: fetch product data when productId changes
  useEffect(() => {
    console.log("📦 Mounting/Updating: Fetching product...");
    fetch(`/api/products/${productId}`)
      .then(res => res.json())
      .then(setProduct);

    // UNMOUNT: cleanup
    return () => {
      console.log("🗑️ Unmounting: Cleaning up...");
    };
  }, [productId]);

  // MOUNT only: track page view
  useEffect(() => {
    analytics.trackPageView("ProductPage", productId);
  }, []);

  if (!product) return <Spinner />;
  return <ProductDetails product={product} />;
}
```

### Class Lifecycle (For legacy code understanding)

```jsx
class ProductPage extends React.Component {
  constructor(props) {
    super(props);                       // 1. INITIALIZATION
    this.state = { product: null };
  }

  componentDidMount() {                 // 2. MOUNTING
    this.fetchProduct(this.props.id);
  }

  componentDidUpdate(prevProps) {       // 3. UPDATING
    if (prevProps.id !== this.props.id) {
      this.fetchProduct(this.props.id);
    }
  }

  componentWillUnmount() {              // 4. UNMOUNTING
    // Cancel subscriptions, timers
  }

  shouldComponentUpdate(nextProps, nextState) {
    // Return false to skip render
    return nextProps.id !== this.props.id;
  }

  fetchProduct(id) {
    fetch(`/api/products/${id}`)
      .then(res => res.json())
      .then(product => this.setState({ product }));
  }

  render() {
    return <ProductDetails product={this.state.product} />;
  }
}
```

---

## 2. Error Boundaries — Catch Render Errors Gracefully

Error boundaries catch errors during **rendering, lifecycle methods, and constructors** of child components.

> ⚠️ **Still class-only** — no hook equivalent exists yet!

### 🛒 Real App: Product Grid with Error Boundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error tracking service (Sentry, etc.)
    console.error("Error caught:", error, errorInfo);
    // Sentry.captureException(error);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Usage — wrap sections of your app
function App() {
  return (
    <div>
      <Header /> {/* If header crashes, rest of app still works */}

      <ErrorBoundary>
        <ProductGrid />  {/* If one product card crashes, fallback shown */}
      </ErrorBoundary>

      <ErrorBoundary>
        <ReviewSection /> {/* Reviews crash independently */}
      </ErrorBoundary>

      <Footer />
    </div>
  );
}
```

### What Error Boundaries DON'T catch

| Caught ✅ | NOT caught ❌ |
|-----------|--------------|
| Render errors | Event handler errors |
| Lifecycle errors | Async code (setTimeout, fetch) |
| Constructor errors | Server-side rendering |

```jsx
// ❌ Error boundary won't catch this — use try/catch instead
<button onClick={() => {
  try {
    riskyOperation();
  } catch (err) {
    setError(err.message);
  }
}}>
```

---

## 3. React Performance Optimization

### Strategy 1: React.memo — Skip Re-renders

```jsx
// ProductCard re-renders ONLY if props actually change
const ProductCard = React.memo(function ProductCard({ product, onAdd }) {
  console.log("Rendering:", product.name);
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <button onClick={() => onAdd(product)}>Add</button>
    </div>
  );
});

// Custom comparison function
const PriceDisplay = React.memo(
  function PriceDisplay({ price, currency }) {
    return <span>{currency}{price.toFixed(2)}</span>;
  },
  (prevProps, nextProps) => {
    // Return true = skip re-render, false = re-render
    return prevProps.price === nextProps.price;
  }
);
```

### Strategy 2: useMemo & useCallback

```jsx
function Dashboard({ data, filters }) {
  // ✅ Expensive calculation cached
  const metrics = useMemo(() => {
    return computeComplexMetrics(data, filters);
  }, [data, filters]);

  // ✅ Function reference cached for memoized children
  const handleExport = useCallback(() => {
    exportToCsv(metrics);
  }, [metrics]);

  return <DashboardPanel metrics={metrics} onExport={handleExport} />;
}
```

### Strategy 3: Lazy Loading (Code Splitting)

```jsx
import React, { Suspense, lazy } from "react";

// ✅ Component loaded ONLY when user navigates to it
const AdminDashboard = lazy(() => import("./AdminDashboard"));
const Analytics = lazy(() => import("./Analytics"));
const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<div className="loading">Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/admin" element={<AdminDashboard />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Strategy 4: State Colocation

Move state **as close as possible** to where it's used.

```jsx
// ❌ searchQuery state in App causes EVERYTHING to re-render
function App() {
  const [searchQuery, setSearchQuery] = useState("");
  return (
    <div>
      <SearchBar query={searchQuery} onChange={setSearchQuery} />
      <ProductGrid />    {/* re-renders unnecessarily! */}
      <Sidebar />        {/* re-renders unnecessarily! */}
    </div>
  );
}

// ✅ Move state into SearchBar — only SearchBar re-renders
function SearchBar() {
  const [searchQuery, setSearchQuery] = useState(""); // colocated!
  return <input value={searchQuery} onChange={e => setSearchQuery(e.target.value)} />;
}
```

### Strategy 5: Avoiding Inline Objects/Functions

```jsx
// ❌ New object on every render → child always re-renders
<UserCard style={{ marginTop: 20 }} />
<ProductList onSort={() => sortProducts()} />

// ✅ Define outside or memoize
const cardStyle = { marginTop: 20 };
const handleSort = useCallback(() => sortProducts(), []);

<UserCard style={cardStyle} />
<ProductList onSort={handleSort} />
```

---

## 4. Side Effects — With and Without Cleanup

### Effects WITHOUT Cleanup

```jsx
// Logging, analytics, document title
useEffect(() => {
  analytics.pageView("ProductPage");
}, []);

useEffect(() => {
  document.title = `${product.name} — MyStore`;
}, [product.name]);
```

### Effects WITH Cleanup

```jsx
// WebSocket, event listeners, timers, subscriptions
useEffect(() => {
  const ws = new WebSocket("wss://live-prices.com");
  ws.onmessage = (e) => updatePrice(JSON.parse(e.data));

  return () => ws.close(); // ✅ cleanup on unmount
}, []);
```

---

## 5. React Strict Mode

Wrapping your app in `<React.StrictMode>` enables extra development checks:

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

| What it does | Why |
|-------------|-----|
| Renders components twice (dev only) | Catches impure renders |
| Runs effects twice (dev only) | Catches missing cleanup |
| Warns about deprecated APIs | Keeps code future-proof |
| Warns about unsafe lifecycle methods | Prevents async bugs |

> 💡 Strict Mode has **zero impact in production**. It only runs extra checks in development.

---

## ✅ Quick Revision Checklist

- [ ] 4 phases: Initialization → Mounting → Updating → Unmounting
- [ ] `useEffect(fn, [])` = mount; `return cleanup` = unmount
- [ ] Error boundaries = class components with `getDerivedStateFromError`
- [ ] Error boundaries don't catch event handler / async errors
- [ ] `React.memo` = skip re-render when props unchanged
- [ ] Lazy loading = `React.lazy()` + `<Suspense>`
- [ ] State colocation = move state closer to where it's used
- [ ] Strict Mode = dev-only checks; renders/effects run twice
