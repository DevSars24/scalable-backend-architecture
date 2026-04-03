# 03 — Optimization Hooks Simplified 🏎️

> **How to make your React app run faster.**

---

## 💾 useMemo — “Saving the Result”

### 🧠 One-Liner
**useMemo = A Calculator Cache.**  
(If you already calculated a result, don't do it again!)

### 🍝 Real-Life Analogy: The 10,000 Item List
- Imagine you are counting **10,000 items** on Zomato.
- Every time you type a letter in the search box (re-render), Zomato starts **counting again** (slow!).
- **useMemo** remembers: "The count was 5,000. Use that instead of counting again."

---

## 🛠️ useCallback — “Saving the Tool”

### 🧠 One-Liner
**useCallback = Sticking the Same Hammer in Your Pocket.**  
(Instead of making a new one every time, we reuse the same one.)

### 🍝 Real-Life Analogy: Reusable Tools
- React likes to **throw away its toolset** on every re-render and make a new one (new function references).
- **useCallback** tells React: "Keep this specific hammer. It’s perfect. Don't make a new one unless the toolbox (dependencies) changes."

---

## 👔 useReducer — “The Professional Manager”

### 🧠 One-Liner
**useReducer = A Shop Manager.**  
(If your state has too many rules, give it to a manager.)

### 🍝 Real-Life Analogy: The Toy Store
- **useState** is like a single kid managing a few toys.
- **useReducer** is a professional manager. You say, "Add item" or "Remove item" (the action). The manager **decides exactly how** the shelf should look (the logic).

---

## 🔍 The Difference: useMemo vs useCallback

| Hook | What it saves | Use for |
|------|---------------|---------|
| **useMemo** | The **Result** (Value) | Heavy data filtering, complex math |
| **useCallback** | The **Reference** (Function) | Passing functions to child components to prevent re-renders |
