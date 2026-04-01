# 🚀 TypeScript Interview Questions - The Real-World Guide

Yeh ek complete guide hai jisme TypeScript ke interview questions ko **real-world scenarios** aur **practical applications** ke saath explain kiya gaya hai. Isko padhne ke baad tum interview mein sirf definition nahi, balki yeh bata paoge ki *"yeh actual projects (jaise LinkedIn, Amazon, ya React apps) mein kaise use hota hai"*.

---

## 🟢 TypeScript Interview Questions for Freshers

### 1. Explain the data types available in TypeScript.
**Explanation:**
TypeScript mein do tarah ke data types hote hain:
- **Built-in:** `string`, `number`, `boolean`, `null`, `undefined`, `any`, `void`.
- **User-defined:** `array`, `enum`, `class`, `interface`.

**🔥 Real-World Scenario (Banking App):**
- `string` → User ka naam *"Saurabh"*
- `number` → Account balance `5000.50`
- `boolean` → Kya user verified hai? `true`
- `any` → Third-party payment gateway se unknown data return aana.
- `void` → Function jo sirf in-app notification dikhaaye, kuch return na kare.
- *Fayda:* Runtime pe "balance string mein aa gaya" jaisi bugs (app crash) nahi aayengi.

### 2. In how many ways we can declare variables in TypeScript?
**Explanation:**
- `var` → Function scoped (Avoid karo, purana tarika hai).
- `let` → Block scoped, jiski value change ho sakti hai.
- `const` → Block scoped, jiski value fix hoti hai.

**🔥 Real-World Scenario (React Dashboard):**
- `const API_URL = "https://api.example.com"` → Kabhi change nahi hoga (Security reasons).
- `let userCount = 0` → Loop ya API update pe count badhega.

### 3. How you can declare explicit variables in TypeScript?
**Explanation:** Variable ke baad colon (`:`) laga kar strongly type batana.
```typescript
let userAge: number = 24;
// userAge = "25"; // ❌ Compile time pe Error aayega
```
**🔥 Real-World Scenario (Netflix Profile):** User ki age me string dalne se rokta hai taaki frontend pe wrong data handle na karna pade.

### 4. How to declare a function with typed annotation in TypeScript?
**Explanation:** Function parameters aur uske return type ko explicitly batana.
**🔥 Real-World Scenario (E-commerce Cart - Flipkart):**
```typescript
function calculateTotal(price: number, quantity: number): number {
    return price * quantity;
}
```
*Galti se quantity string chali gayi toh app crash nahi hoga pehle hi error de dega.*

### 5. Describe the "any" type in TypeScript.
**Explanation:** TypeScript ki strict checking bypass karta hai. Koi bhi data type assign kar sakte hain.
**🔥 Real-World Scenario:** Stripe/Razorpay payment APIs implement karte time, jab shuru me response format exact nahi pata hota, tab `any` use hota hai as a temporary bridge.

### 6. What are the advantages of using TypeScript?
**Explanation:** Static typing, code autocompletion (IntelliSense), bug detection before runtime.
**🔥 Real-World Scenario:** LinkedIn aur Microsoft ne JS se TS me switch kiya as 50,000+ files ke codebase me bugs deployment se pehle hi pakde jate hain. 5x faster bug finding.

### 7. List some disadvantages of using TypeScript.
**Explanation:** Extra configuration step (`tsc` setup), syntax likhne mein zyada time lagta hai, browser directly nahi samajhta (pehle JS me compile karna padta hai).
**🔥 Real-World Scenario:** Startup me initial din me setup thoda time-consuming lagta hai par jab team me 10+ devs kaam karein, tab TS vardan lagta hai.

### 8. Explain the void type in TypeScript.
**Explanation:** Function se explicitly kuch return na hone ko `void` kehte hain.
**🔥 Real-World Scenario:**
```typescript
function sendPushNotification(userId: string): void {
    console.log(`Sending ping to ${userId}`); // Retuns nothing
}
```

### 9. What is type null and its use in TypeScript?
**Explanation:** `null` intentionally empty/no value ko darshata hai.
**🔥 Real-World Scenario:** API request lagai par data nahi aaya:
```typescript
function fetchUser(id: string): User | null { ... }
```

