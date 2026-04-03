# Part 4 — Advanced Hooks: useMemo, useCallback, useReducer & More

> **Performance optimization hooks and complex state management.**

---

## 1. useMemo — Cache Expensive Calculations

`useMemo` memoizes a **computed value**. The computation only re-runs when its dependencies change.

```jsx
const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
```

### 🛒 Real App: Filter & Sort 10,000 Products

```jsx
function ProductListing({ products, filters, sortBy }) {
  // ✅ Without useMemo: this runs on EVERY render (even when typing in search)
  // ✅ With useMemo: only runs when products, filters, or sortBy change
  const displayProducts = useMemo(() => {
    console.log("Computing filtered + sorted list...");

    let result = products
      .filter(p => filters.category === "all" || p.category === filters.category)
      .filter(p => p.price >= filters.minPrice && p.price <= filters.maxPrice)
      .filter(p => p.rating >= filters.minRating);

    if (sortBy === "price-asc") result.sort((a, b) => a.price - b.price);
    if (sortBy === "price-desc") result.sort((a, b) => b.price - a.price);
    if (sortBy === "rating") result.sort((a, b) => b.rating - a.rating);

    return result;
  }, [products, filters, sortBy]);

  return (
    <div>
      <p>{displayProducts.length} products found</p>
      {displayProducts.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

### When NOT to use useMemo

```jsx
// ❌ Overkill — simple math doesn't need memoization
const total = useMemo(() => price * quantity, [price, quantity]);

// ✅ Just compute it directly
const total = price * quantity;
```

> **Rule of thumb**: Only use `useMemo` when you can **measure** a performance problem.

---

## 2. useCallback — Cache Function References

`useCallback` memoizes a **function** so it doesn't get recreated on every render. Critical when passing callbacks to memoized child components.

```jsx
const memoizedFn = useCallback(() => { doSomething(a) }, [a]);
```

### 🛒 Real App: Product Card with Memoized Add-to-Cart

```jsx
// Parent
function ProductGrid({ products }) {
  const [cart, setCart] = useState([]);

  // ✅ Without useCallback: new function every render → all cards re-render
  // ✅ With useCallback: same reference → memoized cards skip re-render
  const handleAddToCart = useCallback((product) => {
    setCart(prev => [...prev, product]);
  }, []);  // no deps — setCart is stable

  const handleRemoveFromCart = useCallback((productId) => {
    setCart(prev => prev.filter(p => p.id !== productId));
  }, []);

  return (
    <div className="grid">
      {products.map(p => (
        <MemoizedProductCard
          key={p.id}
          product={p}
          onAdd={handleAddToCart}
          onRemove={handleRemoveFromCart}
        />
      ))}
    </div>
  );
}

// Child — wrapped in React.memo to skip re-renders when props don't change
const MemoizedProductCard = React.memo(function ProductCard({ product, onAdd, onRemove }) {
  console.log("Rendering:", product.name); // only logs when this product changes
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>₹{product.price}</p>
      <button onClick={() => onAdd(product)}>Add</button>
    </div>
  );
});
```

### useMemo vs useCallback

| Hook | Caches | Returns |
|------|--------|---------|
| `useMemo(() => value, deps)` | Computed **value** | The cached value |
| `useCallback(fn, deps)` | Function **reference** | The cached function |

```jsx
// These are equivalent:
useCallback(fn, deps)
useMemo(() => fn, deps)
```

---

## 3. useReducer — Complex State Logic

`useReducer` is an alternative to `useState` for **complex state** with multiple related values or logic-heavy updates.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### 🛒 Real App: Shopping Cart with useReducer

```jsx
const initialCartState = {
  items: [],
  total: 0,
  itemCount: 0,
  coupon: null,
  discount: 0,
};

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const exists = state.items.find(i => i.id === action.payload.id);
      let newItems;
      if (exists) {
        newItems = state.items.map(i =>
          i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i
        );
      } else {
        newItems = [...state.items, { ...action.payload, qty: 1 }];
      }
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, i) => sum + i.price * i.qty, 0),
        itemCount: newItems.reduce((sum, i) => sum + i.qty, 0),
      };
    }
    case "REMOVE_ITEM": {
      const newItems = state.items.filter(i => i.id !== action.payload);
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, i) => sum + i.price * i.qty, 0),
        itemCount: newItems.reduce((sum, i) => sum + i.qty, 0),
      };
    }
    case "APPLY_COUPON":
      return { ...state, coupon: action.payload, discount: 10 };
    case "CLEAR_CART":
      return initialCartState;
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// In the component
function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, initialCartState);

  return (
    <div>
      <p>Items: {cart.itemCount} | Total: ₹{cart.total - cart.discount}</p>
      {cart.items.map(item => (
        <div key={item.id}>
          <span>{item.name} x{item.qty}</span>
          <button onClick={() => dispatch({ type: "REMOVE_ITEM", payload: item.id })}>
            Remove
          </button>
        </div>
      ))}
      <button onClick={() => dispatch({ type: "CLEAR_CART" })}>
        Clear Cart
      </button>
    </div>
  );
}
```

### useState vs useReducer

| Use | useState | useReducer |
|-----|---------|-----------|
| Simple toggle, counter | ✅ Best | Overkill |
| Multiple related state values | Messy | ✅ Clean |
| Complex update logic | Scattered | ✅ Centralized |
| Next state depends on previous | Works | ✅ Predictable |
| Testing state transitions | Harder | ✅ Easy (pure function) |

---

## 4. useImperativeHandle — Custom Ref API

Exposes specific methods from a child component to a parent via `ref`.

### 🛒 Real App: Reusable Modal with Custom API

```jsx
const Modal = React.forwardRef((props, ref) => {
  const [isOpen, setIsOpen] = useState(false);

  useImperativeHandle(ref, () => ({
    open() { setIsOpen(true); },
    close() { setIsOpen(false); },
    toggle() { setIsOpen(prev => !prev); },
  }));

  if (!isOpen) return null;
  return (
    <div className="modal-overlay">
      <div className="modal">
        {props.children}
        <button onClick={() => setIsOpen(false)}>Close</button>
      </div>
    </div>
  );
});

