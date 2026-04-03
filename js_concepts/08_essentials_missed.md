# Part 8 — Essentials Often Missed in Interviews

> **The details that separate good developers from great ones.**

---

## 1. Deep Copy vs Shallow Copy

JavaScript stores objects as references. Copying one object to another can have surprising results.

### Shallow Copy

Copies **only the top level**. Nested objects are still shared.

```js
const user = {
  name: "Alice",
  address: { city: "Mumbai" }
};

// Methods that create shallow copies:
const copy1 = { ...user };           // spread operator
const copy2 = Object.assign({}, user); // Object.assign
const arr2 = [...originalArr];        // array shallow copy

// Problem with shallow copy!
copy1.address.city = "Delhi";
console.log(user.address.city); // "Delhi" — original was mutated! 😱
// Both point to the SAME nested `address` object
```

### Deep Copy

Creates a **fully independent clone** at all levels.

```js
// Old approach (has limitations)
const deepCopy = JSON.parse(JSON.stringify(user));
// ❌ Breaks for: undefined, functions, Date, Map, Set, circular refs

// Modern approach — structuredClone() (recommended)
const trueDeepCopy = structuredClone(user);
trueDeepCopy.address.city = "Delhi";
console.log(user.address.city); // "Mumbai" — original safe! ✅
```

**What `structuredClone()` supports:**

| Supported ✅ | NOT supported ❌ |
|-------------|----------------|
| Map, Set | Functions |
| Date, RegExp | DOM nodes |
| ArrayBuffer | Error objects |
| Circular references | — |

```js
// lodash for objects with functions
const _ = require("lodash");
const deepClone = _.cloneDeep(complexObject);
```

---

## 2. `map()` vs `forEach()`

| Feature | `map()` | `forEach()` |
|---------|---------|------------|
| Returns | New array | `undefined` |
| Chainable | ✅ | ❌ |
| Use when | You need the transformed result | You just need to run something |

```js
const numbers = [1, 2, 3];

// map — use when you NEED the result
const doubled = numbers.map(n => n * 2); // [2, 4, 6]

// forEach — use when you DON'T need the result
numbers.forEach(n => console.log(n)); // just print each

// Common mistake: using map without using the return value
numbers.map(n => console.log(n)); // ❌ map wasted — use forEach!
```

---

## 3. Error Handling: try/catch/finally & Custom Errors

### `try / catch / finally`

```js
try {
  const data = JSON.parse("invalid json"); // throws SyntaxError
} catch (error) {
  console.log("Caught:", error.message);
} finally {
  console.log("Always runs — cleanup here");
  // even if try returns, finally still runs!
}
```

### Throwing custom errors

```js
// Basic throw
throw new Error("Something went wrong");

// ❌ Avoid throwing primitives — no stack trace
throw "Invalid input"; // string — bad practice

// ✅ Always throw Error objects
throw new Error("Invalid input");
```

### Custom error classes

```js
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

class AuthenticationError extends Error {
  constructor(message) {
    super(message);
    this.name = "AuthenticationError";
    this.statusCode = 401;
  }
}

function registerUser(user) {
  if (!user.email) {
    throw new ValidationError("email", "Email is required");
  }
  if (user.password.length < 6) {
    throw new ValidationError("password", "Password too short");
  }
}

try {
  registerUser({ email: "", password: "123" });
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Field "${error.field}": ${error.message}`);
  } else {
    console.log("Unexpected error:", error.message);
  }
}
```

### Async error handling

```js
// async/await style
async function getData() {
  try {
    const res = await fetch("https://api.example.com/data");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    console.error("Fetch failed:", error.message);
  }
}

// Promise style
fetch("https://api.example.com/data")
  .then(res => res.json())
  .catch(err => console.error(err.message));

// ❌ Silent error swallowing — the cardinal sin
try {
  doSomething();
} catch (e) {} // never do this! impossible to debug
```

### Global error handlers

```js
window.onerror = function(message, source, lineno, colno, error) {
  console.error("Global error:", message);
};

