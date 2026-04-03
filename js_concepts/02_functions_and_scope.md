# Part 2 — Functions, Scope & `this` Keyword

> **Understanding how JavaScript manages variables, functions, and execution context.**

---

## 1. Scope & Scope Chain

**Scope** determines where variables and functions are accessible in your code.

### 🔹 Three Types of Scope

**1. Global Scope** — accessible everywhere

```js
var globalVar = "I'm everywhere";

function sayHello() {
  return globalVar; // ✅ accessible
}
```

**2. Function / Local Scope** — accessible only inside the function

```js
function awesomeFunction() {
  var localVar = "only inside here";
  console.log(localVar); // ✅
}
console.log(localVar); // ❌ ReferenceError
```

**3. Block Scope** — `let` and `const` only, locked inside `{}`

```js
{
  let x = 45;
}
console.log(x); // ❌ ReferenceError

for (let i = 0; i < 2; i++) { /* ... */ }
console.log(i); // ❌ ReferenceError

// var ignores block scope!
for (var j = 0; j < 2; j++) { /* ... */ }
console.log(j); // ✅ 2
```

### 🔹 Scope Chain

When JS can't find a variable locally, it looks **outward** through parent scopes.

```js
var y = 24;

function favFunction() {
  var x = 667;

  var inner = function() {
    console.log(x); // ✅ found in parent (favFunction)
  };

  var anotherInner = function() {
    console.log(y); // ✅ found in global scope
  };

  inner();        // 667
  anotherInner(); // 24
}
```

> If a variable isn't found all the way up to global scope → **ReferenceError** is thrown.

---

## 2. Closures

A **closure** is when a function "remembers" variables from its outer scope even after the outer function has finished executing.

```js
function randomFunc() {
  var obj1 = { name: "Vivian", age: 45 };

  return function() {
    // ✅ Can still access obj1 even after randomFunc() is done
    console.log(obj1.name + " is awesome");
  };
}

var initialiseClosure = randomFunc(); // stores the returned function
initialiseClosure(); // "Vivian is awesome"
```

> 💡 `randomFunc()` has finished, but the returned function still holds a **reference** to `obj1` in memory — that's the closure.

### Real-world use case: Private counter

```js
function makeCounter() {
  var count = 0; // private variable
  return {
    increment() { count++; },
    decrement() { count--; },
    value()     { return count; }
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
console.log(counter.value()); // 2
// count is NOT directly accessible outside!
```

---

## 3. IIFE — Immediately Invoked Function Expression

An **IIFE** runs immediately when it is defined. Useful for avoiding global namespace pollution.

```js
(function() {
  var secret = "I'm scoped inside!";
  console.log("Runs immediately!");
})();

// 'secret' is NOT accessible here
```

**Why two sets of parentheses?**

- First `( )` → wraps the function so JS treats it as an **expression**, not a **declaration**
- Second `( )` → **invokes** (calls) the function immediately

```js
// ❌ This errors — function declaration needs a name
function() {
  console.log("error");
}

// ✅ This works — expression form
(function() {
  console.log("works!");
})();
```

---

## 4. The `this` Keyword

`this` refers to the **object that is calling the function**.

```js
// Case 1: Global context — `this` = window/global
function doSomething() {
  console.log(this); // Window object in browser
}
doSomething();

// Case 2: Object method — `this` = the object
var obj = {
  name: "vivek",
  getName: function() {
    console.log(this.name); // "vivek"
  }
};
obj.getName();

// Case 3: Method borrowed — `this` = new calling object
var getName = obj.getName;
var obj2 = { name: "akshay", getName };
obj2.getName(); // "akshay" — `this` is now obj2!

// Case 4: Missing property — results in error
var obj3 = { name: "akshay" };
obj3.getAddress(); // ❌ TypeError: obj3.getAddress is not a function
```

> 💡 **Trick**: Look at **what's before the dot** — that's what `this` refers to. No dot? → global object.

---

## 5. Arrow Functions & `this`

Arrow functions **do NOT have their own `this`**. They inherit `this` from the surrounding scope.

```js
var obj1 = {
  valueOfThis: function() {
    return this; // ✅ refers to obj1
  }
};

var obj2 = {
  valueOfThis: () => {
    return this; // ⚠️ refers to WINDOW, not obj2!
  }
};

obj1.valueOfThis(); // obj1
obj2.valueOfThis(); // Window
```

### Arrow function vs traditional function syntax

```js
// Traditional
var add = function(a, b) { return a + b; };

// Arrow — shorter
var arrowAdd = (a, b) => a + b;

// Single param — no parentheses needed
var double = num => num * 2;
```

---

## 6. call(), apply(), bind()

These three methods let you **manually set `this`**.

### `call()` — invoke with custom `this`, args separate

```js
function sayHello() {
  return "Hello " + this.name;
}

var obj = { name: "Sandy" };
sayHello.call(obj); // "Hello Sandy"

// With arguments:
function saySomething(message) {
  return this.name + " is " + message;
}
saySomething.call({ name: "John" }, "awesome"); // "John is awesome"
```

### `apply()` — same as call, but args as array

```js
saySomething.apply({ name: "John" }, ["awesome"]); // "John is awesome"
```

### `bind()` — returns a NEW function with `this` bound

```js
var bikeDetails = {
  displayDetails: function(reg, brand) {
    return this.name + " — " + reg + " , " + brand;
  }
};

var person1 = { name: "Vivek" };
var detailFn = bikeDetails.displayDetails.bind(person1, "TS0122", "Bullet");
detailFn(); // "Vivek — TS0122 , Bullet"
```

| Method | Arguments | Returns |
|--------|-----------|---------|
| `call()` | Separate | Result immediately |
| `apply()` | Array | Result immediately |
| `bind()` | Separate | New function |

---

## 7. Higher Order Functions

A **Higher Order Function** either:
- Takes a function as an argument, OR
- Returns a function

```js
// Takes a function as argument
function higherOrder(fn) {
  fn();
}
higherOrder(function() { console.log("Hello world"); });

// Returns a function
function higherOrder2() {
  return function() {
    return "Do something";
  };
}
var x = higherOrder2();
x(); // "Do something"
```

> 💡 HOFs are the foundation of functional programming. Common built-in HOFs: `map()`, `filter()`, `forEach()`, `reduce()`.

---

## 8. Self-Invoking Functions

Same concept as IIFE — function runs automatically.

```js
(function() {
  console.log("I run immediately and won't be called again!");
})();
```

Used when you need a one-time setup without polluting the global scope.

---

## 9. exec() vs test() — RegExp Methods

| Method | Returns | Use when... |
|--------|---------|-------------|
| `test()` | `true` / `false` | You just need to check if pattern exists |
| `exec()` | Match array or `null` | You need to extract the match |

```js
var pattern = /hello/i;

pattern.test("Hello World");  // true
pattern.exec("Hello World");  // ["Hello", index: 0, ...]
pattern.exec("Goodbye");      // null
```

---

## ✅ Quick Revision Checklist

- [ ] 3 scope types: Global, Function, Block
- [ ] Closure = function remembers outer scope variables
- [ ] IIFE = self-executing anonymous function
- [ ] `this` = the object calling the function
- [ ] Arrow functions inherit `this` from parent scope
- [ ] `call`/`apply` = invoke immediately, `bind` = return new function
- [ ] HOF = takes or returns a function
