# Part 4 — Async JavaScript: Callbacks, Promises & Async/Await

> **How JavaScript handles time-consuming operations without blocking the main thread.**

---

## 1. The JavaScript Event Loop

JavaScript is **single-threaded** — it can only do one thing at a time. Yet it handles async operations smoothly through the **Event Loop**.

### Key Components

```
┌──────────────────────────────────────────┐
│              Call Stack                  │  ← synchronous code runs here
└──────────────────────────────────────────┘
          ↓ when empty, pulls from:
┌──────────────────┐   ┌───────────────────┐
│  Microtask Queue │   │  Macrotask Queue  │
│  (Promises,      │   │  (setTimeout,     │
│   queueMicrotask)│   │   setInterval,    │
│                  │   │   DOM events)     │
└──────────────────┘   └───────────────────┘
       ↑ runs FIRST        ↑ runs AFTER microtasks
```

### Execution order example

```js
console.log("1");                      // synchronous

setTimeout(() => console.log("2"), 0); // macrotask queue

Promise.resolve().then(() => console.log("3")); // microtask queue

console.log("4");                      // synchronous

// Output: 1, 4, 3, 2
```

**Why?**
1. `"1"` — synchronous, runs immediately
2. `setTimeout` → handed to Web APIs, callback goes to macrotask queue
3. `Promise` callback → goes to microtask queue
4. `"4"` — synchronous, runs immediately
5. Call stack is empty → microtask queue drains first → `"3"`
6. Macrotask runs → `"2"`

> 💡 **Rule**: Microtasks ALWAYS run before the next macrotask.

---

## 2. Callbacks

A **callback** is a function passed as an argument to another function, executed after the outer function completes.

```js
function divideByHalf(sum) {
  console.log(Math.floor(sum / 2));
}

function multiplyBy2(sum) {
  console.log(sum * 2);
}

function operationOnSum(num1, num2, operation) {
  var sum = num1 + num2;
  operation(sum); // ← calling the callback
}

operationOnSum(3, 3, divideByHalf); // 3
operationOnSum(5, 5, multiplyBy2);  // 20
```

### Problem: Callback Hell

```js
// Hard to read, maintain, and debug
getUser(id, function(user) {
  getOrders(user, function(orders) {
    getOrderDetails(orders[0], function(details) {
      processPayment(details, function(result) {
        console.log(result); // 4 levels deep! 😱
      });
    });
  });
});
```

---

## 3. Promises

A **Promise** is an object representing the eventual result of an async operation.

### Promise States

| State | Meaning |
|-------|---------|
| `Pending` | Initial — neither fulfilled nor rejected |
| `Fulfilled` | Async operation succeeded |
| `Rejected` | Async operation failed |
| `Settled` | Either fulfilled or rejected |

```js
function sumOfThreeElements(...elements) {
  return new Promise((resolve, reject) => {
    if (elements.length > 3) {
      reject("Only 3 elements or fewer allowed");
    } else {
      let sum = elements.reduce((a, b) => a + b, 0);
      resolve("Sum: " + sum);
    }
  });
}

// Consuming a promise
sumOfThreeElements(4, 5, 6)
  .then(result => console.log(result))  // "Sum: 15"
  .catch(error => console.log(error));

sumOfThreeElements(7, 0, 33, 41)
  .then(result => console.log(result))
  .catch(error => console.log(error));  // "Only 3 elements or fewer allowed"
```

### Promise Chaining (vs Callback Hell)

```js
getUser(id)
  .then(user => getOrders(user))
  .then(orders => getOrderDetails(orders[0]))
  .then(details => processPayment(details))
  .then(result => console.log(result))
  .catch(err => console.error(err)); // one catch handles ALL errors
```

---

## 4. Async / Await

`async/await` is **syntactic sugar over Promises** — makes async code look and behave like synchronous code.

```js
// Without async/await
function fetchData() {
  return fetch("https://api.example.com/users")
    .then(res => res.json())
    .then(data => console.log(data))
    .catch(err => console.error(err));
}

// With async/await
async function fetchData() {
  try {
    const res = await fetch("https://api.example.com/users");
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error("Error:", err);
  }
}
```

