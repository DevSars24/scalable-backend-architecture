# 03 — Implementation: High-Performance Admin Filters 🏎️

> **Core Concepts: `useMemo`, `useCallback`, and `React.memo`.**

In this section, we build the **Admin Dashboard**, which frequently deals with large datasets (e.g., 5,000+ orders). Without optimization, the UI can feel "laggy" or slow when a user types in a search bar or applies filters.

---

## 1. `useMemo` — Caching Expensive Calculations

### 🧠 Detailed Concept
**`useMemo`** memoizes (caches) a **result** of a function. It's used when you have a piece of logic that is expensive to compute (like filtering or sorting a large array). React only recalculates this value if its dependencies change.

### 🛒 Real-World Implementation
In ShopSmart, an admin might be filtering 5,000 orders by status (Pending, Shipped, Delivered).

```jsx
import React, { useState, useMemo } from 'react';

function AdminDashboard({ allOrders }) {
  const [filterStatus, setFilterStatus] = useState('all');
  const [query, setQuery] = useState('');

  // 1. We wrap the filter in useMemo.
  // 2. React only re-performs the filter if 'allOrders', 'query', or 'filterStatus' changes.
  // 3. Typing in a DIFFERENT field (like 'Theme Toggle') won't trigger this filter.
  const filteredOrders = useMemo(() => {
    console.log("Filtering 5,000 orders... (expensive)");
    return allOrders.filter(order => {
      const matchStatus = filterStatus === 'all' || order.status === filterStatus;
      const matchQuery = order.customerName.toLowerCase().includes(query.toLowerCase());
      return matchStatus && matchQuery;
    });
  }, [allOrders, filterStatus, query]);

  return (
    <div className="admin-dashboard">
      <input 
        value={query} 
        onChange={(e) => setQuery(e.target.value)} 
        placeholder="Search Orders..." 
      />
      
      {/* Displaying the filtered results */}
      <OrderTable orders={filteredOrders} />
    </div>
  );
}
```

---

## 2. `useCallback` — Caching Function References

### 🧠 Detailed Concept
**`useCallback`** memoizes (caches) a **function** itself. 

In JavaScript, `() => {}` (an arrow function) creates a **new object** in memory every time a component re-renders. If you pass this function as a prop to a child component, the child will **re-render** because it thinks its props changed. `useCallback` prevents this.

### 🛒 Real-World Implementation
In ShopSmart, the "Delete" function is passed down to every single `OrderRow`.

```jsx
import React, { useCallback } from 'react';

function AdminDashboard({ allOrders, deleteOrder }) {
  // 1. We wrap the handleDelete in useCallback.
  // 2. The handleDelete function reference will STAY THE SAME across renders.
  const handleDelete = useCallback((id) => {
    deleteOrder(id);
    alert(`Order ${id} deleted!`);
  }, [deleteOrder]);

  return (
    <div className="order-list">
      {allOrders.map(order => (
        <OrderRow 
          key={order.id} 
          order={order} 
          onDelete={handleDelete} 
        />
      ))}
    </div>
  );
}
```

---

## 3. `React.memo` — Skipping Re-renders

### 🧠 Detailed Concept
**`React.memo`** is a Higher-Order Component (HOC) that tells React: "Only re-render this component IF its props have changed." 

### 🛒 Real-World Implementation
Each individual `OrderRow` contains a lot of UI. We want to skip re-rendering all 100 rows just because the admin changed a global theme or a search query.

```jsx
// 1. Wrap the entire component in React.memo.
const OrderRow = React.memo(({ order, onDelete }) => {
  console.log(`Rendering Row for order: ${order.id}`); // This only logs when THIS row's data changes.
  
  return (
    <div className="order-row">
      <span>{order.customerName}</span>
      <span>{order.total}</span>
      <button onClick={() => onDelete(order.id)}>Delete</button>
    </div>
  );
});
```

---

## 🏗️ The Full Flow Recap
1. **Admin types** a character into the Search input.
2. The **`AdminDashboard` re-renders**.
3. **`useMemo`** re-calculates the `filteredOrders` list (because the `query` changed).
4. **`useCallback`** provides the **EXACT SAME** `handleDelete` function as before.
5. React goes to render the **`OrderRow`** list. 
6. **`React.memo`** checks: "Has the `order` data changed? No. Has the `onDelete` function changed? No (thanks to `useCallback`)."
7. **`OrderRow` SKIPS rendering**, saving CPU cycles and making the app feel **instantly responsive**.
