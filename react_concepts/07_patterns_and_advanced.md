# Part 7 — Patterns: HOC, Context API, Conditional Rendering & More

> **Design patterns that keep large React codebases clean and scalable.**

---

## 1. Higher Order Components (HOC)

A HOC is a **function that takes a component and returns a new enhanced component**.

```
HOC(Component) → EnhancedComponent
```

### 🛒 Real App: withAuth — Add authentication check to any page

```jsx
// HOC: wraps any component with auth checking
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { user, loading } = useAuth();

    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage — any page becomes "login-required" by wrapping
const ProtectedCheckout = withAuth(CheckoutPage);
const ProtectedAccount = withAuth(AccountPage);
const ProtectedOrders = withAuth(OrderHistory);

// In routes
<Route path="/checkout" element={<ProtectedCheckout />} />
<Route path="/account" element={<ProtectedAccount />} />
```

### 🛒 Real App: withLoader — Add loading state to any data component

```jsx
function withLoader(WrappedComponent, fetchFn) {
  return function LoaderComponent(props) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
      fetchFn(props)
        .then(setData)
        .catch(err => setError(err.message))
        .finally(() => setLoading(false));
    }, []);

    if (loading) return <Spinner />;
    if (error) return <ErrorMessage message={error} />;

    return <WrappedComponent {...props} data={data} />;
  };
}

// Wrap any component with automatic data loading
const ProductListWithLoader = withLoader(
  ProductList,
  () => fetch("/api/products").then(r => r.json())
);
```

### HOC Rules

```
✅ Don't mutate the original component
✅ Pass through all props with {...props}
✅ Name should start with "with" (withAuth, withLoader, withTheme)
❌ Don't use HOC inside render — creates new component every render
```

> 💡 **Modern alternative**: Custom Hooks often replace HOCs for logic reuse.

---

## 2. Context API — Avoid Prop Drilling

Context lets you pass data through the component tree **without passing props manually** at every level.

### 🛒 Real App: Shopping Cart Context

```jsx
// 1. CREATE CONTEXT
const CartContext = React.createContext();

// 2. PROVIDER — wraps the app, provides data
function CartProvider({ children }) {
  const [items, setItems] = useState([]);

  const addItem = (product) => {
    setItems(prev => {
      const exists = prev.find(i => i.id === product.id);
      if (exists) {
        return prev.map(i =>
          i.id === product.id ? { ...i, qty: i.qty + 1 } : i
        );
      }
      return [...prev, { ...product, qty: 1 }];
    });
  };

  const removeItem = (id) => {
    setItems(prev => prev.filter(i => i.id !== id));
  };

  const total = items.reduce((sum, i) => sum + i.price * i.qty, 0);
  const itemCount = items.reduce((sum, i) => sum + i.qty, 0);

  return (
    <CartContext.Provider value={{ items, addItem, removeItem, total, itemCount }}>
      {children}
    </CartContext.Provider>
  );
}

// 3. CUSTOM HOOK — clean way to consume context
function useCart() {
  const context = React.useContext(CartContext);
  if (!context) throw new Error("useCart must be used within CartProvider");
  return context;
}

// 4. WRAP APP
function App() {
  return (
    <CartProvider>
      <Header />
      <Routes>...</Routes>
      <Footer />
    </CartProvider>
  );
}

// 5. USE ANYWHERE — no prop drilling!
function Header() {
  const { itemCount } = useCart();
  return <nav>🛒 Cart ({itemCount})</nav>;
}

function ProductCard({ product }) {
  const { addItem } = useCart();
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => addItem(product)}>Add to Cart</button>
    </div>
  );
}

function CartSidebar() {
  const { items, removeItem, total } = useCart();
  return (
    <aside>
      {items.map(i => (
        <div key={i.id}>
          {i.name} x{i.qty} — ₹{i.price * i.qty}
          <button onClick={() => removeItem(i.id)}>×</button>
        </div>
      ))}
      <strong>Total: ₹{total}</strong>
    </aside>
  );
}
```

### 🔐 Real App: Auth Context

```jsx
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user is already logged in (token in cookie)
    authService.getCurrentUser()
      .then(setUser)
      .catch(() => setUser(null))
      .finally(() => setLoading(false));
  }, []);

  const login = async (email, password) => {
    const user = await authService.login(email, password);
    setUser(user);
  };

  const logout = async () => {
    await authService.logout();
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return React.useContext(AuthContext);
}
```

### Multiple Contexts — Compose Providers

```jsx
function App() {
  return (
    <AuthProvider>
      <CartProvider>
        <ThemeProvider>
          <NotificationProvider>
            <Router>
              <AppRoutes />
            </Router>
          </NotificationProvider>
        </ThemeProvider>
      </CartProvider>
    </AuthProvider>
  );
}
```

### When to use Context vs Redux vs Zustand

