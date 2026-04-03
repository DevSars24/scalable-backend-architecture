# Part 5 — ES6+ Modern Syntax

> **The features introduced in ES6 (2015) and beyond that every modern JS developer must know.**

---

## 1. `var` vs `let` vs `const`

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| Scope | Function | Block | Block |
| Hoisted | ✅ (initialized as `undefined`) | ✅ (but in TDZ) | ✅ (but in TDZ) |
| Re-assignable | ✅ | ✅ | ❌ |
| Re-declarable | ✅ | ❌ | ❌ |
| Added to `window` | ✅ | ❌ | ❌ |

```js
// var — function scoped, added to window
var x = 5;
window.x; // 5

// let — block scoped, NOT on window
let y = 10;
window.y; // undefined

// Block scope demo
{
  var a = 1; // accessible outside block!
  let b = 2; // NOT accessible outside block
  const c = 3; // NOT accessible outside block
}
console.log(a); // 1 ✅
console.log(b); // ❌ ReferenceError
console.log(c); // ❌ ReferenceError
```

### `const` with objects

```js
const user = { name: "Vivek" };

user = { name: "Akki" }; // ❌ can't reassign the binding

user.name = "Akki"; // ✅ can modify properties!
```

> `const` prevents **reassignment**, NOT mutation of the object content.

---

## 2. Temporal Dead Zone (TDZ)

The TDZ is the period between when `let`/`const` are **hoisted** and when they are **initialized**.

```js
// The TDZ exists from start of block until the let declaration
console.log(x); // ❌ ReferenceError — in TDZ!
let x = 23;

function example() {
  message = "Hello"; // ❌ ReferenceError — in TDZ!
  let message;
}
```

> **Why does TDZ exist?** To catch bugs from using variables before they're ready.

---

## 3. Template Literals

Write strings with backticks for easier interpolation and multi-line support.

```js
const name = "Justin";
const age = 25;

// Old way
console.log("Hello " + name + ", you are " + age + " years old.");

// Template literal
console.log(`Hello ${name}, you are ${age} years old.`);

// Any expression works inside ${}
console.log(`Sum: ${2 + 3}`);
console.log(`Status: ${age >= 18 ? "Adult" : "Minor"}`);

// Multi-line — no \n needed
const html = `
  <div>
    <h1>Hello ${name}</h1>
  </div>
`;
```

### Tagged Templates

```js
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<b>${values[i]}</b>` : "");
  }, "");
}

const product = "Laptop";
const price = 999;

highlight`Buy the ${product} for just $${price}!`;
// "Buy the <b>Laptop</b> for just $<b>999</b>!"
```

---

## 4. Object & Array Destructuring

### Object Destructuring

```js
// Before ES6
const strength = classDetails.strength;
const benches = classDetails.benches;

// With destructuring
const classDetails = { strength: 78, benches: 39, blackBoard: 1 };
const { strength, benches, blackBoard } = classDetails;

// Rename while destructuring
const { strength: classStrength } = classDetails;
console.log(classStrength); // 78

// Default values
const { wifi = false } = classDetails;
console.log(wifi); // false (not in object)

// Nested destructuring
const { address: { city } } = { address: { city: "Mumbai" } };
console.log(city); // "Mumbai"
```

### Array Destructuring

```js
const arr = [1, 2, 3, 4];
const [first, second, third, fourth] = arr;

// Skip elements
const [a, , c] = [1, 2, 3]; // a=1, c=3

// Swap variables
let x = 1, y = 2;
[x, y] = [y, x]; // swap!
console.log(x, y); // 2, 1

// With rest
const [head, ...tail] = [1, 2, 3, 4];
// head = 1, tail = [2, 3, 4]
```

---

## 5. Rest Parameter & Spread Operator

Both use `...` but in different contexts.

### Rest Parameter — collects multiple args into array

```js
// Must be the LAST parameter
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4); // 10
sum(5, 10);      // 15

// Mix with regular params
function logDetails(name, age, ...hobbies) {
  console.log(name, age, hobbies);
}
logDetails("Alice", 25, "coding", "gaming", "reading");
// Alice 25 ["coding", "gaming", "reading"]
```

### Spread Operator — expands array/object

```js
// Spread in function call
function add(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
add(...nums); // 6

// Clone array (shallow)
const arr1 = [1, 2, 3];
const arr2 = [...arr1]; // [1, 2, 3] — new array

// Merge arrays
const merged = [...arr1, ...[4, 5, 6]]; // [1,2,3,4,5,6]

// Clone object
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a:1, b:2, c:3 }

// Override properties
const updated = { ...obj1, b: 99 }; // { a:1, b:99 }
```

| Feature | Rest `...` | Spread `...` |
|---------|-----------|-------------|
| Context | Function parameters | Function calls, array/object literals |
| Action | Collects into array | Expands array/object |

---

## 6. ES Modules — import / export

### Named Exports vs Default Export

```js
// utils.js
export const add = (a, b) => a + b;        // named
export const multiply = (a, b) => a * b;   // named

export default function greet(name) {       // default (only ONE allowed)
  return `Hello, ${name}`;
}
```

```js
// app.js
import greet, { add, multiply } from "./utils.js";
// greet  → default export (any name works)
// { add, multiply } → named exports (must match!)

// Rename named export
import { add as sum } from "./utils.js";

// Import everything
import * as utils from "./utils.js";
utils.add(1, 2); // 3
```

### Dynamic Import (lazy loading)

```js
// Load module only when needed
const module = await import("./heavyCalculation.js");
module.runHeavyTask();
```

### ES Modules vs CommonJS

| Feature | ES Modules | CommonJS |
|---------|-----------|---------|
| Syntax | `import` / `export` | `require()` / `module.exports` |
| Loading | Async (static analysis) | Sync |
| Where | Browsers + modern Node.js | Older Node.js |
| Tree-shaking | ✅ Supported | ❌ Not supported |

---

## 7. Optional Chaining (`?.`) & Nullish Coalescing (`??`)

### Optional Chaining — safe property access

```js
const user = { profile: { address: { city: "Mumbai" } } };

// Old way — verbose
const city = user && user.profile && user.profile.address && user.profile.address.city;

// Optional chaining — clean
const city = user?.profile?.address?.city; // "Mumbai"
const zip = user?.profile?.address?.zipCode; // undefined (no error!)

// Works with methods and arrays too
user?.getName?.();    // calls method if it exists
arr?.[0];             // safe array access
```

### Nullish Coalescing — default only for null/undefined

```js
// Problem with ||: treats 0 and "" as falsy
const port = config.port || 3000;  // if port=0, gets 3000 ❌

// ?? only defaults for null/undefined
const port = config.port ?? 3000;  // if port=0, gets 0 ✅

// Combining both
const city = user?.profile?.address?.city ?? "Unknown City";
```

---

## ✅ Quick Revision Checklist

- [ ] `var` = function scope; `let`/`const` = block scope
- [ ] TDZ = accessing `let`/`const` before declaration → ReferenceError
- [ ] `const` prevents reassignment, NOT mutation
- [ ] Template literals use backticks and `${expression}`
- [ ] Destructuring extracts values in fewer lines
- [ ] Rest `...` in params → collects; Spread `...` in calls → expands
- [ ] Named exports use `{}` on import; default doesn't
- [ ] `?.` prevents TypeError on null/undefined chains
- [ ] `??` only uses fallback when value is `null` or `undefined`
