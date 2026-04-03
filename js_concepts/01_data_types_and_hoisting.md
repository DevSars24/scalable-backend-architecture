# Part 1 — Data Types, Hoisting & Type System

> **Core fundamentals every JS developer must know cold.**

---

## 1. What are the Different Data Types in JavaScript?

JavaScript has **7 primitive types** + **1 non-primitive (Object)**.

### 🔹 Primitive Types

| Type | Example | `typeof` result |
|------|---------|----------------|
| String | `"Hello"` | `"string"` |
| Number | `3.14` | `"number"` |
| BigInt | `12345n` | `"bigint"` |
| Boolean | `true` / `false` | `"boolean"` |
| Undefined | `let x;` | `"undefined"` |
| Null | `let y = null;` | `"object"` ⚠️ (JS bug) |
| Symbol | `Symbol('id')` | `"symbol"` |

```js
typeof "John"         // "string"
typeof 3.14           // "number"
typeof true           // "boolean"
typeof undefined      // "undefined"
typeof null           // "object"  ← famous JS quirk!
typeof Symbol('x')    // "symbol"
typeof 234567890n     // "bigint"
```

### 🔹 Non-Primitive Types (Object)

```js
// Object — key-value pairs
var obj = { x: 43, y: "Hello", z: function(){ return this.x; } };

// Array — ordered list (also an Object!)
var arr = [5, "Hello", true, 4.1];
```

> 💡 **Remember**: Any data type that is NOT primitive is of type `Object` in JS.

---

## 2. Hoisting in JavaScript

**Hoisting** = JavaScript moves all **declarations** (not initializations) to the top of their scope before code runs.

```js
// ✅ Works — variable declaration is hoisted
hoistedVar = 3;
console.log(hoistedVar); // 3
var hoistedVar;

// ✅ Works — function declaration is hoisted entirely
hoistedFn();
function hoistedFn() {
  console.log("Hello world!");
}

// ⚠️ Undefined — only declaration hoisted, NOT initialization
console.log(x); // undefined
var x = 23;

// ❌ Error — let/const are hoisted but NOT initialized (Temporal Dead Zone)
console.log(y); // ReferenceError
let y = 23;
```

### How to avoid hoisting bugs?

Use `"use strict"` at the top:

```js
"use strict";
x = 23; // ❌ ReferenceError: x is not defined
var x;
```

---

## 3. `==` vs `===` Operators

| Operator | Compares | Type Coercion? |
|----------|----------|----------------|
| `==` | Values only | ✅ Yes |
| `===` | Values AND types | ❌ No |

```js
var x = 2;
var y = "2";

x == y   // true  → "2" is coerced to number 2
x === y  // false → number !== string
```

> 💡 **Rule of thumb**: Always prefer `===` to avoid unexpected bugs.

---

## 4. What is `NaN`?

`NaN` = "Not a Number" — a special numeric value for invalid math operations.

```js
typeof NaN  // "number" ← another JS quirk!

isNaN("Hello")    // true  — "Hello" can't convert to number
isNaN(345)        // false — it IS a number
isNaN('1')        // false — '1' converts to 1
isNaN(true)       // false — true converts to 1
isNaN(undefined)  // true
```

> ⚠️ `isNaN()` converts the value first. Use `Number.isNaN()` for strict checking:
```js
Number.isNaN("Hello") // false — doesn't convert, just checks
Number.isNaN(NaN)     // true
```

---

## 5. Implicit Type Coercion

JavaScript automatically converts types during operations.

### String Coercion (`+` operator)

```js
3 + "3"       // "33"  → number becomes string
24 + "Hello"  // "24Hello"
"Vivek" + " Bisht" // "Vivek Bisht" (string concat)
```

### Numeric Coercion (`-` operator)

```js
3 - "3"   // 0  → string "3" becomes number 3
```

### Boolean Coercion

**Falsy values** (converted to `false`):
```
false, 0, 0n, -0, "", null, undefined, NaN
```

**Everything else is truthy.**

```js
if(0)   { /* skipped */ }
if(23)  { /* runs */ }
if("")  { /* skipped */ }
if("0") { /* runs — non-empty string is truthy! */ }
```

### Logical Operator Coercion

```js
// OR (||) — returns first TRUTHY value
220 || "Hello"    // 220
null || "default" // "default"

// AND (&&) — returns first FALSY value, or last value if all truthy
220 && "Hello"    // "Hello" (both truthy → returns last)
null && "Hello"   // null (first falsy → returns it)
```

---

## 6. Is JavaScript Statically or Dynamically Typed?

**Dynamically typed** — type is checked at **runtime**, not compile time.

```js
var a = 23;         // number
var a = "Hello!";   // now it's a string — no error!
```

> Unlike TypeScript (static) where type mismatches are caught during compilation.

---

## 7. Passed by Value vs Passed by Reference

| Data Category | Passing Method |
|---------------|----------------|
| Primitive types | **By Value** (copy is made) |
| Non-primitive (Objects, Arrays) | **By Reference** (same memory address) |

```js
// --- Passed by VALUE (primitives) ---
var y = 234;
var z = y;    // z gets a NEW copy of 234
y = 23;
console.log(z); // still 234 — changes to y don't affect z

// --- Passed by REFERENCE (objects) ---
var obj = { name: "Vivek" };
var obj2 = obj;     // obj2 points to SAME memory

obj.name = "Akki";
console.log(obj2.name); // "Akki" — both point to same object!
```

---

## 8. What is `debugger` in JavaScript?

The `debugger` keyword **pauses execution** at that point in the browser DevTools:

```js
function calculateTax(income) {
  debugger; // 👈 execution pauses here when DevTools is open
  return income * 0.3;
}
```

> 💡 The browser must have DevTools open for `debugger` to have any effect.

---

## 9. What is `strict mode`?

`"use strict"` enables a stricter parsing mode that catches common mistakes.

```js
"use strict";

x = 23;          // ❌ ReferenceError — must declare first
delete Object.prototype; // ❌ not allowed

function test(a, a) { } // ❌ duplicate params not allowed
```

**Characteristics of strict mode:**
- No undeclared variables
- No duplicate function parameters
- No writing to read-only properties
- `this` inside a regular function is `undefined` (not global)

---

## ✅ Quick Revision Checklist

- [ ] 7 primitives + Object type
- [ ] Hoisting moves **declarations** (not initializations)
- [ ] `let`/`const` are in Temporal Dead Zone until declared
- [ ] `==` coerces types, `===` does not
- [ ] `typeof null === "object"` is a bug
- [ ] `NaN` has type `"number"`
- [ ] Primitives = pass by value, Objects = pass by reference