// Parent can control modal via ref
function ProductPage() {
  const modalRef = useRef();

  return (
    <div>
      <button onClick={() => modalRef.current.open()}>
        Quick View
      </button>
      <Modal ref={modalRef}>
        <ProductDetails />
      </Modal>
    </div>
  );
}
```

---

## 5. useLayoutEffect vs useEffect

| Hook | Runs When | Blocks Paint? | Use For |
|------|-----------|---------------|---------|
| `useEffect` | After paint | ❌ No | API calls, subscriptions |
| `useLayoutEffect` | Before paint | ✅ Yes | Measuring DOM, preventing flicker |

```jsx
// useLayoutEffect — measure element before user sees it
function Tooltip({ targetRef, text }) {
  const [pos, setPos] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef();

  useLayoutEffect(() => {
    // Runs BEFORE browser paints → no flicker!
    const rect = targetRef.current.getBoundingClientRect();
    setPos({ top: rect.bottom + 8, left: rect.left });
  }, []);

  return <div ref={tooltipRef} style={pos} className="tooltip">{text}</div>;
}
```

---

## 6. Stale Closures — The #1 Hooks Bug

A function captures an **old value** from when it was created, not the current value.

```jsx
// ❌ Bug: count always logs 0 because closure captured initial value
const [count, setCount] = useState(0);

useEffect(() => {
  const interval = setInterval(() => {
    console.log(count); // always 0! stale closure
  }, 1000);
  return () => clearInterval(interval);
}, []); // empty deps → closure captures count=0 forever

// ✅ Fix 1: Add count to deps (interval restarts on each change)
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(interval);
}, [count]);

// ✅ Fix 2: Use functional update (no dependency needed)
useEffect(() => {
  const interval = setInterval(() => {
    setCount(prev => prev + 1); // always gets latest
  }, 1000);
  return () => clearInterval(interval);
}, []);

// ✅ Fix 3: Use ref for latest value
const countRef = useRef(count);
useEffect(() => { countRef.current = count; }, [count]);
useEffect(() => {
  const interval = setInterval(() => {
    console.log(countRef.current); // always fresh
  }, 1000);
  return () => clearInterval(interval);
}, []);
```

---

## 7. State Batching (React 18)

React **batches** multiple state updates into a single re-render.

```jsx
// React 18: ONE re-render, not two!
function handleClick() {
  setCount(c => c + 1);
  setName("Alice");
  // React renders ONCE with both updates
}

// Even inside setTimeout and Promises (new in React 18!)
setTimeout(() => {
  setCount(c => c + 1);
  setName("Alice");
  // React 18: still batched → ONE render
  // React 17: NOT batched → TWO renders
}, 1000);
```

---

## ✅ Quick Revision Checklist

- [ ] `useMemo` — caches computed value; only re-computes when deps change
- [ ] `useCallback` — caches function reference; prevents child re-renders with `React.memo`
- [ ] `useReducer` — centralized state logic via actions + reducer; best for complex state
- [ ] `useImperativeHandle` — expose custom ref API to parent
- [ ] `useLayoutEffect` — runs before paint; use for DOM measurements
- [ ] Stale closures: fix with deps, functional updates, or refs
- [ ] React 18 batches state updates everywhere automatically
