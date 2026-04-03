# Part 3 — React Hooks Basics: useState, useEffect, useRef & Custom Hooks

> **The hooks that replaced class components and changed how we write React.**

---

## 1. useState — Adding State to Functional Components

`useState` returns a pair: the **current state value** and a **function to update it**.

```jsx
const [value, setValue] = useState(initialValue);
```

### 🛒 Real App: Product Quantity Selector

```jsx
function QuantitySelector({ maxStock }) {
  const [quantity, setQuantity] = useState(1);

  const increment = () => {
    // ✅ Functional update — always gets latest state
    setQuantity(prev => Math.min(prev + 1, maxStock));
  };

  const decrement = () => {
    setQuantity(prev => Math.max(prev - 1, 1));
  };

  return (
    <div className="qty-selector">
      <button onClick={decrement} disabled={quantity === 1}>−</button>
      <span>{quantity}</span>
      <button onClick={increment} disabled={quantity >= maxStock}>+</button>
    </div>
  );
}
```

### Common Patterns

```jsx
// Boolean toggle
const [isOpen, setIsOpen] = useState(false);
const toggle = () => setIsOpen(prev => !prev);

// Object state
const [filters, setFilters] = useState({ category: "all", minPrice: 0 });
const updateCategory = (cat) =>
  setFilters(prev => ({ ...prev, category: cat }));

// Array state
const [cart, setCart] = useState([]);
const addItem = (item) => setCart(prev => [...prev, item]);
const removeItem = (id) => setCart(prev => prev.filter(p => p.id !== id));
```

### ⚠️ Common Mistakes

```jsx
// ❌ State updates are asynchronous — can't read new value immediately
setCount(count + 1);
console.log(count); // still old value!

// ❌ Mutating state directly
const [user, setUser] = useState({ name: "Alice", age: 25 });
user.name = "Bob";    // ❌ This won't trigger re-render!
setUser(user);        // ❌ Same reference — React thinks nothing changed

// ✅ Always create a new object/array
setUser(prev => ({ ...prev, name: "Bob" }));
```

---

## 2. useEffect — Side Effects in React

`useEffect` runs code **after the component renders**. It handles things that live outside React's rendering system.

