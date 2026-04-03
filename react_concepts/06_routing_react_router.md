# Part 6 — React Router v6: Routing, Navigation & Protected Routes

> **How single-page apps handle navigation without page reloads.**

---

## 1. What is React Router?

React Router lets you navigate between pages in a **single-page app** (SPA) without full page reloads.

```
URL changes → React Router matches a route → renders the correct component
No server request. No page reload. Instant navigation.
```

### 🛒 Real App: E-commerce URL structure

```
mystore.com/                    → Home
mystore.com/products            → Product listing
mystore.com/products/nike-42    → Product detail page
mystore.com/cart                → Shopping cart
mystore.com/checkout            → Checkout
mystore.com/account/orders      → Order history (protected)
mystore.com/admin/dashboard     → Admin panel (role-protected)
```

---

## 2. React Router v5 vs v6 Changes

| Feature | v5 | v6 |
|---------|----|----|
| Wrapper | `<Switch>` | `<Routes>` |
| Rendering | `component={X}` or `render={}` | `element={<X />}` |
| Exact match | `exact` prop needed | **Default** — all routes exact |
| Nested routes | Manual | Built-in with `<Outlet />` |
| Redirect | `<Redirect />` | `<Navigate />` |
| Programmatic nav | `useHistory()` | `useNavigate()` |

```jsx
// v5
<Switch>
  <Route exact path="/" component={Home} />
  <Route path="/about" component={About} />
</Switch>

// v6
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

---

## 3. Basic Routing Setup

```jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/products">Products</Link>
        <Link to="/cart">Cart</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<ProductListing />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="/cart" element={<Cart />} />
        <Route path="*" element={<NotFound />} />  {/* 404 */}
      </Routes>
    </BrowserRouter>
  );
}
```

### Reading URL Parameters

```jsx
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { id } = useParams();   // /products/nike-42 → id = "nike-42"
  const { data: product, loading } = useFetch(`/api/products/${id}`);

  if (loading) return <Spinner />;
  return <h1>{product.name}</h1>;
}
```

---

## 4. Nested Routes + Outlet

Parent routes render **shared layout**, child routes render inside `<Outlet />`.

### 🛒 Real App: Account Section with Sidebar

```jsx
// Route config
<Routes>
  <Route path="/account" element={<AccountLayout />}>
    <Route index element={<AccountOverview />} />      {/* /account */}
    <Route path="orders" element={<OrderHistory />} />  {/* /account/orders */}
    <Route path="profile" element={<EditProfile />} />  {/* /account/profile */}
    <Route path="addresses" element={<Addresses />} />  {/* /account/addresses */}
  </Route>
</Routes>

// AccountLayout — shared sidebar + header stays consistent
function AccountLayout() {
  return (
    <div className="account-page">
      <aside className="sidebar">
        <NavLink to="/account">Overview</NavLink>
        <NavLink to="/account/orders">Orders</NavLink>
        <NavLink to="/account/profile">Profile</NavLink>
        <NavLink to="/account/addresses">Addresses</NavLink>
      </aside>
      <main>
        <Outlet />   {/* ← child route renders here! */}
      </main>
    </div>
  );
}
```

> Clicking "Orders" only changes the `<Outlet />` content — sidebar stays put.

---

## 5. Protected / Auth Routes

### 🔐 Real App: Login Required for Checkout & Account

```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();
  const location = useLocation();

  if (loading) return <Spinner />;

  if (!isAuthenticated) {
    // Redirect to login, but remember where user wanted to go
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  return children;
}

// Role-based protection
function AdminRoute({ children }) {
  const { user } = useAuth();
  if (user?.role !== "admin") {
    return <Navigate to="/" replace />;
  }
  return children;
}

// Usage in routes
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/products" element={<Products />} />

  {/* Protected — login required */}
  <Route path="/cart" element={
    <ProtectedRoute><Cart /></ProtectedRoute>
  } />
  <Route path="/checkout" element={
    <ProtectedRoute><Checkout /></ProtectedRoute>
  } />

  {/* Admin only */}
  <Route path="/admin/*" element={
    <AdminRoute><AdminDashboard /></AdminRoute>
  } />
