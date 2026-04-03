# Part 7 — DOM Events, Debounce & Throttle

> **How browser events flow, and techniques to control how often your code runs.**

---

## 1. Event Bubbling

When an event fires on an element, it **travels upward** through parent elements.

```
Click on <button> inside <div> inside <body>
↓
button click fires
↓  (bubbles up)
div click fires
↓  (bubbles up)
body click fires
↓  (bubbles up)
document click fires
```

```js
document.querySelector("button").addEventListener("click", (e) => {
  console.log("button clicked!");
});

document.querySelector("div").addEventListener("click", (e) => {
  console.log("div clicked!"); // also fires when button is clicked
});
```

### Stopping bubbling

```js
button.addEventListener("click", (e) => {
  e.stopPropagation(); // ✅ stops the event from traveling up
  console.log("button clicked, no more bubbling");
});
```

---

## 2. Event Capturing

Events travel **down** from document to target before bubbling back up.

```
document → html → body → div → button  (capturing phase ↓)
button → div → body → html → document  (bubbling phase ↑)
```

Enable capturing with the third argument:

```js
// Third arg: { capture: true } enables capturing phase
document.querySelector("div").addEventListener(
  "click",
  (e) => console.log("div capturing!"),
  { capture: true } // or just pass `true`
);
```

> 💡 **Default is bubbling**. Capturing is rarely used in practice.

---

## 3. `event.target` vs `event.currentTarget`

| Property | Meaning |
|----------|---------|
| `event.target` | Element where the event **originated** |
| `event.currentTarget` | Element where the **listener is attached** |

```js
document.querySelector("ul").addEventListener("click", (e) => {
  console.log(e.target);         // the specific <li> that was clicked
  console.log(e.currentTarget);  // always the <ul>
});
```

---

## 4. Event Delegation

Instead of attaching listeners to every child, attach **one listener to the parent** and use `event.target` to identify which child triggered it.

```js
// ❌ Attaching listeners to every list item
document.querySelectorAll("li").forEach(li => {
  li.addEventListener("click", handleClick);
});

// ✅ Event delegation — one listener on parent
document.querySelector("ul").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
  }
});
```

**Why it matters:**
- ✅ Works for dynamically added elements
- ✅ Less memory (1 listener vs N listeners)
- ✅ Simpler cleanup

### Events that don't bubble (delegation won't work)

| Doesn't Bubble | Alternative |
|----------------|-------------|
| `focus` | `focusin` |
| `blur` | `focusout` |
| `scroll` | — |

---

## 5. Debouncing

**Debounce** delays function execution until the user **stops triggering** the event for a set time.

```
User types: k-e-y-b-o-a-r-d
Without debounce: 8 API calls 😱
With debounce (300ms): 1 API call ✅ (after last keystroke)
```

### How debounce works (step by step)

1. User triggers event → start a timer
2. User triggers again before timer ends → **cancel old timer, start new one**
3. Timer finally completes → **run the function**

```js
function debounce(fn, delay) {
  let timeoutId; // stored in closure

  return function() {
    const context = this;
    const args = arguments;

    clearTimeout(timeoutId);          // cancel previous timer

    timeoutId = setTimeout(function() {
      fn.apply(context, args);        // run after inactivity
    }, delay);
  };
}

// Usage: search input
const searchInput = document.getElementById("search");
const handleSearch = debounce(function(e) {
  console.log("Searching for:", e.target.value);
  // API call here
}, 300);

searchInput.addEventListener("input", handleSearch);
```

---

## 6. Throttling

**Throttle** ensures a function runs **at most once per time interval**, regardless of how many times the event fires.

```
User scrolls rapidly: fires 100 times per second
Without throttle: 100 function calls 😱
With throttle (200ms): ~5 function calls ✅
```

```js
function throttle(fn, limit) {
  let inThrottle = false;

  return function() {
    if (!inThrottle) {
      fn.apply(this, arguments);
      inThrottle = true;
      setTimeout(() => { inThrottle = false; }, limit);
    }
  };
}

// Usage: scroll handler
window.addEventListener("scroll", throttle(function() {
  console.log("Scroll position:", window.scrollY);
}, 200));
```

---

## 7. Debounce vs Throttle Comparison

| Aspect | Debounce | Throttle |
|--------|---------|---------|
| **When it runs** | After user STOPS triggering | Every N milliseconds |
| **Best for** | Search inputs, form validation | Scroll, resize, mousemove |
| **Goal** | Only care about final state | Regular sampling during activity |
| **Behavior** | Resets timer on every trigger | Ignores triggers within interval |

```
Debounce:  ----key----key-----------key--------[FIRE]
Throttle:  ----key----key-----[FIRE]--key--[FIRE]---key--[FIRE]
```

---

## 8. `stopPropagation` vs `stopImmediatePropagation`

```js
// stopPropagation — stops going UP to parent elements
button.addEventListener("click", (e) => {
  e.stopPropagation();
  // this listener runs, but event won't travel to parent div
});

// stopImmediatePropagation — stops ALL listeners on SAME element too
button.addEventListener("click", handler1);
button.addEventListener("click", (e) => {
  e.stopImmediatePropagation();
  // handler1 still ran, but any subsequent listeners on button won't
});
button.addEventListener("click", handler3); // ❌ won't run
```

---

## 9. Common DOM Event Types

```js
// Mouse events (bubble)
element.addEventListener("click", handler);
element.addEventListener("dblclick", handler);
element.addEventListener("mouseenter", handler); // doesn't bubble
element.addEventListener("mouseover", handler);  // bubbles

// Keyboard events (bubble)
document.addEventListener("keydown", handler);
document.addEventListener("keyup", handler);

// Form events
form.addEventListener("submit", handler);
input.addEventListener("input", handler);   // fires on every change
input.addEventListener("change", handler);  // fires on blur after change
input.addEventListener("focus", handler);   // doesn't bubble
input.addEventListener("focusin", handler); // bubbles

// Window events
window.addEventListener("resize", throttle(handler, 200));
window.addEventListener("scroll", throttle(handler, 200));
window.addEventListener("load", handler);
document.addEventListener("DOMContentLoaded", handler);
```

---

## ✅ Quick Revision Checklist

- [ ] Event bubbling: child → parent → document (default)
- [ ] Event capturing: document → parent → child (opt-in with `{capture:true}`)
- [ ] `event.target` = where event originated
- [ ] `event.currentTarget` = where listener is attached
- [ ] Event delegation: one parent listener handles all children
- [ ] Debounce: fires AFTER user stops (good for search inputs)
- [ ] Throttle: fires at most once per interval (good for scroll)
- [ ] `stopPropagation` = stop bubbling; `stopImmediatePropagation` = also stop other listeners on same element
