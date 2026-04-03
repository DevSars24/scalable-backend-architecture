# 02 — Implementation: Secure Authentication and Routing 🔐

> **Core Concepts: Context API (`useContext`) and React Router v6.**

In this section, we implement **User Authentication** and **Protected Routing**. This ensures that only logged-in users can access the **Checkout** and **Admin** pages.

---

## 1. Context API (`useContext`)

### 🧠 Detailed Concept
**Context** is a way to share data (like user info or theme) with **many components** at once, without passing props through every single level (this is called "Prop Drilling").

### 🛒 Real-World Implementation
We create an `AuthContext` to store the `user` object. This way, the **Navbar**, **Sidebar**, and **Checkout** can all access the user's name and email instantly.

```jsx
import React, { createContext, useState, useContext } from 'react';

// 1. Create the Context.
const AuthContext = createContext();

// 2. Create the Provider (Wraps the whole app).
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null); // { id: 1, name: 'Saurabh', role: 'admin' }

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}

// 3. Custom Hook to easily use the Auth context.
export const useAuth = () => useContext(AuthContext);
```

---

## 2. Shared User Data (`useAuth`)

### 🛒 Real-World Implementation
Any component in ShopSmart can now check if the user is logged in.

```jsx
function Navbar() {
  const { user, logout, isAuthenticated } = useAuth();

  return (
    <nav className="navbar">
      <h1>ShopSmart</h1>
      {isAuthenticated ? (
        <div className="user-section">
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <button onClick={() => login({ name: 'Saurabh' })}>Login</button>
      )}
    </nav>
  );
}
```

---

## 3. React Router v6 & Protected Routes

### 🧠 Detailed Concept
**React Router** handles navigation between different URLs (e.g., `/products`, `/cart`) in a Single Page Application (SPA). **Protected Routes** are high-order components that verify if a user meets certain criteria (like being logged in) before rendering a page.

### 🛒 Real-World Implementation
We create a `ProtectedRoute` component to secure the checkout process.

```jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  // If the user is NOT authenticated, we redirect them to /login.
  // We save the 'current location' so we can redirect them back after they login.
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// Usage in App.js
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/cart" element={<Cart />} />
  <Route path="/checkout" element={
    <ProtectedRoute>
      <CheckoutPage />
    </ProtectedRoute>
  } />
</Routes>
```

---

## 🏗️ The Full Flow Recap
1. **User visits** the Home page.
2. The `AuthProvider` spans the whole tree, holding the `null` user.
3. User clicks **Checkout**.
4. `React Router` tries to load `/checkout`, but the `ProtectedRoute` sees `isAuthenticated` is `false`.
5. User is **Redirected** to `/login`.
6. User logs in → `AuthContext` now has the `user` → User is **Redirected back** to `/checkout` automatically.