</Routes>
```

### Redirect After Login

```jsx
function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();

  const handleLogin = async (credentials) => {
    await authService.login(credentials);

    // Go back to where user came from, or home
    const redirectTo = location.state?.from || "/";
    navigate(redirectTo, { replace: true });
  };

  return <LoginForm onSubmit={handleLogin} />;
}
```

---

## 6. Programmatic Navigation

### `useNavigate` — Navigate from code

```jsx
function CheckoutButton({ cartTotal }) {
  const navigate = useNavigate();

  const handleCheckout = () => {
    if (cartTotal === 0) {
      navigate("/products");  // redirect to shop
    } else {
      navigate("/checkout", {
        state: { total: cartTotal }  // pass hidden data
      });
    }
  };

  return <button onClick={handleCheckout}>Proceed to Checkout</button>;
}
```

### `<Navigate />` — Redirect in JSX

```jsx
function OldProductPage() {
  // Permanent redirect from old URL to new URL
  return <Navigate to="/products" replace />;
}
```

---

## 7. Params, Query Strings & Navigation State

### Route Params — required, in URL path

```jsx
// Route: /products/:id
const { id } = useParams(); // "nike-42"
```

### Query Strings — optional, after ?

```jsx
// URL: /products?category=shoes&sort=price
import { useSearchParams } from "react-router-dom";

function ProductFilters() {
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get("category") || "all";
  const sort = searchParams.get("sort") || "relevance";

  const updateFilter = (key, value) => {
    setSearchParams(prev => {
      prev.set(key, value);
      return prev;
    });
  };

  return (
    <div>
      <select value={category} onChange={e => updateFilter("category", e.target.value)}>
        <option value="all">All</option>
        <option value="shoes">Shoes</option>
        <option value="clothing">Clothing</option>
      </select>
    </div>
  );
}
```

### Navigation State — hidden data

```jsx
// Send
navigate("/order-confirmation", { state: { orderId: "ORD-123" } });

// Receive
const location = useLocation();
const orderId = location.state?.orderId; // "ORD-123"
// ⚠️ Lost on page refresh!
```

---

## 8. BrowserRouter vs HashRouter

| Feature | `BrowserRouter` | `HashRouter` |
|---------|----------------|-------------|
| URL format | `mystore.com/products` | `mystore.com/#/products` |
| Requires server config | ✅ Yes | ❌ No |
| SEO friendly | ✅ Yes | ❌ No |
| Use case | Modern apps with server control | Static hosting (GitHub Pages) |

---

## 9. Lazy Loading Routes

```jsx
const ProductListing = lazy(() => import("./pages/ProductListing"));
const Checkout = lazy(() => import("./pages/Checkout"));
const AdminDashboard = lazy(() => import("./pages/AdminDashboard"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<FullPageSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/products" element={<ProductListing />} />
          <Route path="/checkout" element={<Checkout />} />
          <Route path="/admin/*" element={<AdminDashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

---

## 10. Scroll to Top on Route Change

```jsx
function ScrollToTop() {
  const { pathname } = useLocation();

  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);

  return null;
}

// Add inside BrowserRouter
<BrowserRouter>
  <ScrollToTop />
  <Routes>...</Routes>
</BrowserRouter>
```

---

## 11. Prevent Navigation — Unsaved Form Warning

```jsx
function EditProfilePage() {
  const [isDirty, setIsDirty] = useState(false);

  const blocker = useBlocker(
    ({ currentLocation, nextLocation }) =>
      isDirty && currentLocation.pathname !== nextLocation.pathname
  );

  useEffect(() => {
    if (blocker.state === "blocked") {
      const leave = window.confirm("You have unsaved changes. Leave anyway?");
      if (leave) blocker.proceed();
      else blocker.reset();
    }
  }, [blocker]);

  return (
    <form onChange={() => setIsDirty(true)}>
      <input name="name" />
      <input name="email" />
      <button type="submit">Save</button>
    </form>
  );
}
```

---

## ✅ Quick Revision Checklist

- [ ] v6: `<Routes>` + `element={}` (not `<Switch>` + `component={}`)
- [ ] `useParams()` for `/products/:id`; `useSearchParams()` for `?key=value`
- [ ] Nested routes use `<Outlet />` as placeholder for child content
- [ ] Protected routes: wrapper component + `<Navigate to="/login" />`
- [ ] `useNavigate()` for programmatic nav; `<Navigate />` for declarative redirect
- [ ] Navigation state is hidden from URL, lost on refresh
- [ ] Lazy load routes with `React.lazy()` + `<Suspense>`
- [ ] Scroll-to-top: `useLocation()` + `useEffect` + `window.scrollTo(0,0)`
