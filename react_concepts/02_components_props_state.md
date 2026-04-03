# Part 2 — Components, Props, State & Keys

> **The building blocks of every React application.**

---

## 1. Functional vs Class Components

Before React 16.8 (Hooks), class components were needed for state and lifecycle. Today, **functional components are the standard**.

### Functional Component (Modern — Preferred ✅)

```jsx
// Arrow function style
const ProductCard = (props) => {
  return <div className="card">{props.name}</div>;
};

// Or regular function style
function ProductCard(props) {
  return <div className="card">{props.name}</div>;
}
```

### Class Component (Legacy — Still valid ✅)

```jsx
class ProductCard extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <div className="card">{this.props.name}</div>;
  }
}
```

### Side-by-Side Comparison

| Feature | Functional | Class |
|---------|-----------|-------|
| State | `useState()` hook | `this.state` + `this.setState()` |
| Lifecycle | `useEffect()` hook | `componentDidMount()` etc. |
| `this` keyword | ❌ Not needed | ✅ Required everywhere |
| Code size | Shorter, cleaner | More boilerplate |
| Performance | Slightly better | Slightly heavier |
| Recommended | ✅ Yes | For legacy codebases |

---

## 2. Props — Passing Data Parent → Child

**Props** (properties) are the way to pass data and behavior from a parent component into a child component. They are **read-only** inside the child.

### 🛒 Real App: Product Grid passing data to each Card

```jsx
// PARENT — ProductGrid
function ProductGrid() {
  const products = [
    { id: 1, name: "Nike Air Max", price: 8999, rating: 4.5, inStock: true },
    { id: 2, name: "Adidas Ultraboost", price: 12999, rating: 4.8, inStock: false },
    { id: 3, name: "Puma RS-X", price: 6499, rating: 4.2, inStock: true },
  ];

  const handleAddToCart = (product) => {
    console.log("Added:", product.name);
  };

  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard
          key={product.id}
          name={product.name}
          price={product.price}
          rating={product.rating}
          inStock={product.inStock}
          onAddToCart={handleAddToCart}  {/* passing function as prop */}
        />
      ))}
    </div>
  );
}

// CHILD — ProductCard
function ProductCard({ name, price, rating, inStock, onAddToCart }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>₹{price.toLocaleString()}</p>
      <p>⭐ {rating}</p>
      {inStock ? (
        <button onClick={() => onAddToCart({ name, price })}>
          Add to Cart
        </button>
      ) : (
        <button disabled>Out of Stock</button>
      )}
    </div>
  );
}
```

### Props Rules

```jsx
// ✅ Props are READ-ONLY — never modify them inside child!
function Child({ price }) {
  price = price * 1.1; // ❌ Never do this!
  return <p>₹{price}</p>;
}

// ✅ Default props
function Button({ label = "Click me", variant = "primary" }) {
  return <button className={variant}>{label}</button>;
}

// ✅ Spread props
const config = { type: "text", placeholder: "Search...", maxLength: 100 };
<input {...config} />
```

---

## 3. State — A Component's Memory

**State** is data that belongs to a component and can change over time. When state changes, React **re-renders** the component.

### 🛒 Real App: Shopping Cart Counter

```jsx
function CartButton() {
  const [cartCount, setCartCount] = React.useState(0);
  const [isAnimating, setIsAnimating] = React.useState(false);

  const addToCart = () => {
    setCartCount(prev => prev + 1);     // ✅ Always use functional update
    setIsAnimating(true);

    setTimeout(() => setIsAnimating(false), 300);
  };

  return (
    <button
      onClick={addToCart}
      className={isAnimating ? "cart-btn animate" : "cart-btn"}
    >
      🛒 Cart ({cartCount})
    </button>
  );
}
```

### State vs Props

| | State | Props |
|-|-------|-------|
| Owned by | The component itself | Parent component |
| Mutable? | ✅ Yes (via setState/setX) | ❌ No (read-only) |
| Triggers re-render? | ✅ Yes | ✅ Yes (if parent re-renders) |
| Passed from | Internal | Outside |

### Complex State — Object State

