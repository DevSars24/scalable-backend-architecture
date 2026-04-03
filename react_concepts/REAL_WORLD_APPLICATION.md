# 🛒 Real-World Application: Building a Scalable E-commerce Store

> This guide brings every React concept together to build a **Feature-Rich E-commerce Dashboard**. 

---

## 🏗️ The Application Architecture

Our app, **"ShopifyLite"**, is divided into these layers:

1. **Routing**: Managed by `React Router v6`.
2. **Global State**: Managed by `Context API` (Auth, Cart).
3. **Data Fetching**: Custom Hook `useFetch` (Products, Orders).
4. **UI Components**: Atoms (Button, Input), Molecules (Card), and Organisms (Grid, Navbar).

---

## 🏙️ Concept → Implementation Map

| Concepts | Application in ShopifyLite |
|----------|---------------------------|
| **Functional Components** | Used for all UI elements (Header, Footer, ProductCard). |
| **Props & Destructuring** | Passing `product` data from `ProductGrid` to `ProductCard`. |
| **useState** | Local UI states (Search query, Filter toggles, Modal open/close). |
| **useEffect** | Fetching product list API, setting document title (`Cart (3)`). |
| **useContext** | Storing `currentUser` and `cartItems` globally. |
| **useReducer** | Complex Cart logic (Add, Remove, Clear, Discount calculations). |
| **useCallback** | Preventing `ProductCard` from re-rendering when parent state changes. |
| **React.memo** | Memoizing 100+ product cards in a massive grid. |
| **Error Boundaries** | Preventing a broken "Recently Viewed" sidebar from crashing the whole site. |
| **React Router v6** | Handling `/home`, `/cart`, and protected `/admin` pages. |

---

## 🧑‍💻 The Implementation Code Snippets

### 1. Global Authentication (Context API)
We store user info at the root so the Navbar and Account page can access it instantly.

```javascript
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (data) => setUser(data);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

### 2. Complex Cart Logic (useReducer)
Instead of 10 different `useStates`, we use one `useReducer` to handle all cart mutations.

```javascript
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE':
      return { ...state, items: state.items.filter(i => i.id !== action.id) };
    case 'CLEAR':
      return { items: [] };
    default:
      return state;
  }
}
```

---

### 3. Creating a Protected Route (HOC/Wrapper)
A "Private Route" that redirects non-logged-in users to `/login`.

```javascript
function AuthGuard({ children }) {
  const { user } = useContext(AuthContext);
  const location = useLocation();

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// In App Router:
<Route path="/checkout" element={<AuthGuard><Checkout /></AuthGuard>} />
```

---

### 4. Performance & Batching (Large Product List)
We use `React.memo` and `useCallback` to keep our product feed buttery smooth, even with 500+ items.

```javascript
const ProductCard = React.memo(({ product, onAddToCart }) => {
  console.log(`Rendering ${product.name}`);
  return (
    <div className="card">
      <img src={product.img} alt={product.name} />
      <button onClick={() => onAddToCart(product)}>Add</button>
    </div>
  );
});

// usage in parent
const handleAdd = useCallback((p) => dispatch({ type: 'ADD', payload: p }), []);
```

---

### 5. Custom Hook for API calls (useFetch)
A reusable logic piece to handle loading, error, and data fetching for any API endpoint.

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return { data, loading };
}
```

---

## 🔥 Scenario-Based Debugging

**SCENARIO:** The user is typing in a Search Bar. Every letter they type causes the **entire** product list to flicker and lag.

**FIX:**
1. **Debouncing**: Don't trigger the API call on every keystroke. Use a `useEffect` with `setTimeout` to wait 500ms after the last key is pressed.
2. **Memoization**: Wrap the heavy product grid in `React.memo` and the search filter logic in `useMemo`.

**SCENARIO:** The "Recently Viewed Products" API fails. The whole website shows a "White Screen of Death."

**FIX:**
1. **Error Boundaries**: Wrap the "Recently Viewed" component in an `ErrorBoundary` class. If it fails, only that component disappears and shows "Could not load products," but the rest of the store (Cart, Navbar) still functions.

---

## 🏁 Summary Checklist for a Production React App

- [ ] Split code using `React.lazy()` for different pages.
- [ ] Use `Context` only for state that is shared by **MANY** components (Auth, Theme).
- [ ] Use `useReducer` for complex state transitions.
- [ ] Implement `ErrorBoundary` for 3rd-party widgets or unreliable components.
- [ ] Always provide a `cleanup` function in `useEffect` (Abort controllers, clearing timers).
- [ ] Use `useCallback/useMemo` to keep reference types stable for child components.

**This is how professional companies build apps in 2026!**