```jsx
useEffect(() => {
  // Side effect code
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

### Dependency Array Controls WHEN Effect Runs

| Dependency Array | When it runs |
|------------------|-------------|
| `useEffect(() => {})` | After **every** render |
| `useEffect(() => {}, [])` | Only on **first mount** |
| `useEffect(() => {}, [userId])` | When `userId` **changes** |

### 🛒 Real App: Fetch Products on Category Change

```jsx
function ProductListing({ category }) {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isMounted = true;   // prevent state update after unmount
    setLoading(true);

    fetch(`/api/products?category=${category}`)
      .then(res => {
        if (!res.ok) throw new Error("Failed to fetch");
        return res.json();
      })
      .then(data => {
        if (isMounted) {
          setProducts(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (isMounted) {
          setError(err.message);
          setLoading(false);
        }
      });

    // ✅ Cleanup: cancel if component unmounts or category changes
    return () => { isMounted = false; };
  }, [category]); // ← Runs again whenever category changes

  if (loading) return <Spinner />;
  if (error) return <p>Error: {error}</p>;

  return (
    <div className="grid">
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

### Type 1: Effect WITHOUT Cleanup

```jsx
// Setting document title
useEffect(() => {
  document.title = `${cartCount} items in cart`;
}, [cartCount]);
```

### Type 2: Effect WITH Cleanup

```jsx
// WebSocket subscription
useEffect(() => {
  const ws = new WebSocket("wss://notifications.example.com");

  ws.onmessage = (event) => {
    const notification = JSON.parse(event.data);
    addNotification(notification);
  };

  // ✅ Cleanup: close connection when component unmounts
  return () => ws.close();
}, []);

// Event listener
useEffect(() => {
  const handleResize = () => setWidth(window.innerWidth);
  window.addEventListener("resize", handleResize);

  return () => window.removeEventListener("resize", handleResize);
}, []);

// Timer
useEffect(() => {
  const timer = setInterval(() => {
    setSeconds(prev => prev + 1);
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

### Cancelling Fetch with AbortController

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then(res => res.json())
    .then(data => setResults(data))
    .catch(err => {
      if (err.name !== "AbortError") {
        setError(err.message);
      }
    });

  return () => controller.abort(); // ✅ Cancel request on new query
}, [query]);
```

---

## 3. useRef — Accessing DOM & Persisting Values

`useRef` creates a mutable reference that:
- **Persists across renders** (like state)
- **Does NOT trigger re-render** when updated (unlike state)

### Use Case 1: Accessing DOM Elements

```jsx
function SearchBar() {
  const inputRef = useRef(null);

  // Auto-focus search input on page load
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" placeholder="Search products..." />;
}
```

### Use Case 2: Storing Previous Value

```jsx
function PriceTracker({ currentPrice }) {
  const prevPriceRef = useRef(currentPrice);

  useEffect(() => {
    prevPriceRef.current = currentPrice;
  }, [currentPrice]);

  const previousPrice = prevPriceRef.current;
  const trend = currentPrice > previousPrice ? "📈" : "📉";

  return (
    <p>
      ₹{currentPrice} {trend} (was ₹{previousPrice})
    </p>
  );
}
```

### Use Case 3: Storing Timer/Interval IDs

```jsx
function Countdown() {
  const [seconds, setSeconds] = useState(60);
  const intervalRef = useRef(null);

  const start = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => {
        if (prev <= 1) {
          clearInterval(intervalRef.current);
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
  };

  const stop = () => clearInterval(intervalRef.current);

  useEffect(() => {
    return () => clearInterval(intervalRef.current); // cleanup
  }, []);

  return (
    <div>
      <p>OTP expires in: {seconds}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Pause</button>
    </div>
  );
}
```

### useState vs useRef

| Feature | `useState` | `useRef` |
|---------|-----------|---------|
| Triggers re-render? | ✅ Yes | ❌ No |
| Persists between renders? | ✅ Yes | ✅ Yes |
| Use for | UI-affecting data | DOM access, timers, previous values |

---

## 4. Custom Hooks — Reusable Logic

A custom hook is a **JavaScript function starting with `use`** that groups reusable hook logic.

### 🛒 Real App: useFetch — Reusable Data Fetching

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== "AbortError") {
          setError(err.message);
          setLoading(false);
        }
      });

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage — any component can now fetch data!
function ProductPage({ id }) {
  const { data: product, loading, error } = useFetch(`/api/products/${id}`);

  if (loading) return <Spinner />;
  if (error) return <p>Error: {error}</p>;
  return <ProductDetails product={product} />;
}

function UserProfile({ userId }) {
  const { data: user, loading } = useFetch(`/api/users/${userId}`);
  if (loading) return <Spinner />;
  return <h1>{user.name}</h1>;
}
```

### More Custom Hook Examples

```jsx
// useLocalStorage — persist state across page reloads
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
const [theme, setTheme] = useLocalStorage("theme", "dark");

// useDebounce — debounce a value (search input)
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchBar() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      searchProducts(debouncedQuery);
    }
  }, [debouncedQuery]); // only fires 300ms after user stops typing
}
```

---

## 5. Rules of Hooks

| Rule | Why |
|------|-----|
| Call hooks **at the top level** only | React tracks hooks by ORDER of calls |
| Don't call inside loops, conditions, nesting | Breaks the order → bugs |
| Only call from **React functions** or **custom hooks** | Regular JS functions can't use hooks |

```jsx
// ❌ Hook inside condition — breaks order!
if (isLoggedIn) {
  const [user, setUser] = useState(null); // BAD!
}

// ✅ Condition INSIDE the hook
const [user, setUser] = useState(null);
useEffect(() => {
  if (isLoggedIn) {
    fetchUser().then(setUser);
  }
}, [isLoggedIn]);
```

---

## ✅ Quick Revision Checklist

- [ ] `useState` — state variable + setter; triggers re-render
- [ ] Always use functional updates: `setX(prev => ...)`
- [ ] `useEffect(fn, [])` — runs once; `[dep]` — runs when dep changes
- [ ] Always clean up: timers, subscriptions, event listeners
- [ ] `useRef` — persists value without re-render; DOM access
- [ ] Custom hooks = reusable logic (useFetch, useDebounce, useLocalStorage)
- [ ] Never call hooks inside loops, conditions, or non-React functions