window.onunhandledrejection = function(event) {
  console.error("Unhandled Promise:", event.reason);
};
```

---

## 4. localStorage vs sessionStorage vs Cookies

| Feature | localStorage | sessionStorage | Cookies |
|---------|-------------|----------------|---------|
| Lifetime | Until manually cleared | Until tab closes | Set expiry or session |
| Shared across tabs | ✅ | ❌ (tab-specific) | ✅ |
| Sent to server | ❌ | ❌ | ✅ (every request!) |
| Size limit | ~5–10 MB | ~5 MB | ~4 KB |
| JS access | ✅ | ✅ | ✅ (unless httpOnly) |

```js
// localStorage — persists forever
localStorage.setItem("theme", "dark");
localStorage.getItem("theme"); // "dark"
localStorage.removeItem("theme");
localStorage.clear();

// sessionStorage — cleared when tab closes
sessionStorage.setItem("formStep", "2");

// Cookies — basic usage
document.cookie = "user=Pranav; max-age=3600; path=/";

// Reading cookies (awkward without libraries)
document.cookie; // "user=Pranav; anotherCookie=value"
```

### Security consideration

```js
// ❌ Don't store tokens in localStorage — vulnerable to XSS
localStorage.setItem("authToken", token);

// ✅ Use httpOnly cookies for auth tokens — JS can't access them
// Server sets: Set-Cookie: token=abc; HttpOnly; Secure; SameSite=Strict
```

---

## 5. `map()`, `filter()`, `reduce()` & Custom `map()`

```js
const users = [
  { name: "Alice", age: 28, isActive: true },
  { name: "Bob", age: 17, isActive: false },
  { name: "Carol", age: 32, isActive: true }
];

// filter — select elements that match condition
const activeUsers = users.filter(u => u.isActive);
// [Alice, Carol]

// map — transform each element
const names = activeUsers.map(u => u.name);
// ["Alice", "Carol"]

// reduce — fold array into single value
const totalAge = users.reduce((sum, u) => sum + u.age, 0);
// 77

// Chaining — power combo!
const result = users
  .filter(u => u.isActive)
  .map(u => u.name)
  .reduce((acc, name) => acc + ", " + name);
// "Alice, Carol"
```

### ⚠️ Common reduce mistake: missing initial value

```js
// ❌ No initial value — breaks on empty array
[].reduce((a, b) => a + b); // TypeError!

// ✅ Always provide initial value
[].reduce((a, b) => a + b, 0); // 0 (safe)
```

### Implement your own `map()`

```js
function myMap(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i], i, arr));
    //                    value  index  original array
  }
  return result;
}

// Usage — identical to built-in
myMap([1, 2, 3], n => n * 2); // [2, 4, 6]
```

---

## 6. `==` vs `===` for Objects

For objects, **both `==` and `===` compare references**, NOT content.

```js
const a = { name: "Ankit" };
const b = { name: "Ankit" };

a == b;  // false — different memory locations!
a === b; // false — different memory locations!

const c = a;
a === c; // true — same reference!
```

### Comparing object values

```js
// Quick (but limited) — breaks with functions, undefined, circular refs
JSON.stringify(a) === JSON.stringify(b); // true (for simple objects)

// Proper deep comparison
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) return true;
  if (typeof obj1 !== "object" || typeof obj2 !== "object") return false;
  if (obj1 === null || obj2 === null) return false;

  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  if (keys1.length !== keys2.length) return false;

  return keys1.every(key => deepEqual(obj1[key], obj2[key]));
}
```

---

## ✅ Quick Revision Checklist

- [ ] Shallow copy → spread `{...obj}` or `Object.assign()` — nested objects still shared
- [ ] Deep copy → `structuredClone()` (modern) or `JSON.parse(JSON.stringify())` (limited)
- [ ] `map()` returns new array; `forEach()` returns nothing
- [ ] `try/catch/finally` — `finally` ALWAYS runs
- [ ] Custom errors: `class MyError extends Error` — use `instanceof` to check type
- [ ] `localStorage` = permanent; `sessionStorage` = tab session; `cookies` = sent to server
- [ ] Don't store auth tokens in `localStorage` — use `httpOnly` cookies
- [ ] Objects compared by **reference**, not by value
- [ ] `reduce()` always provide an initial value to avoid edge case bugs