### 10-11. Syntax for creating objects & optional properties
**Explanation:** Object me properties set karna, aur `?` lagakar optional batana.
**🔥 Real-World Scenario (Instagram Profile):**
```typescript
const user: { name: string; age?: number } = {
    name: "Saurabh"
};
```
*Naya user abhi age nahi daala, code phir bhi error nahi dega.*

### 12. Explain the undefined type in TypeScript.
**Explanation:** Variable declared hai par aaj tak usko koi value verify nahi hui.
**🔥 Real-World Scenario:** Form mein user ne last name waala field blank chhod diya, toh wo `undefined` banke aayega backend process se pehle code handle kar lega.

### 13. Explain the behavior of arrays in TypeScript.
**Explanation:**
Ek Array mein specifically single data types ki entry limit karna.
**🔥 Real-World Scenario (Trello/Todo App):** 
```typescript
let taskList: string[] = ["Create Component", "Test API"];
// taskList.push(404); // Error dega!
```

### 14-15. How can you compile a TypeScript file? & Differentiate between .ts and .tsx
**Explanation:** 
- Compile command: `tsc filename.ts` → convert into `filename.js`.
- `.ts`: Standard TypeScript logic.
- `.tsx`: TS + JSX syntax (React components).
**🔥 Real-World Scenario:** React apps me components hamesha `.tsx` me banenge taaki HTML+JS mix handle ho sake smoothly.

### 16. What is "in" operator and why it is used?
**Explanation:** Yeh check karne ke liye ki object me koi property / module exist karti hai ya nahi.
**🔥 Real-World Scenario:** API se dynamic payload aaya hai.
`if ("role" in user)` se check karna ki user backend admin hai ya normal user.

### 17. Explain union types in TypeScript.
**Explanation:** Pipeline `|` symbol se ek variable ko 2 ya zyada data types combine karna.
**🔥 Real-World Scenario (Razorpay):**
```typescript
type PaymentMethod = "card" | "upi" | "wallet";
let method: PaymentMethod = "upi"; // Yeh hi 3 options strict allowed hain.
```

### 18. Explain type alias in TypeScript.
**Explanation:** Complex type signatures ko ek custom aasan naam dena.
```typescript
type UserId = string | number;
```

### 19-22. Static typing, template literals, arrow functions, optional parameters.
**🔥 Real-World Scenario (Cricbuzz App):**
```typescript
const showScore = (player: string, score?: number): void => {
    console.log(`Score of ${player} is ${score ? score : 'Not Batted'}`);
};
```
Arrow functions se syntax clean, template literal se string read karna easy.

### 23. Explain noImplicitAny in TypeScript.
**Explanation:** tsconfig file ki ek strict checking setting, jisse tum directly kisi ko bina type describe kare param send nahi kar sakte.
**🔥 Real-World Scenario:** Big tech teams (Microsoft) code quality enforce karne ke liye ise `true` rakhti hain.

### 24. What are interfaces in TypeScript?
**Explanation:** Structural checking ki contract, ek object me exactly yahi fields aana zaroori hai.
**🔥 Real-World Scenario (React Components Props):**
```typescript
interface ButtonProps {
    label: string;
    onClick: () => void;
    isDisabled?: boolean;
}
```

### 25. In how many ways you can use the for loop in TypeScript?
**Explanation:** Regular `for` loop, `for...in` (keys/index), `for...of` (values), array methods `.forEach()`.
**🔥 Real-World Scenario:** Admin Dashboard me User records map and data table pe iterate karane ke liye React me `Array.forEach` ya `map` loop.

---

## 🟡 TypeScript Intermediate Interview Questions

### 26. What is the `never` type and its uses?
**Explanation:** Ye represent karta hai un values ko jo "kabhi nahi aayengi" ya function "kabhi smoothly end nahi hoga".
**🔥 Real-World Scenario (API Error Handling):**
```typescript
function throwAPIError(msg: string): never {
    throw new Error(msg); // Function crash hoga, return kuch nahi.
}
```

### 27. Explain enums in TypeScript.
**Explanation:** Constants ko clearly numbers ya string words assign karna. Magic numbers se bachana.
**🔥 Real-World Scenario (Amazon Order Tracking):**
```typescript
enum Status { Pending = 1, Shipped, Delivered }
function trackOrder(id: string, state: Status) { ... }
```

### 28-29. Parameter destructuring & Type inference.
**Explanation:** Explicit objects break karna & automatically types guess kar lena basic values par.
**🔥 Real-World Scenario:** Redux reducer mein action payload destruct karna direct function call (`{ data, error }`) aur auto variable strings types samajhna.

