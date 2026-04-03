# Part 6 — Advanced JS Patterns

> **Currying, Memoization, Generators, WeakMap/WeakSet, Garbage Collection & JS Engine internals.**

---

## 1. Currying

**Currying** transforms a function that takes multiple arguments into a chain of functions that each take **one argument**.

```
f(a, b, c)  →  f(a)(b)(c)
```

```js
// Normal function
function multiply(a, b) {
  return a * b;
}
multiply(4, 3); // 12

// Curried version
function currying(fn) {
  return function(a) {
    return function(b) {
      return fn(a, b);
    };
  };
}

var curriedMultiply = currying(multiply);
curriedMultiply(4)(3); // 12

// Practical use: partial application
var triple = curriedMultiply(3); // lock first arg
triple(5);  // 15
triple(10); // 30
```

### Real-world curry example: event logging

```js
const log = level => message => timestamp =>
  `[${timestamp}] [${level}] ${message}`;

const info = log("INFO");
const warn = log("WARN");

info("Server started")("2024-01-01T10:00:00"); // "[2024-01-01T10:00:00] [INFO] Server started"
warn("Disk space low")("2024-01-01T10:05:00"); // "[2024-01-01T10:05:00] [WARN] Disk space low"
```

---

## 2. Memoization

**Memoization** = caching the return value of a function based on its inputs. If the same input is seen again, return the cached result directly.

```js
// Non-memoized — expensive function
function add(num) {
  return num + 256; // imagine this is heavy computation
}

// Memoized version
function memoize(fn) {
  var cache = {};
  return function(num) {
    if (num in cache) {
      console.log("Cache hit!");
      return cache[num];
    }
    cache[num] = fn(num);
    return cache[num];
  };
}

var memoizedAdd = memoize(add);
memoizedAdd(20); // computed: 276
memoizedAdd(20); // "Cache hit!" — 276 (from cache)
memoizedAdd(40); // computed: 296
```

> ⚠️ Tradeoff: **Speed ↑** but **Memory ↑** (all results are stored)

### Memoizing with multiple arguments

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

---

## 3. Recursion

A function that **calls itself** until it hits a base case.

```js
// Sum of numbers 1 to n
function add(n) {
  if (n <= 0) return 0;         // base case
  return n + add(n - 1);        // recursive call
}

add(3); // 3 + 2 + 1 + 0 = 6

// Recursive sum of array
function computeSum(arr) {
  if (arr.length === 1) return arr[0]; // base case
  return arr.pop() + computeSum(arr);
}

computeSum([7, 8, 9, 99]); // 123
```

### Common recursion pitfalls

```js
// ❌ Missing base case → stack overflow!
function infinite(n) {
  return n + infinite(n - 1);
}

// ✅ Correct
function factorial(n) {
  if (n === 0) return 1;     // base case!
  return n * factorial(n - 1);
}
```

---

## 4. Generator Functions

Generator functions can **pause and resume** execution using `yield`.

```js
// Declared with function*
function* genFunc() {
  yield 3;
  yield 4;
  return 5;
}

const gen = genFunc(); // doesn't run yet, returns generator object

gen.next(); // { value: 3, done: false }
gen.next(); // { value: 4, done: false }
gen.next(); // { value: 5, done: true }
gen.next(); // { value: undefined, done: true }
```

### Iterator example

```js
function* iteratorFunc() {
  let count = 0;
  for (let i = 0; i < 2; i++) {
    count++;
    yield i;
  }
  return count;
}

let iterator = iteratorFunc();
iterator.next(); // {value: 0, done: false}
iterator.next(); // {value: 1, done: false}
iterator.next(); // {value: 2, done: true}
```

### Infinite sequence generator

```js
function* idMaker() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const getId = idMaker();
getId.next().value; // 1
getId.next().value; // 2
getId.next().value; // 3
// ... infinite but lazy!
```

---

## 5. WeakSet

- Stores **only objects** (no primitives)
- References are **weak** — if no other reference, object gets garbage collected
- Only 3 methods: `add()`, `delete()`, `has()`