```jsx
function CheckoutForm() {
  const [form, setForm] = React.useState({
    name: "",
    email: "",
    address: "",
    pincode: ""
  });

  // ✅ Correct: spread existing state, update only one field
  const updateField = (field) => (e) => {
    setForm(prev => ({ ...prev, [field]: e.target.value }));
  };

  return (
    <form>
      <input
        value={form.name}
        onChange={updateField("name")}
        placeholder="Full Name"
      />
      <input
        value={form.email}
        onChange={updateField("email")}
        placeholder="Email"
      />
      <input
        value={form.address}
        onChange={updateField("address")}
        placeholder="Delivery Address"
      />
    </form>
  );
}
```

---

## 4. Keys in React — Helping React Track List Items

Keys help React identify **which item changed, was added, or removed** in a list.

### 🛒 Real App: Order History List

```jsx
function OrderHistory({ orders }) {
  return (
    <ul className="orders">
      {orders.map(order => (
        // ✅ Use unique, stable ID — NOT array index!
        <li key={order.orderId}>
          <span>#{order.orderId}</span>
          <span>{order.product}</span>
          <span>{order.status}</span>
        </li>
      ))}
    </ul>
  );
}
```

### Why NOT to use index as key

```jsx
// ❌ Dangerous — causes bugs when list changes
{orders.map((order, index) => (
  <li key={index}>  {/* Bug: if order is deleted/reordered, inputs shift! */}
    <input defaultValue={order.note} />
  </li>
))}

// ✅ Always use a unique, stable ID
{orders.map(order => (
  <li key={order.orderId}>
    <input defaultValue={order.note} />
  </li>
))}
```

**What happens with index keys when you delete item 1 from [A, B, C]?**

```
Before:  key=0→A  key=1→B  key=2→C
After:   key=0→B  key=1→C

React thinks:  B (key=0) = A updated  ← WRONG!
               C (key=1) = B updated  ← WRONG!
```

> ⚠️ This causes state (like input values) to appear in the WRONG row!

---

## 5. Controlled vs Uncontrolled Components

### Controlled Component — React manages the value

```jsx
function SearchBar() {
  const [query, setQuery] = React.useState("");
  const [suggestions, setSuggestions] = React.useState([]);

  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);

    // ✅ React drives the input value — can validate, filter, suggest
    if (value.length > 2) {
      fetchSuggestions(value).then(setSuggestions);
    } else {
      setSuggestions([]);
    }
  };

  return (
    <div>
      <input
        type="text"
        value={query}           {/* ← controlled by React state */}
        onChange={handleChange}
        placeholder="Search products..."
      />
      {suggestions.map(s => <p key={s.id}>{s.name}</p>)}
    </div>
  );
}
```

### Uncontrolled Component — DOM manages the value

```jsx
function QuickContactForm() {
  const nameRef = React.useRef();
  const emailRef = React.useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    // ✅ Read value only when needed (on submit)
    const data = {
      name: nameRef.current.value,
      email: emailRef.current.value,
    };
    console.log("Submitted:", data);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} type="text" placeholder="Name" />
      <input ref={emailRef} type="email" placeholder="Email" />
      <button type="submit">Send</button>
    </form>
  );
}
```

### When to use which?

| Use Case | Approach |
|----------|----------|
| Real-time validation (email format) | Controlled |
| Dependent fields (city depends on state) | Controlled |
| Conditional submit button (disable if invalid) | Controlled |
| Simple one-time form (contact, feedback) | Uncontrolled |
| Third-party DOM library integration | Uncontrolled |

---

## 6. Props Drilling Problem

When you pass props through many layers just to reach a deeply nested component:

```
App
 └── Header
      └── Nav
           └── UserMenu
                └── UserAvatar  ← needs `user` prop
```

```jsx
// ❌ Prop drilling — Header, Nav, UserMenu don't need user but must pass it
<App user={user}>
  <Header user={user}>
    <Nav user={user}>
      <UserMenu user={user}>
        <UserAvatar user={user} />    {/* finally uses it! */}
      </UserMenu>
    </Nav>
  </Header>
</App>
```

> 💡 Solution: **Context API** (covered in Part 7) or state management (Redux/Zustand)

---

## ✅ Quick Revision Checklist

- [ ] Functional components = modern standard; class = legacy
- [ ] Props flow one way: parent → child; they are READ-ONLY
- [ ] State = component's internal memory; triggers re-render on change
- [ ] Always use functional updates: `setCount(prev => prev + 1)`
- [ ] Keys must be unique among siblings and stable (not array index!)
- [ ] Controlled = React drives input value; Uncontrolled = DOM drives it
- [ ] Prop drilling = passing props through many layers (solved by Context)
