# 01 — Implementation: The Dynamic Shopping Cart 🛍️

> **Core Concepts: State, Props, Lists, and Keys.**

In this section, we build the **Shopping Cart**. This is the heart of any e-commerce app, where data changes constantly as users add, remove, or update product quantities.

---

## 1. Component State (`useState`)

### 🧠 Detailed Concept
**State** is the "memory" of a single component. It represents data that can change over time. When state changes, React **re-renders** the component to reflect those changes in the UI.

### 🛒 Real-World Implementation
In ShopSmart, the `CartContainer` component needs to remember which items the user has selected.

```jsx
import React, { useState } from 'react';

function CartContainer() {
  // 1. We initialize the state as an empty array of cart items.
  const [cartItems, setCartItems] = useState([
    { id: 'p1', name: 'Nike Air Max', price: 120, qty: 1 },
    { id: 'p2', name: 'Adidas Ultraboost', price: 180, qty: 2 },
  ]);

  // Function to update the state when a user changes quantity.
  const updateQty = (id, newQty) => {
    setCartItems(prevItems => 
      prevItems.map(item => 
        item.id === id ? { ...item, qty: newQty } : item
      )
    );
  };

  return (
    <div className="cart-container">
      <h2>Your Shopping Cart</h2>
      {cartItems.map(item => (
        /* 2. Passing data and functions as PROPS */
        <CartItem 
          key={item.id} 
          item={item} 
          onUpdate={updateQty} 
        />
      ))}
    </div>
  );
}
```

---

## 2. Component Props (Properties)

### 🧠 Detailed Concept
**Props** are like "arguments" passed to a function. They allow a parent component to send data (configuration) or functions (behavior) down to a child component. Props are **read-only** (immutable) from the child's perspective.

### 🛒 Real-World Implementation
Each individual `CartItem` doesn't "own" its data—it receives it from the parent.

```jsx
function CartItem({ item, onUpdate }) {
  return (
    <div className="cart-item">
      <span>{item.name}</span>
      <span>${item.price}</span>
      <input 
        type="number" 
        value={item.qty} 
        onChange={(e) => onUpdate(item.id, parseInt(e.target.value))} 
      />
      <span>Total: ${item.price * item.qty}</span>
    </div>
  );
}
```

---

## 3. Lists and Keys

### 🧠 Detailed Concept
When rendering a list of items (like a cart), React uses the **`key`** prop to track which item is which. 

### 🛒 Why are Keys "Critical" in Production?
If you don't use a stable, unique key (like `item.id`):
- React might **incorrectly re-render** items.
- If you use the **index** as the key and delete the second item, React thinks the *third* item is now the second one, often causing visual bugs in input fields or animations.

**✅ Correct Production Usage:**
```jsx
{cartItems.map(item => (
  <CartItem key={item.id} item={item} />
))}
```
**❌ Unsafe Production Usage (Don't do this with dynamic lists):**
```jsx
{cartItems.map((item, index) => (
  <CartItem key={index} item={item} />
))}
```

---

## 🏗️ The Full Flow Recap
1. **User clicks** a quantity input.
2. The `CartItem` child calls the `onUpdate` function (passed via **Props**).
3. The `CartContainer` parent updates its `cartItems` array (**State**).
4. React sees the State change and looks for the specific `CartItem` (**Key**) to update.
5. The **UI reflects** the new total price instantly.