| Scenario | Solution |
|----------|---------|
| Theme, language, auth (rarely changes) | ✅ Context |
| Cart, filters (changes often, few consumers) | ✅ Context |
| Large app, 30+ components consuming same state | Redux / Zustand |
| Complex state transitions (undo/redo, middleware) | Redux |
| Simple global state, minimal boilerplate | Zustand / Jotai |

---

## 3. Conditional Rendering

### Pattern 1: `if/else` (for full component swap)

```jsx
function OrderStatus({ order }) {
  if (!order) return <p>No order found</p>;

  if (order.status === "delivered") {
    return <DeliveredBanner order={order} />;
  }
  if (order.status === "shipped") {
    return <TrackingInfo order={order} />;
  }
  return <ProcessingMessage order={order} />;
}
```

### Pattern 2: Ternary `? :` (for inline toggle)

```jsx
function ProductCard({ product }) {
  return (
    <div className="card">
      <h3>{product.name}</h3>
      {product.inStock ? (
        <button className="btn-primary">Add to Cart</button>
      ) : (
        <button className="btn-disabled" disabled>Out of Stock</button>
      )}
    </div>
  );
}
```

### Pattern 3: `&&` (show or hide)

```jsx
function Navbar({ user }) {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/products">Products</Link>

      {user && <Link to="/account">My Account</Link>}
      {user?.role === "admin" && <Link to="/admin">Admin Panel</Link>}

      {!user && <Link to="/login">Login</Link>}
    </nav>
  );
}
```

> ⚠️ Gotcha with `&&`: `{count && <p>Items: {count}</p>}` — if `count` is `0`, it renders `0` on screen! Use `{count > 0 && ...}` instead.

### Pattern 4: Object Mapping (clean multi-way switch)

```jsx
const STATUS_COMPONENTS = {
  pending: <Badge color="yellow">Pending</Badge>,
  processing: <Badge color="blue">Processing</Badge>,
  shipped: <Badge color="purple">Shipped</Badge>,
  delivered: <Badge color="green">Delivered</Badge>,
  cancelled: <Badge color="red">Cancelled</Badge>,
};

function OrderStatus({ status }) {
  return STATUS_COMPONENTS[status] || <Badge color="gray">Unknown</Badge>;
}
```

---

## 4. Data Passing Between Components

### Parent → Child: Props

```jsx
<ProductCard name="Nike Air" price={8999} />
```

### Child → Parent: Callback Props

```jsx
// Parent provides callback
function ProductPage() {
  const handleAddToCart = (product) => {
    console.log("Added:", product.name);
  };
  return <ProductCard onAddToCart={handleAddToCart} />;
}

// Child calls the callback
function ProductCard({ onAddToCart }) {
  const product = { name: "Nike Air", price: 8999 };
  return <button onClick={() => onAddToCart(product)}>Add</button>;
}
```

### Sibling → Sibling: Lift State Up

```jsx
// Parent holds shared state, passes down to both children
function ProductPage() {
  const [selectedSize, setSelectedSize] = useState(null);

  return (
    <div>
      <SizeSelector sizes={["S","M","L"]} onSelect={setSelectedSize} />
      <AddToCartButton size={selectedSize} />
    </div>
  );
}
```

### Any → Any: Context or State Management

```jsx
// See Context API section above
const { addItem } = useCart(); // any component, any depth
```

---

## 5. Styling React Components

### Method 1: CSS Stylesheets

```jsx
import "./ProductCard.css";

function ProductCard() {
  return <div className="product-card">...</div>;
}
```

### Method 2: CSS Modules (scoped — no class name collisions)

```css
/* ProductCard.module.css */
.card { border: 1px solid #eee; padding: 16px; }
.title { font-size: 18px; font-weight: bold; }
```

```jsx
import styles from "./ProductCard.module.css";

function ProductCard() {
  return (
    <div className={styles.card}>
      <h3 className={styles.title}>Nike Air Max</h3>
    </div>
  );
}
```

### Method 3: Inline Styles (use sparingly)

```jsx
<div style={{ padding: "16px", backgroundColor: "#f5f5f5" }}>
```

### Method 4: Conditional Classes

```jsx
<button className={`btn ${isActive ? "btn-primary" : "btn-secondary"}`}>
  {isActive ? "Active" : "Inactive"}
</button>
```

---

## ✅ Quick Revision Checklist

- [ ] HOC = function that wraps a component to add behavior (withAuth, withLoader)
- [ ] Context = Provider + useContext — skip prop drilling
- [ ] Always create a custom hook for context (`useCart`, `useAuth`)
- [ ] Conditional: `if/else` for logic, ternary for inline, `&&` for show/hide
- [ ] ⚠️ `{0 && <Component />}` renders `0` — use `{count > 0 && ...}`
- [ ] Child → Parent: callback props
- [ ] Sibling → Sibling: lift state up to shared parent
- [ ] CSS Modules = scoped class names, no collisions
