# Part 3 — Objects, Prototypes & Constructor Functions

> **How JavaScript builds and connects objects under the hood.**

---

## 1. Object Prototypes

Every JavaScript object **inherits properties and methods from a prototype**.

```
Your Array → Array.prototype → Object.prototype → null
Your Date  → Date.prototype  → Object.prototype → null
```

```js
var arr = [];
arr.push(2); // push is NOT on `arr` — it's on Array.prototype!

console.log(arr); // [2]
```

JS engine's lookup process:
1. Look for `push` on `arr` itself → ❌ not found
2. Look on `Array.prototype` → ✅ found! Run it.

> 💡 `Object.prototype` is at the **top of the chain**. Every object ultimately inherits from it.

### Prototype chain visualization

```js
var obj = {};
console.log(obj.toString()); // works! from Object.prototype

// Check the prototype
console.log(Object.getPrototypeOf(obj)); // Object.prototype
```

---

## 2. Constructor Functions

Constructor functions **create multiple objects with the same structure**.

> 📌 Convention: Constructor function names start with **Capital Letter** (Pascal Case).

```js
function Person(name, age, gender) {
  this.name = name;
  this.age = age;
  this.gender = gender;
}

// Create instances using `new`
var person1 = new Person("Vivek", 76, "male");
var person2 = new Person("Courtney", 34, "female");

console.log(person1); // Person { name: 'Vivek', age: 76, gender: 'male' }
```

### What does `new` do?

1. Creates a blank new object
2. Sets its prototype to `Person.prototype`
3. Runs the constructor with `this` = new object
4. Returns the new object

### Adding methods via prototype

```js
Person.prototype.greet = function() {
  return "Hi, I'm " + this.name;
};

person1.greet(); // "Hi, I'm Vivek"
person2.greet(); // "Hi, I'm Courtney"
// Method is shared — not duplicated for every instance!
```

---

## 3. ES6 Classes (Syntactic Sugar over Constructor Functions)

Classes are just a **cleaner syntax** for constructor functions + prototype methods.

```js
// Old way — Constructor Function
function Student(name, rollNumber, grade, section) {
  this.name = name;
  this.rollNumber = rollNumber;
  this.grade = grade;
  this.section = section;
}
Student.prototype.getDetails = function() {
  return `Name: ${this.name}, Roll: ${this.rollNumber}`;
};

// ES6 Class — same result, cleaner syntax
class Student {
  constructor(name, rollNumber, grade, section) {
    this.name = name;
    this.rollNumber = rollNumber;
    this.grade = grade;
    this.section = section;
  }

  getDetails() {
    return `Name: ${this.name}, Roll: ${this.rollNumber}`;
  }
}

let s1 = new Student("Vivek", 354, "6th", "A");
s1.getDetails(); // "Name: Vivek, Roll: 354"
```

### Key facts about ES6 Classes

- ❌ **Not hoisted** — cannot use before declaration
- ✅ Support `extends` for inheritance
- ✅ Always run in **strict mode** automatically

---

## 4. Prototypal vs Classical Inheritance

| Aspect | Classical (Java, C++) | Prototypal (JavaScript) |
|--------|-----------------------|-------------------------|
| Based on | Classes → Objects | Objects → Objects |
| Inheritance via | Class hierarchy | Object linking |
| Flexibility | Rigid, fixed hierarchy | Flexible, dynamic |
| JS mechanism | `extends` (class syntax) | `Object.create()` / prototype chain |

```js
// Prototypal inheritance with Object.create()
var animal = {
  breathe() { return "I breathe"; }
};

var dog = Object.create(animal); // dog's prototype = animal
dog.bark = function() { return "Woof!"; };

dog.breathe(); // "I breathe" — inherited!
dog.bark();    // "Woof!"
```

---

## 5. Prototype Design Pattern

The **Prototype Pattern** creates new objects by **cloning** a template object.

```js
// Template (prototype)
var carPrototype = {
  getModel() { return this.model; },
  getYear()  { return this.year; }
};

function createCar(model, year) {
  var car = Object.create(carPrototype);
  car.model = model;
  car.year = year;
  return car;
}

var car1 = createCar("Tesla Model S", 2023);
var car2 = createCar("BMW 3 Series", 2022);

car1.getModel(); // "Tesla Model S"
car2.getYear();  // 2022
```

---

## 6. What is DOM?

**DOM** = Document Object Model — a **programming interface** for HTML documents.

When the browser loads HTML, it creates a **tree of objects** (the DOM):

```
document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        └── p
```

```js
// Accessing DOM elements
document.getElementById("title")
document.querySelector(".card")
document.querySelectorAll("li")

// Changing DOM
document.getElementById("btn").textContent = "Clicked!";
document.body.style.backgroundColor = "lightblue";
```

---

## 7. What is BOM?

**BOM** = Browser Object Model — allows interaction with the **browser itself** (not just the page content).

Root object is **`window`**.

```js
window.location.href        // current URL
window.location.reload()    // refresh page
window.history.back()       // go back
window.screen.width         // screen dimensions
window.alert("Hello!")      // browser alert dialog
window.innerWidth           // viewport width
```

> 💡 Since `window` is global, you can call these without `window.`:
> `alert("Hi")` works the same as `window.alert("Hi")`

---

## 8. Client-Side vs Server-Side JavaScript

| Aspect | Client-Side JS | Server-Side JS (Node.js) |
|--------|---------------|--------------------------|
| Runs in | Browser | Server |
| Has access to | DOM, BOM, window | File system, databases, OS |
| Use case | UI interactions, animations | APIs, business logic, data |
| Examples | React, Vue, Vanilla JS | Express.js, NestJS, Fastify |
| Security | Code is visible to users | Code is hidden on server |

```js
// Client-side only
document.querySelector(".btn").addEventListener("click", () => {
  window.location.href = "/dashboard";
});

// Server-side only (Node.js)
const fs = require("fs");
fs.readFile("config.json", "utf8", (err, data) => {
  console.log(data);
});
```

---

## 9. `charAt()` Method

Returns the character at a specified index in a string.

```js
var str = "JavaScript";
str.charAt(0);  // "J"
str.charAt(4);  // "S"
str.charAt(99); // "" — index out of range
```

> Alternatively: `str[0]` does the same, but `charAt()` returns `""` for invalid index while `str[99]` returns `undefined`.

---

## ✅ Quick Revision Checklist

- [ ] Every JS object has a prototype chain ending at `Object.prototype`
- [ ] Constructor functions create multiple instances (capitalize name + use `new`)
- [ ] ES6 classes are syntactic sugar — same result as constructor functions
- [ ] `Object.create(proto)` creates an object with proto as its prototype
- [ ] DOM = tree of page elements; BOM = browser APIs (window, location, history)
- [ ] Client-side JS has DOM access; server-side JS has system access
