# 🚀 TypeScript Interview Questions - Theory Practice

This document contains theoretical explanations for common TypeScript interview questions to help you build a solid foundation.

---

## 🟢 TypeScript Interview Questions for Freshers

### 1. Explain the data types available in TypeScript.
**Answer:** TypeScript supports both built-in and user-defined data types. 
- **Built-in types:** `string`, `number`, `boolean`, `null`, `undefined`, `any`, and `void`. 
- **User-defined types:** `array`, `enum`, `class`, and `interface`.

### 2. In how many ways we can declare variables in TypeScript?
**Answer:** Variables can be declared in three ways: 
- `var`: Function-scoped (Avoid using this due to hoisting issues).
- `let`: Block-scoped and mutable.
- `const`: Block-scoped and immutable.

### 3. How you can declare explicit variables in TypeScript?
**Answer:** By appending a colon (`:`) followed by the data type directly after the variable name (e.g., `let age: number = 24;`), ensuring strict typing.

### 4. How to declare a function with typed annotation in TypeScript?
**Answer:** By explicitly defining the data types for both the parameters and the return type of the function.

### 5. Describe the "any" type in TypeScript.
**Answer:** The `any` type bypasses TypeScript's strict type checking, allowing a variable to hold any data type. It is typically used as a temporary bridge for migrating legacy code or dealing with undocumented third-party API data.

### 6. What are the advantages of using TypeScript?
**Answer:**
- Static typing ensures type safety.
- Early bug detection at compile time.
- Improved autocompletion (IntelliSense) in modern IDEs.
- Highly scalable and easier to maintain for large codebases.

### 7. List some disadvantages of using TypeScript.
**Answer:**
- Requires an additional compilation step (transpiling to JavaScript).
- Has a steeper learning curve compared to standard JavaScript.
- Demands extra setup time and configuration (`tsconfig.json`).

### 8. Explain the void type in TypeScript.
**Answer:** The `void` type is used to indicate that a function does not implicitly return any value.

### 9. What is type null and its use in TypeScript?
**Answer:** `null` represents the intentional absence of any object value. It is often used to indicate that an object, variable, or data point is intentionally empty.

### 10-11. Syntax for creating objects & optional properties.
**Answer:** Objects can be explicitly typed within an interface or inline pattern. Properties can be designated as optional by appending a question mark (`?`) after the property name.

### 12. Explain the undefined type in TypeScript.
**Answer:** A variable that has been declared but not yet assigned a value is automatically of type `undefined`.

### 13. Explain the behavior of arrays in TypeScript.
**Answer:** Arrays restrict their elements to a specific, uniform data type. You cannot mix different data types in an array unless explicitly permitted via union types or `any`.

### 14-15. How can you compile a TypeScript file? & Differentiate between .ts and .tsx
**Answer:** 
- Use the `tsc filename.ts` compiler command. 
- The `.ts` extension is for standard TypeScript logic, while `.tsx` is for TypeScript files containing JSX (often used in React components).

### 16. What is the "in" operator and why it is used?
**Answer:** The `in` operator checks if a specific property exists within an object.

### 17. Explain union types in TypeScript.
**Answer:** Union types, denoted by the pipe (`|`) symbol, allow a variable to validly hold one of multiple predefined data types.

### 18. Explain type alias in TypeScript.
**Answer:** A type alias allows you to create a custom name for a type (often complex object shapes or union types) for easier readability and reuse.

### 19-22. Static typing, template literals, arrow functions, optional parameters.
**Answer:** 
- **Static typing:** Ensures variable type safety.
- **Template literals:** Use backticks (`` ` ``) for easy string interpolation. 
- **Arrow functions:** Provide a more concise syntax for functions. 
- **Optional parameters:** Marked with `?`, allowing a parameter to be omitted during a function call.

### 23. Explain noImplicitAny in TypeScript.
**Answer:** It is a strict compiler flag in `tsconfig.json` that forces an error if the compiler cannot infer a type and defaults to `any` due to a lack of explicit typing.

### 24. What are interfaces in TypeScript?
**Answer:** Interfaces act as structural contracts that define the exact properties and method signatures an object or class must contain.

### 25. In how many ways you can use the for loop in TypeScript?
**Answer:** Through the traditional `for` loop, `for...in` (iterates over keys/indexes), `for...of` (iterates over values), and array prototype methods like `.forEach()`.

---

## 🟡 TypeScript Intermediate Interview Questions

### 26. What is the never type and its uses?
**Answer:** The `never` type indicates values that will never occur. It is used for functions that never successfully complete, such as those that continually throw errors or enter infinite loops.

### 27. Explain enums in TypeScript.
**Answer:** Enums allow you to define a set of named constants (either numeric or string-based) to organize related values and remove "magic-numbers" from code.

### 28-29. Parameter destructuring & Type inference.
**Answer:** 
- **Parameter destructuring:** Unpacking object properties directly into variables during a function call. 
- **Type inference:** TypeScript's ability to automatically deduce the type of a variable upon assignment without explicit mapping.

### 30-32. Modules & tsconfig.json.
**Answer:** 
- **Modules:** Organize code into maintainable, separate files using `import` and `export` statements. 
- **tsconfig.json:** The root configuration file that controls compiler behavior, output paths, and strictness rules.

### 33. What are Decorators in TypeScript?
**Answer:** Decorators (prefixed with the `@` symbol) are a meta-programming paradigm used to observe, modify, or replace the behavior of classes, methods, accessor functions, or properties before runtime.

### 34. How to debug a TypeScript file?
**Answer:** Running `tsc --sourcemap filename.ts` creates a `.map` file. This allows developers to debug the original, readable TypeScript code directly in a browser or IDE.

### 35. Describe anonymous functions.
**Answer:** Functions declared without a defined identifier/name, typically used as temporary, inline arguments for callbacks or event listeners.

### 36. Calling constructor of Base Class from Child class?
**Answer:** A child class can trigger the base class's internal constructor by calling `super(params)` within its own constructor.

### 38. Typeof operator.
**Answer:** It returns the primitive data type of a variable as a string. In TypeScript context, it can also be used to capture and reuse exact types dynamically.

### 41. Explain Mixins in TypeScript.
**Answer:** Mixins are a design pattern used to bypass JavaScript's single-inheritance limitation. They allow a main class to inherit features/methods from multiple distinct, generic classes.

### 42. Immutable Object properties.
**Answer:** Properties can be locked and made immutable by prefixing them with the `readonly` keyword, preventing further reassignment post-initialization.

---

## 🔴 TypeScript Interview Questions For Experienced

### 43-44. Differences between Classes vs Interfaces.
**Answer:** 
- **Interfaces:** Define a pure shape or contract. They do not emit any JavaScript code upon compilation and consume zero runtime memory.
- **Classes:** Define contracts but also harbor concrete instances, data initialization, and implementation logic. They compile into functional JavaScript objects.

### 45-47. Visibility Methods (public/private/protected) in inheritance.
**Answer:** Access modifiers control variable visibility: 
- `public`: Accessible freely anywhere. 
- `private`: Accessible uniquely within its own defining class. 
- `protected`: Accessible in its defining class and any extending child classes.

### 48. Convert .ts file to TypeScript Definition File .d.ts?
**Answer:** Run `tsc --declaration source.ts`. This generates a `.d.ts` file containing solely the type definitions for distribution alongside JavaScript libraries.

### 50. Explain Conditional Typing in TypeScript?
**Answer:** Conditional types establish rules to evaluate and resolve definitions dynamically based on input types utilizing ternary-like syntax (`T extends U ? TrueType : FalseType`).