### 30-32. Modules & `tsconfig.json`.
**Explanation:** Code ko multiple file system me break karke Export/Import karna aur `tsconfig.json` compiler behaviour handle karna.
**🔥 Real-World Scenario:** React Monorepo environment jahan API calls ek TS file module, config tsconfig me strict mode par rakha gaya.

### 33. What are Decorators in TypeScript?
**Explanation:** Meta-programming syntax `@` laga kar class/method behave manipulate karna runtime se pehle. 
**🔥 Real-World Scenario:** NestJS/Angular applications me endpoints pe `@Get()` ya validation route guards ke liye use of `@UseGuards()`.

### 34. How to debug a TypeScript file?
**Explanation:** `tsc --sourcemap filename.ts` chalane se `.map` ban jayegi, jis se directly developer tools / VS code Original TS file ko padh kar debug krega. 

### 35. Describe anonymous functions.
**Explanation:** Bina naam ke functions aur callbacks assign karna. 
**🔥 Real-World Scenario:** React ke button onClick event: `() => console.log('Clicked')`.

### 36. Calling constructor of Base Class from Child class?
**Explanation:** Yes, `super(params)` trigger karwa skte hain.
**🔥 Real-World Scenario:** Common Admin Panel Authentication ke basic logic Parent Class Auth ko provide karna using `super()` jabki child Dashboard login run kare.

### 38. Typeof operator.
**Explanation:** `typeof` operator current variable ki data-type detect karke TS system ko return karta hai. User definition me convert kar sakte ho type aliasing ke liye.

### 41. Explain Mixins in TypeScript.
**Explanation:** Multiple JS inheritance limit todhne ke liye Mixins design pattern. Ek class me alag alag characteristics inject karna.
**🔥 Real-World Scenario:** Game Dev System: Car class me `Flyable` and `Shootable` Mixin functions ek sath patch lagana bina direct heavy chain inheritance banaye.

### 42. Immutable Object properties.
**Explanation:** Using `readonly` properties for hardcoding restrictions in class objects.
**🔥 Real-World Scenario:** User environment configurations `{ readonly apiUrl: "..." }` to stop any accident change variable from anywhere in dev flow.

---

## 🔴 TypeScript Interview Questions For Experienced

### 43-44. Differences between Classes vs Interfaces
**Explanation:** 
- **Interface:** Bas Contract / Shape design batati hai, runtime me JS memory nahi leti (Gayab hojati hai compile k baad).
- **Classes:** Actual data implementation aur structure karti hain, memory store rakhti hai.
**🔥 Real-World Scenario:** Node.js Express controllers + React components me Response API schema banate samay Interface prefer hota hai. Business Logic services handling ke time Controller class setup hoti hai.

### 45-47. Visibility Methods (public/private/protected) in inheritance.
**Explanation:** OOP properties encapsulation handling restriction for Dev flow limit. 
**🔥 Real-World Scenario:** `AuthService` class me password verification function ko `private` tag dena padta hai taaki any unauthorized UI controller service external se access na kar paye. Child access ko `protected` banate hain.

### 48. Convert `.ts` file to TypeScript Definition File `.d.ts`?
**Explanation:** `tsc --declaration source.ts`. Sirf types exports hoti hain source data nahi.
**🔥 Real-World Scenario:** Apna custom npm package release karte samay (e.g. lodash library ki custom definitions .d.ts hoti hain).  

### 50. Explain Conditional Typing in TypeScript?
**Explanation:** JavaScript k advance level ke Ternary behavior types check karna basis upon input parameter. `T extends Something ? TypeA : TypeB`
**🔥 Real-World Scenario (Advanced Dynamic Component):** 
```typescript
type InputProps<T> = T extends string 
    ? { value: T; onChange: (val: T) => void } 
    : { data: T; onLoad: () => void };
```
*Ek Form element pe agar tum "Hello" pass kro to 'onChange', but Array input daalogey to 'onLoad' trigger expect karega. Bhot robust Type safe library banane me ye powerfull technique hai.*

---
**💡 Tips for Interviews:** 
Hamesha pehle basic meaning batao, fir turant ek "Real World example" do (jaise Ecommerce / Banking). Interviewer tumhe "experienced developer" maanega kyuki tum application ka architectural view rakhte ho. All the best for 2026 interviews! 💪