> `await` pauses execution of **only the async function** (not the entire program).

### Rules for async/await

```js
// ✅ async function always returns a Promise
async function getValue() {
  return 10; // auto-wrapped: Promise.resolve(10)
}

// ✅ await can only be used INSIDE an async function
async function example() {
  const result = await somePromise;
}

// ❌ await at top level fails in older environments
const data = await fetch("..."); // ❌ (unless in ESM with top-level await)
```

---

## 5. Promise Utility Methods

### `Promise.all()` — All must succeed

```js
const [users, orders] = await Promise.all([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/orders").then(r => r.json())
]);
// Runs BOTH in parallel → faster!
// If ONE fails → the whole thing fails
```

### `Promise.allSettled()` — Get all results, even failures

```js
const results = await Promise.allSettled([
  fetch("/api/users"),
  fetch("/api/broken-endpoint")
]);

results.forEach(r => {
  if (r.status === "fulfilled") console.log(r.value);
  else console.log("Failed:", r.reason);
});
```

### `Promise.race()` — First one wins

```js
const result = await Promise.race([
  fetch("/api/fast"),
  fetch("/api/slow")
]);
// Returns the FIRST promise to settle (fulfilled OR rejected)
```

### `Promise.any()` — First SUCCESS wins

```js
const result = await Promise.any([
  fetch("/server1"),
  fetch("/server2"), // if server1 fails, server2 still wins
]);
```

---

## 6. Sequential vs Parallel Async

```js
// ❌ Sequential — slow (each waits for previous)
const user1 = await fetchUser(1);
const user2 = await fetchUser(2);
const user3 = await fetchUser(3);

// ✅ Parallel — fast (all run simultaneously!)
const [user1, user2, user3] = await Promise.all([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3)
]);
```

### Common mistake: await inside forEach

```js
// ❌ forEach doesn't await properly
users.forEach(async (user) => {
  await processUser(user); // these all run simultaneously, uncontrolled
});

// ✅ Use for...of for sequential async
for (const user of users) {
  await processUser(user); // truly waits for each
}
```

---

## 7. Microtasks vs Macrotasks — Deep Dive

| Category | Examples | Queue |
|----------|---------|-------|
| Macrotask | `setTimeout`, `setInterval`, DOM events, I/O | Macrotask Queue |
| Microtask | `Promise.then()`, `queueMicrotask()`, `MutationObserver` | Microtask Queue |

```js
// Microtasks created by microtasks still run before macrotasks
Promise.resolve()
  .then(() => {
    console.log("micro 1");
    return Promise.resolve(); // creates another microtask!
  })
  .then(() => console.log("micro 2"));

setTimeout(() => console.log("macro"), 0);

// Output: micro 1, micro 2, macro
```

---

## 8. Types of Errors in JavaScript

| Type | Description | Example |
|------|-------------|---------|
| `SyntaxError` | Code that can't be parsed | `let x = ;` |
| `ReferenceError` | Using undeclared variable | `console.log(undeclaredVar)` |
| `TypeError` | Wrong type operation | `null.property` |
| `RangeError` | Value out of range | `new Array(-1)` |
| Logical Error | Code runs but wrong output | Wrong formula used |

```js
// Syntax error — code doesn't run at all
let x = ; // ❌ SyntaxError

// Reference error — variable doesn't exist
console.log(z); // ❌ ReferenceError

// Type error — wrong operation on type
null.toString(); // ❌ TypeError

// Logical error — hardest to find
function calculateArea(r) {
  return 2 * Math.PI * r; // ❌ This is circumference, not area!
}
```

---

## ✅ Quick Revision Checklist

- [ ] Event Loop: Call Stack → Microtask Queue → Macrotask Queue
- [ ] Callback Hell = deeply nested callbacks (solved by Promises)
- [ ] Promise states: Pending → Fulfilled / Rejected → Settled
- [ ] `async` function always returns a Promise
- [ ] `await` pauses the async function, not the full program
- [ ] `Promise.all` = parallel, fails if any fail
- [ ] `Promise.allSettled` = parallel, returns all results
- [ ] Microtasks always run before macrotasks
