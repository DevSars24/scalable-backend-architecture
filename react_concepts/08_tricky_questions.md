# Part 8 — Tricky Interview Questions & Deep Dives

> **The "Senior Level" section: Reconciliation, Stale Closures, and Debugging UI Bugs.**

---

## 1. Reconciliation & Diffing Algorithm

How does React actually decide what to update?

### The "Rules" of Diffing:
1. **Different Element Types**: If a `<div>` is replaced by a `<span>`, React tears down the entire tree and builds a new one. All state is lost.
2. **Same Element Types**: React keeps the DOM node and only updates the changed attributes (e.g., `className`, `style`).
3. **Component Elements**: If the component type is the same, React keeps the instance and updates the props.
4. **Recursing on Children**: By default, React iterates over both lists of children at the same time and generates a mutation whenever there’s a difference.

### 🔑 The Role of Keys
Keys tell React: "This item with ID `101` is the same one from the previous render, it just moved from index 0 to index 2."

**Without Keys:**
- Old: `[A, B]`
- New: `[C, A, B]`
- React thinks: "A changed to C, B changed to A, and a new B was added." (Inefficient!)

**With Keys:**
- Old: `{id:1:A, id:2:B}`
- New: `{id:3:C, id:1:A, id:2:B}`
- React thinks: "C is new. A and B just shifted down." (Efficient!)

---

## 2. Stale Closures in Hooks

A "Stale Closure" happens when a function (like an event handler or `useEffect` callback) captures variables from a previous render.

### 🛒 The Bug: Infinite Counter that stays at 1
```javascript
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      // ❌ count is captured as 0 when the effect first ran.
      // Every second it does: 0 + 1 = 1.
      setCount(count + 1); 
    }, 1000);
    return () => clearInterval(id);
  }, []); // Empty deps = only runs once

  return <h1>{count}</h1>;
}
```

### The Fixes:
1. **Functional Update**: `setCount(prev => prev + 1)` (React handles the state reference).
2. **Dependency Array**: Add `[count]` to deps (but this resets the interval every second).
3. **useRef**: Store the latest count in a ref.

---

## 3. Why `setState` is "Async" (Batching)

When you call `setCount(count + 1)`, the `count` variable doesn't change immediately.

```javascript
const handleClick = () => {
  setCount(count + 1);
  console.log(count); // ❌ Still the old value!
};
```

**Why?** Consistency and Performance.
- **Consistency**: If `count` updated immediately but the UI hadn't re-rendered yet, the code and UI would be out of sync.
- **Performance**: If you update 5 different states, React "batches" them into ONE re-render instead of five.

### React 18 Automatic Batching
In React 18, batching happens even inside `Promises`, `setTimeout`, and native events.

---

## 4. Re-renders: Myths vs Reality

### "My component re-renders because props changed!"
**Not exactly.** A component re-renders if its **Parent** re-renders. 

- Even if the child has NO props, it re-renders when the parent does.
- Unless you wrap the child in `React.memo()`.

### React.memo vs useMemo vs useCallback
- **React.memo**: Skips re-rendering a *Component* if props are same.
- **useMemo**: Skips re-computing a *Value* if deps are same.
- **useCallback**: Skips re-creating a *Function* if deps are same (important for `React.memo` children).

---

## 5. The `key={Math.random()}` Anti-pattern

Never do this.
```javascript
<li key={Math.random()}>Item</li>
```
**Result:** Every single re-render, React thinks every list item is brand new.
- It destroys the old DOM nodes.
- It recreates new DOM nodes.
- **Input focus is lost** (the keyboard closes on mobile) because the element you were typing in was deleted and replaced.

---

## 6. Portals: Escaping the DOM Hierarchy

Sometimes a child needs to be **visually** outside its parent (like a Modal or Tooltip) to avoid `z-index` or `overflow: hidden` issues.

```javascript
import { createPortal } from 'react-dom';

function Modal({ children }) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root') // Somewhere outside our App div
  );
}
```

---

## 7. `useEffect` twice in Dev (Strict Mode)

In React 18 Dev mode, `useEffect` runs:
1. Mount
2. Unmount (Immediate)
3. Mount again

**Why?** To force you to write clean **Cleanup functions**. 
- If your app breaks because an effect ran twice, your cleanup logic is likely missing (e.g., forgetting to `clearInterval` or `abort` a fetch).

---

## ✅ Quick Revision Checklist

- [ ] Reconciliation: Diffing algorithm + Keys.
- [ ] Stale Closures: Functions holding old state values.
- [ ] Batching: Scheduled updates for performance.
- [ ] Re-render vs Re-mount: Persistence of state.
- [ ] `React.memo`: Pure component equivalent for functional components.
- [ ] Portals: Rendering outside the parent DOM node.
- [ ] Strict Mode: Intentional double-rendering to catch bugs.