```js
const ws = new WeakSet();

let obj1 = { message: "Hello" };
ws.add(obj1);
ws.has(obj1); // true

obj1 = null; // original reference removed
// obj1 is now eligible for garbage collection — WeakSet won't prevent it!

// ❌ No primitives
const ws2 = new WeakSet([3, 4, 5]); // TypeError!
```

**Use case**: Track which DOM elements have been processed without preventing cleanup.

---

## 6. WeakMap

- Keys must be **objects** (no primitives)
- Weak references — values GC'd when key is removed
- Methods: `set()`, `get()`, `delete()`, `has()`

```js
const wm = new WeakMap();

let user = { name: "Vivek" };
wm.set(user, { age: 23 });
wm.get(user); // { age: 23 }

user = null; // user object eligible for GC
// The WeakMap entry is automatically cleaned up

// ❌ String keys not allowed
const wm2 = new WeakMap();
wm2.set("key", "value"); // TypeError!
```

**Use case**: Associate private data with DOM elements or objects without memory leaks.

---

## 7. Garbage Collection — Mark & Sweep

V8 uses **mark-and-sweep** to free unused memory:

1. **Mark Phase**: Start from "roots" (globals, call stack) and mark every reachable object
2. **Sweep Phase**: Objects NOT marked are unreachable → freed from memory

```js
// Object eligible for GC when no references remain
let obj = { data: "large data" };
obj = null; // now eligible for garbage collection

// Memory leak example — hidden reference keeps object alive
const data = [];
function leakMemory() {
  const bigObject = new Array(1000).fill("data");
  data.push(bigObject); // reference kept in `data` array → never GC'd!
}
```

---

## 8. JS Engine Optimizations (V8)

### JIT Compilation
- V8 interprets JS first, then compiles **hot code** (frequently run) to machine code

### Hidden Classes
- V8 creates internal "hidden classes" for objects with identical shapes
- Modifying object structure (adding/deleting props) breaks hidden class optimization

```js
// ✅ Consistent shape — V8 can optimize
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const p1 = new Point(1, 2);
const p2 = new Point(3, 4);

// ❌ Dynamic shape — harder to optimize
const p3 = new Point(1, 2);
p3.z = 5;     // new property added dynamically
delete p3.x;  // property deleted
```

### Deopts (Deoptimization)
Variable type changes force V8 to throw away optimized code and re-interpret:

```js
// ❌ Type changes cause deopt
function add(a, b) { return a + b; }
add(1, 2);       // V8 optimizes for numbers
add("a", "b");   // Type changed! Deopt!

// ✅ Consistent types
function addNumbers(a, b) { return a + b; }
// Always call with numbers
```

---

## 9. Memory Leaks — Causes & Detection

### Common Causes

```js
// 1. Global variables
function leak() {
  leakedVar = "I'm a global!"; // forgot var/let/const
}

// 2. Forgotten timer
const intervalId = setInterval(() => {
  // runs forever if never cleared
  doSomething();
}, 1000);
// Fix: clearInterval(intervalId) when done

// 3. Event listeners not removed
const button = document.getElementById("btn");
button.addEventListener("click", handler);
// Fix: button.removeEventListener("click", handler) when element removed

// 4. Detached DOM nodes
let detachedDiv = document.createElement("div");
document.body.appendChild(detachedDiv);
document.body.removeChild(detachedDiv);
// detachedDiv variable still holds reference → memory leak!
```

### Detection with Chrome DevTools

1. Open DevTools → **Memory** tab
2. Take **Heap Snapshot** before and after action
3. Growing memory over time = likely leak

---

## ✅ Quick Revision Checklist

- [ ] Currying: `f(a, b)` → `f(a)(b)` — each function takes one arg
- [ ] Memoization: cache function results to avoid recomputation
- [ ] Recursion: function calls itself + needs a base case
- [ ] Generators: `function*` + `yield` — pause and resume execution
- [ ] WeakSet/WeakMap: weak references → GC-friendly, objects only
- [ ] V8 GC: mark reachable → sweep unreachable
- [ ] JIT: hot code gets compiled to machine code for speed
- [ ] Memory leaks: globals, timers, listeners, detached DOM nodes
