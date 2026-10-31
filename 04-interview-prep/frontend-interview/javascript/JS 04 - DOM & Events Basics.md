# JS 04 — DOM & Events Basics

> [!info] Scope
> Selecting & updating elements · `classList` · `addEventListener` · The event object · Debugging with DevTools
> Previous: [[JS 03 - Arrays & Objects]] · Next: [[JS 05 - Core Concepts (Scope, Closures, this)]]
> Event bubbling/capturing in depth, delegation patterns, debounce/throttle and observers are in [[JS 11 - Browser & DOM Advanced]].

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| DOM | A live tree of objects the browser builds from HTML. JS reads and changes the page through it. |
| `querySelector` | Returns the **first** match or `null`. `querySelectorAll` returns a **static** `NodeList`. |
| `getElementsBy*` | Return **live** `HTMLCollection`s (no `forEach`/`map`). |
| `textContent` vs `innerHTML` | `textContent` is safe plain text. `innerHTML` parses HTML (XSS risk, destroys child listeners). |
| `classList` | `add`, `remove`, `toggle`, `contains`, `replace`. Prefer it over `className`. |
| `addEventListener` | Many listeners per event. Removing needs the **same function reference**. |
| `event.target` vs `currentTarget` | `target` = what was clicked. `currentTarget` = where the listener is attached. |
| `preventDefault` vs `stopPropagation` | One cancels the browser's default action, the other stops bubbling. They're independent. |
| Form `submit` | Call `e.preventDefault()` or the page reloads. |
| Input values | `input.value` is **always a string**. |
| DevTools | `console.log`, `debugger`, breakpoints, Elements and Network panels. Read the error message first. |

---

## 1. The DOM and selecting elements

### Definition
The **DOM (Document Object Model)** is a tree of objects representing the HTML document. Every tag is an **element node**, text is a **text node**, and `document` is the root. JavaScript uses the DOM API to find, change, add, and remove elements.

### Summary

| Method | Returns | Live? | Notes |
|---|---|---|---|
| `document.getElementById("id")` | Element or `null` | n/a | Fastest, no `#` in the argument |
| `document.querySelector(css)` | **First** match or `null` | n/a | Any CSS selector |
| `document.querySelectorAll(css)` | `NodeList` (maybe empty) | **No** (static) | Has `forEach`, `for...of`; no `map` |
| `document.getElementsByClassName("c")` | `HTMLCollection` | **Yes** | No `forEach`/`map` |
| `document.getElementsByTagName("p")` | `HTMLCollection` | **Yes** | |
| `el.closest(css)` | Nearest matching ancestor (or itself) or `null` | n/a | Great for event handling |
| `el.matches(css)` | `true` / `false` | n/a | Test an element against a selector |

```js
const title  = document.getElementById("title");
const btn    = document.querySelector(".btn.primary");
const items  = document.querySelectorAll("ul#list li");
const inForm = form.querySelector("input[name='email']");   // search within an element

items.forEach(li => console.log(li.textContent));
[...items].map(li => li.textContent);        // convert before using map
Array.from(items).filter(li => li.dataset.active === "true");
```

**Navigating the tree**
```js
el.parentElement;
el.children;               // HTMLCollection of child elements
el.firstElementChild;
el.lastElementChild;
el.nextElementSibling;
el.previousElementSibling;
```

**When does the script run?**

| How the script is loaded | DOM ready? |
|---|---|
| `<script>` in `<head>` (no attributes) | **No**. Elements are `null`. |
| `<script defer>` / `type="module"` | Yes, runs after parsing |
| `<script>` at end of `<body>` | Yes |
| Inside `DOMContentLoaded` listener | Yes |

```js
document.addEventListener("DOMContentLoaded", () => { /* DOM is ready */ });
```

### Edge cases
- **`null` results:** `querySelector` / `getElementById` return `null` when nothing matches. Using the result then throws `TypeError: Cannot read properties of null`. Check the selector and the script timing.
- **Live vs static:** a `getElementsByClassName` result updates as the DOM changes, while a `querySelectorAll` result is a snapshot.
  ```js
  const live = document.getElementsByClassName("i");
  const snap = document.querySelectorAll(".i");
  // add one more .i element...
  live.length;  // grows
  snap.length;  // unchanged
  ```
- **`NodeList` isn't an array:** it has `forEach`, `entries`, `for...of`, but no `map`/`filter`/`reduce`. Use `[...nodeList]` or `Array.from`.
- **Invalid selectors throw** `SyntaxError`. IDs starting with a digit (`#1abc`) need escaping: `` `#${CSS.escape(id)}` ``.
- **Duplicate IDs** are invalid HTML; `getElementById` returns the first one.
- **Element IDs become global variables** (`window.title`). Don't rely on it.
- `children` only has elements, `childNodes` also contains text and comment nodes (including whitespace).
- `querySelector` on an element only searches its **descendants**, but the selector may still mention ancestors.
- `document.querySelector("input").value` fails if the input doesn't exist. Use optional chaining (`?.`) or a guard.

### Use cases
- Grabbing form fields, buttons, containers, and list items to attach behavior.
- `closest()` to find the card/row an inner button belongs to.
- Scoped searches (`card.querySelector(".price")`) in repeated UI blocks.

---

## 2. Updating elements

### Definition
Updating the DOM means changing an element's **content**, **attributes**, **styles**, or **structure** (adding, moving, removing nodes).

### Summary

**Content**

| Property | Reads / writes | Notes |
|---|---|---|
| `textContent` | All text (including hidden) | **Safe**; no HTML parsing; fast |
| `innerText` | Rendered text (respects CSS) | Triggers layout; ignores `display:none` text |
| `innerHTML` | HTML string | Parses HTML. **XSS risk** with untrusted data. Rewrites children |
| `outerHTML` | Element + children as HTML | Replaces the element itself when set |
| `value` | Form control's current value | Always a string |

```js
title.textContent = "Hello";
box.innerHTML = "<strong>Bold</strong>";   // only with trusted/sanitized strings
input.value = "";
```

**Attributes, properties, dataset, styles**
```js
img.setAttribute("alt", "Logo");
img.getAttribute("alt");
img.removeAttribute("hidden");
el.hasAttribute("disabled");

btn.disabled = true;                 // property (boolean)
checkbox.checked = true;

card.dataset.userId = "42";          // <div data-user-id="42">
card.dataset.userId;                 // "42" (always a string)

el.style.backgroundColor = "tomato"; // camelCase, inline style
el.style.setProperty("--gap", "8px");
getComputedStyle(el).fontSize;       // final computed value, read-only
el.hidden = true;                    // uses the hidden attribute
```

**Creating, inserting, removing**

| Method | What it does |
|---|---|
| `document.createElement("li")` | Creates an element (not in the page yet) |
| `parent.append(a, "text", b)` | Adds multiple nodes/strings at the end |
| `parent.appendChild(node)` | Adds one node, returns it |
| `parent.prepend(node)` | Adds at the start |
| `el.before(node)` / `el.after(node)` | Inserts next to `el` |
| `el.replaceWith(node)` | Replaces `el` |
| `el.remove()` | Removes `el` from the DOM |
| `el.cloneNode(true)` | Deep copy (listeners are **not** copied) |
| `parent.insertAdjacentHTML("beforeend", html)` | Adds HTML without re-parsing existing children |

```js
const li = document.createElement("li");
li.textContent = "New task";
li.classList.add("task");
list.append(li);

// Many items at once → build offscreen, insert once
const frag = document.createDocumentFragment();
tasks.forEach(t => {
  const li = document.createElement("li");
  li.textContent = t.title;
  frag.append(li);
});
list.append(frag);
```

### Edge cases
- **`innerHTML` + untrusted input = XSS:** `el.innerHTML = userInput` can run `<img onerror=...>`. Use `textContent`, or sanitize.
- **`innerHTML +=` is wasteful and destructive:** it re-parses the whole content and **removes event listeners** attached to child elements.
- **`textContent` replaces all children:** setting it wipes any nested elements.
- **`textContent = 0`** displays `"0"`, but `if (value) el.textContent = value` skips `0`. Check against `null`/`undefined` instead.
- **Attribute vs property:** `a.href` (property) returns the **absolute URL**; `a.getAttribute("href")` returns the raw value (`"/home"`). `input.value` is the current value; the `value` attribute is the default.
- **`dataset` converts names:** `data-user-id` → `dataset.userId`. Values are always strings (`"42"`, `"true"`).
- **Style properties are camelCase:** `el.style.background-color` is invalid. `el.style` only reads **inline** styles; use `getComputedStyle` for final values.
- **`appendChild` vs `append`:** `append` takes multiple nodes and strings; `appendChild` takes exactly one node. Appending an already-attached node **moves** it.
- **`cloneNode(true)` doesn't copy listeners** added via `addEventListener`.
- **Layout thrashing:** alternately reading layout (`offsetHeight`) and writing styles in a loop forces repeated reflows. Batch reads, then writes.
- A newly created element isn't visible until you insert it.

### Use cases
- Rendering API results into a list or table.
- Showing/hiding UI (`hidden`, class toggles), updating counters and status text.
- Form feedback (error messages next to inputs), live previews.

---

## 3. `classList`

### Definition
`element.classList` is a `DOMTokenList` for reading and changing an element's CSS classes without string manipulation.

### Summary

| Method | Purpose | Returns |
|---|---|---|
| `add("a", "b")` | Add one or more classes | `undefined` |
| `remove("a")` | Remove class(es) | `undefined` |
| `toggle("a")` | Add if missing, remove if present | `true` if now present |
| `toggle("a", force)` | `force = true` → add, `false` → remove | the `force` value |
| `contains("a")` | Check presence | `true` / `false` |
| `replace("old", "new")` | Swap a class | `true` if replaced |
| `length`, `item(i)` | Count / read by index | |

```js
btn.classList.add("active", "large");
btn.classList.remove("large");
btn.classList.toggle("open");               // flips and returns the new state
btn.classList.toggle("dark", isDarkMode);   // set explicitly from a boolean
if (btn.classList.contains("active")) {}
btn.classList.replace("primary", "secondary");
```

**`className` vs `classList`**

| | `className` | `classList` |
|---|---|---|
| Type | One string of all classes | Token list with methods |
| Change one class | Manual string editing | `add` / `remove` / `toggle` |
| Risk | Overwrites existing classes | Safe, leaves other classes alone |

### Edge cases
- `el.className = "x"` **replaces every class**. Use `classList.add`.
- `classList.add("a b")` (space inside one argument) throws `InvalidCharacterError`; pass separate arguments: `add("a", "b")`. An empty string throws `SyntaxError`.
- `add` an existing class is a no-op (no duplicates).
- `toggle` returns whether the class is **now present**, not whether it changed.
- Class names are **case-sensitive** in selectors (except in quirks mode).
- `classList` doesn't help with **inline styles**. Use `el.style` for one-offs.
- Toggling a class that doesn't exist in your CSS does nothing visually. Check spelling in DevTools.

### Use cases
- Dark mode, open/closed menus, active tab, error/valid input states, loading spinners.
- Preferred approach: keep **styling in CSS classes** and let JS only toggle the classes.

---

## 4. `addEventListener`

### Definition
`target.addEventListener(type, handler, options)` registers a function (the **handler** or **listener**) to run when an event of that `type` happens on `target`.

### Summary

```js
const btn = document.querySelector("#save");

function handleClick(event) {
  console.log("clicked", event.type);
}

btn.addEventListener("click", handleClick);
btn.removeEventListener("click", handleClick);    // needs the SAME function reference

btn.addEventListener("click", () => {}, { once: true });  // runs one time, then removes itself
```

**Options (third argument)**

| Option | Meaning |
|---|---|
| `once: true` | Auto-remove after the first call |
| `capture: true` | Listen during the capture phase instead of the bubbling phase |
| `passive: true` | Promise not to call `preventDefault()` (smoother scrolling) |
| `signal: abortController.signal` | Remove the listener when the controller aborts |

**`addEventListener` vs `onclick`**

| | `addEventListener` | `el.onclick = fn` |
|---|---|---|
| Multiple handlers | Yes | No (new assignment overwrites) |
| Options (once, capture, passive) | Yes | No |
| Remove | `removeEventListener` | Set to `null` |
| Inline HTML `onclick="..."` | n/a | Avoid (mixes markup and logic, CSP issues) |

**Common events**

| Event | Fires when | Bubbles? |
|---|---|---|
| `click`, `dblclick` | Mouse/touch/keyboard activation | Yes |
| `input` | Value changes (every keystroke) | Yes |
| `change` | Value committed (text: on blur; checkbox/select: immediately) | Yes |
| `submit` | Form submitted (on the `<form>`) | Yes |
| `keydown`, `keyup` | Key pressed / released | Yes |
| `focus`, `blur` | Element gains / loses focus | **No** |
| `focusin`, `focusout` | Same, but bubbling versions | Yes |
| `mouseenter`, `mouseleave` | Pointer enters / leaves | **No** |
| `mouseover`, `mouseout` | Pointer enters / leaves (incl. children) | Yes |
| `DOMContentLoaded` | HTML parsed and DOM ready | n/a (on `document`) |
| `load` | Page and resources fully loaded | n/a |

**Cleanup with `AbortController`**
```js
const ac = new AbortController();
window.addEventListener("resize", onResize, { signal: ac.signal });
document.addEventListener("keydown", onKey, { signal: ac.signal });
ac.abort();   // removes both listeners at once
```

**Event flow (preview)**
1. **Capture:** `window` → ... → parent → target
2. **Target:** the element itself
3. **Bubble:** target → parent → ... → `window` *(default phase for listeners)*

### Edge cases
- **Anonymous functions can't be removed:** `removeEventListener("click", () => {})` removes nothing, since it's a different function object. Store the handler in a variable.
- **`capture` must match** when removing: `add(..., fn, true)` is removed with `remove(..., fn, true)`.
- **The same function + type + capture added twice runs once.** Different function objects run separately.
- **Calling instead of passing:** `addEventListener("click", handle())` runs `handle` immediately and registers its **return value**. Pass `handle`, not `handle()`.
- **`this` inside the handler:** a regular function gets `this === event.currentTarget`; an arrow function uses the outer `this`.
- **Listeners can leak:** adding listeners on `window`/`document` in a component and never removing them keeps old components alive (a common React `useEffect` cleanup bug).
- **`passive: true` + `preventDefault()`** does nothing and logs a console warning.
- **Listeners run in registration order.** `stopImmediatePropagation()` stops later listeners on the same element.
- **Elements removed from the DOM** with listeners are garbage-collected, unless something else references them.
- **Replaced `innerHTML` removes child listeners.** Attach after rendering, or use delegation ([[JS 11 - Browser & DOM Advanced]]).
- **Buttons inside a `<form>`** default to `type="submit"`. Add `type="button"` to avoid accidental submits.
- **`click` also fires from the keyboard** (Enter/Space on buttons and links), which is why `click` is preferred over `mousedown` for accessibility.

### Use cases
- Button clicks, form submit and validation, live input feedback (`input`).
- Keyboard shortcuts (`keydown` on `document`), closing modals on `Escape`.
- Window events (`resize`, `scroll`), cleanup with `AbortController`.

---

## 5. The event object

### Definition
When an event fires, the browser passes an **event object** (conventionally `e` or `event`) to the handler. It describes what happened and provides methods to control it.

### Summary

| Property / method | Meaning |
|---|---|
| `e.type` | Event name (`"click"`) |
| `e.target` | The element that **triggered** the event (deepest element clicked) |
| `e.currentTarget` | The element the **listener is attached to** |
| `e.preventDefault()` | Cancel the browser's default action |
| `e.stopPropagation()` | Stop the event from reaching other elements (bubbling/capturing) |
| `e.stopImmediatePropagation()` | Also stop other listeners on the same element |
| `e.defaultPrevented` | Was `preventDefault()` called? |
| `e.bubbles` / `e.cancelable` | Whether it bubbles / can be cancelled |
| `e.eventPhase` | 1 capture, 2 target, 3 bubble |
| `e.isTrusted` | `true` if triggered by a real user action, `false` if from code |
| `e.timeStamp` | When it happened (ms) |

**Mouse, keyboard, form specifics**
```js
// Mouse
e.clientX, e.clientY      // position in the viewport
e.pageX, e.pageY          // position in the page (includes scroll)
e.button                  // 0 left, 1 middle, 2 right
e.shiftKey, e.ctrlKey, e.altKey, e.metaKey

// Keyboard
e.key       // "Enter", "a", "Escape", "ArrowUp"  (what the key produces)
e.code      // "Enter", "KeyA", "Escape"          (physical key position)
e.repeat    // true if the key is held down

// Form / input
e.target.value      // string
e.target.checked    // checkbox / radio
e.target.name
```

**`target` vs `currentTarget`**
```html
<button id="b"><span>Save</span></button>
```
```js
b.addEventListener("click", (e) => {
  console.log(e.target.tagName);         // "SPAN" if the span was clicked, else "BUTTON"
  console.log(e.currentTarget.tagName);  // always "BUTTON"
});
```

**`preventDefault` vs `stopPropagation`**

| | `preventDefault()` | `stopPropagation()` |
|---|---|---|
| Stops | The browser's default behavior | The event travelling to other elements |
| Examples | Form submit reload, link navigation, checkbox toggle | Parent's click handler not running |
| Affects other listeners? | No | Yes (on ancestors/descendants) |
| Independent? | Yes, use one, both, or neither | |

**Form submission pattern**
```js
form.addEventListener("submit", (e) => {
  e.preventDefault();                                   // stop page reload
  const data = Object.fromEntries(new FormData(form));  // inputs need name="..."
  console.log(data);                                    // { email: "...", password: "..." }
});
```

### Edge cases
- **`e.target` may be a child element,** such as the `<span>` inside a button or an icon inside a link. Use `e.currentTarget` or `e.target.closest("button")` when you need the element you attached to.
- **`currentTarget` is `null` after the handler finishes** (e.g. read inside `setTimeout` or after `await`). Save it first: `const el = e.currentTarget;`.
- **`input` vs `change`:** `input` fires on every keystroke; `change` fires when the value is committed (blur/Enter for text inputs).
- **`e.key` vs `e.code`:** `key` depends on layout/shift (`"a"` vs `"A"`, language); `code` is the physical key. `keypress` is deprecated; use `keydown`.
- **Arrow functions and `this`:** `this` isn't the element inside an arrow handler. Use `e.currentTarget`.
- **`preventDefault` only works on cancelable events** and not in `passive` listeners.
- **`stopPropagation` overuse** breaks analytics, dropdown-close logic, and delegation on parents. Prefer checking `e.target`.
- **Value types:** `e.target.value` is a string even for `type="number"`. Use `valueAsNumber` or `Number()`.
- **Checkboxes:** read `checked`, not `value`. Unchecked boxes are **omitted** from `FormData`.
- **Synthetic events (React):** React wraps native events. `e.nativeEvent` is the original.
- **Links:** `preventDefault()` on `click` stops navigation. Don't forget keyboard users (Enter on a focused link triggers `click`).

### Use cases
- Reading what the user typed/clicked, validating forms, building keyboard shortcuts.
- Stopping default form submission and sending data via `fetch`.
- Finding which list item was clicked via `e.target.closest("li")`.

---

## 6. Debugging with DevTools

### Definition
**DevTools** are the built-in browser tools (Chrome, Edge, Firefox, Safari) for inspecting the DOM, running code, setting breakpoints, and examining network and storage. Open with **F12**, **Ctrl+Shift+I** (Windows/Linux), or **Cmd+Opt+I** (Mac).

### Summary

| Panel | What it's for |
|---|---|
| **Elements** | Inspect/edit the live DOM and CSS, see computed styles and box model, force `:hover`/`:focus` states, view event listeners |
| **Console** | Run JS, see logs/errors/warnings, use helper commands |
| **Sources** | Read source files, set breakpoints, step through code, watch variables |
| **Network** | Requests/responses, status codes, headers, payload, timing, throttling |
| **Application** | `localStorage`, `sessionStorage`, cookies, IndexedDB, service workers |
| **Performance / Lighthouse** | Profile slowness, audit performance/accessibility/SEO |

**Console methods**

| Method | Use |
|---|---|
| `console.log(x)` | General output |
| `console.warn(x)` / `console.error(x)` | Warning / error styling with stack trace |
| `console.table(arr)` | Array of objects as a table |
| `console.dir(el)` | Show an element as an object (not HTML) |
| `console.group()` / `groupEnd()` | Collapse related logs |
| `console.time("t")` / `timeEnd("t")` | Measure duration |
| `console.count("label")` | Count calls |
| `console.assert(cond, msg)` | Log only if `cond` is false |
| `console.trace()` | Print the call stack |

**Console helpers (only inside the DevTools console)**
```js
$0                    // the element currently selected in the Elements panel
$("sel") / $$("sel")  // shortcuts for querySelector / querySelectorAll (array)
getEventListeners($0) // list listeners attached to an element (Chrome)
monitorEvents($0, "click")   // log events as they happen
copy(obj)             // copy a value to the clipboard
```

**Breakpoints (Sources panel)**

| Breakpoint | How | Use it when |
|---|---|---|
| Line | Click the line number | Pause at a specific line |
| Conditional | Right-click line → *Add conditional breakpoint* | Pause only if e.g. `i === 5` |
| Logpoint | Right-click → *Add logpoint* | Log without editing code |
| `debugger;` statement | Put it in your code | Quick pause (remove afterwards) |
| Event listener | *Event Listener Breakpoints* → Mouse → click | Find which code runs on a click |
| DOM change | Elements → right-click node → *Break on* | Find what modifies an element |
| XHR/fetch | *XHR/fetch Breakpoints* | Pause when a request is sent |

**Stepping controls**

| Action | Shortcut | Meaning |
|---|---|---|
| Resume | F8 | Run until next breakpoint |
| Step over | F10 | Run the line, don't enter functions |
| Step into | F11 | Enter the called function |
| Step out | Shift+F11 | Finish current function |

While paused, check the **Scope** pane (local/closure/global variables), **Call Stack**, and add **Watch** expressions.

**Reading common errors**

| Error message | Usual cause |
|---|---|
| `Cannot read properties of null (reading 'addEventListener')` | Selector matched nothing, or the script ran before the DOM loaded |
| `Cannot set properties of null (setting 'textContent')` | Same: element is `null` |
| `x is not defined` (ReferenceError) | Typo, wrong scope, variable declared later or in another file |
| `x is not a function` (TypeError) | Wrong name/type; calling something that isn't a function (e.g. `map` on a `NodeList`) |
| `Unexpected token ...` (SyntaxError) | Syntax mistake (missing bracket/comma) |
| `... is not a valid selector` | Invalid CSS selector string |
| `Failed to load resource: 404` / CORS error | Wrong URL / missing server CORS headers (see Network tab) |

**Debugging workflow when "my click does nothing"**
1. Open **Console**: any red errors? Read the message and the file:line link.
2. Is the element found? `console.log(document.querySelector("#save"))` should not be `null`.
3. Is the listener attached? In Elements, select the node → **Event Listeners** tab (or `getEventListeners($0)`).
4. Put a `console.log` (or logpoint) at the start of the handler. Does it run?
5. Add a breakpoint and step through. Check variable values in **Scope**.
6. Is another element covering it (overlay, `z-index`, `pointer-events: none`)? Inspect the clicked spot.
7. Check **Network** if the issue involves an API call: status, response body, request payload.

### Edge cases
- **`console.log(obj)` shows the object's state at the time you *expand* it,** not when it was logged (Chrome keeps a live reference). To snapshot: `console.log(structuredClone(obj))` or `JSON.stringify(obj)`.
- **Minified code is unreadable** without source maps. Enable source maps in the build/dev setup.
- **`console.log` in production** leaks info and slows hot paths. Remove or guard.
- **`debugger` does nothing** when DevTools is closed. Don't commit it.
- **Cached files:** enable *Disable cache* (Network tab, while DevTools is open) to avoid stale JS/CSS.
- **Edits in the Elements panel** are temporary. They disappear on reload (use *Workspaces/overrides* to persist).
- **Network tab:** enable *Preserve log* to keep requests across page navigations or form submits.
- **`async` stack traces:** errors in callbacks may lose context. Use the **Async** call stack checkbox in Sources.
- **Mobile debugging:** use device emulation (Ctrl+Shift+M) for layout, but test real devices for touch and performance.
- **Extensions** can inject scripts/errors into the console. Test in an incognito window if something looks odd.

### Use cases
- Finding why a selector returns `null`, why a class isn't applying, or why a handler never fires.
- Inspecting API responses and status codes; checking `localStorage` values.
- Stepping through logic bugs; measuring slow functions with `console.time`.
- Interview talking point: describe your debugging process (reproduce → read error → isolate → breakpoint → fix → verify).

---

## 7. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer.

### Basic

> [!question]- What is the DOM?
> The Document Object Model is the browser's tree-shaped, in-memory representation of the HTML page. JavaScript can read and modify elements, attributes, text, and styles through it.

> [!question]- Difference between `getElementById`, `querySelector`, and `querySelectorAll`?
> `getElementById` finds one element by ID (or `null`). `querySelector` returns the first element matching any CSS selector (or `null`). `querySelectorAll` returns a static `NodeList` of all matches.

> [!question]- What does `querySelector` return if nothing matches?
> `null`. Accessing a property on it throws `TypeError`.

> [!question]- `textContent` vs `innerText` vs `innerHTML`?
> `textContent` gets/sets all raw text (fast, safe). `innerText` is layout-aware (ignores hidden text, triggers reflow). `innerHTML` gets/sets HTML markup (risk of XSS, rebuilds children).

> [!question]- How do you add, remove, and toggle a CSS class?
> `el.classList.add("x")`, `el.classList.remove("x")`, `el.classList.toggle("x")`.

> [!question]- How do you create and insert an element?
> `const el = document.createElement("div"); el.textContent = "Hi"; parent.append(el);`

> [!question]- What is `addEventListener` and why prefer it over `onclick`?
> It registers a handler for an event. You can attach multiple handlers, use options (`once`, `capture`, `passive`, `signal`), and remove them with `removeEventListener`. `onclick` allows only one handler per event.

> [!question]- What is the event object?
> An object passed to the handler with details about the event (`type`, `target`, `currentTarget`, `key`, `clientX`, ...) and control methods (`preventDefault`, `stopPropagation`).

> [!question]- What does `preventDefault()` do?
> It cancels the browser's default action for the event (e.g., form submission reload, link navigation).

> [!question]- How do you read an input's value?
> `input.value` (or `e.target.value` in an `input`/`change` handler). It is always a string.

### Intermediate

> [!question]- `event.target` vs `event.currentTarget`?
> `target` is the element that originated the event (could be a child like a `<span>` in a button). `currentTarget` is the element the listener is attached to.

> [!question]- `preventDefault` vs `stopPropagation`?
> `preventDefault` stops the browser's default behavior; `stopPropagation` stops the event from bubbling/capturing to other elements. They're independent.

> [!question]- Live `HTMLCollection` vs static `NodeList`?
> `getElementsBy*` return live collections that update with the DOM. `querySelectorAll` returns a static snapshot. Live collections have no `forEach`/`map`; convert them with `Array.from` or spread.

> [!question]- Why can't you `map` over `querySelectorAll` results?
> A `NodeList` isn't an array. It has `forEach` but not `map`. Use `[...nodeList].map(...)` or `Array.from(nodeList, fn)`.

> [!question]- Why does `removeEventListener` sometimes not work?
> It needs the exact same function reference (and same `capture` flag). Anonymous/arrow functions defined inline create a new reference each time, so they can't be removed. Store the handler in a variable (or use `{ signal }`).

> [!question]- `input` vs `change` event?
> `input` fires on every value change (each keystroke). `change` fires when the value is committed (on blur for text inputs; immediately for checkboxes, radios, and selects).

> [!question]- What's the difference between `key` and `code` in keyboard events?
> `key` is the character/function produced (`"a"`, `"A"`, `"Enter"`) and depends on layout/modifiers. `code` is the physical key (`"KeyA"`) regardless of layout.

> [!question]- Why does my form reload the page when I click submit?
> The default `submit` action navigates/reloads. Call `e.preventDefault()` in the `submit` handler (attach it to the form, not the button click).

> [!question]- Why does `innerHTML +=` cause problems?
> It reserializes and re-parses existing content, which destroys child event listeners/state and is slower. Use `append`/`insertAdjacentHTML` or create elements.

> [!question]- Attribute vs property?
> Attributes are what's written in the HTML (`getAttribute`); properties are the live DOM object values (`el.value`, `el.checked`, `a.href`). They can differ, e.g. `a.href` is absolute while the attribute is raw.

> [!question]- How do you stop a listener from firing more than once?
> Pass `{ once: true }`, or remove it manually inside the handler.

> [!question]- When does the DOM become available to your script?
> After parsing. Use `defer`, put the script at the end of `<body>`, or wait for `DOMContentLoaded`. A plain `<script>` in `<head>` runs before elements exist.

### Tricky / follow-up

> [!question]- What happens if you write `addEventListener("click", handle())`?
> `handle()` runs immediately and its **return value** (often `undefined`) is registered as the handler, so nothing runs on click. Pass `handle` without parentheses.

> [!question]- If a `<button>` contains a `<span>` and you click the span, what is `e.target`?
> The `<span>`. `e.currentTarget` is the button (if the listener is on the button). Use `closest("button")` to normalize.

> [!question]- Why is `e.currentTarget` `null` inside a `setTimeout`?
> It's only set while the event is being dispatched. Capture it in a variable first.

> [!question]- Does `cloneNode(true)` copy event listeners?
> No. Listeners added with `addEventListener` aren't cloned (inline `onclick` attributes are, since they're part of the markup).

> [!question]- How can you avoid XSS when inserting user text?
> Use `textContent` (or create text nodes) instead of `innerHTML`. If HTML is required, sanitize it (e.g., DOMPurify) and never trust user input.

> [!question]- Why would `focus` event delegation not work, and what's the alternative?
> `focus`/`blur` don't bubble. Use `focusin`/`focusout` (they bubble) or capture-phase listeners.

> [!question]- How would you debug "Cannot read properties of null (reading 'addEventListener')"?
> The element wasn't found. Check the selector spelling, confirm the element exists in the Elements panel, and verify the script runs after the DOM is parsed (`defer` or `DOMContentLoaded`).

> [!question]- What are the three phases of event flow?
> Capture (window → target's ancestors), target, and bubble (back up to window). Listeners run in the bubble phase by default; pass `capture: true` to run in the capture phase.

---

## 8. Predict the output

```html
<ul id="list">
  <li class="i">1</li>
  <li class="i">2</li>
</ul>
<div id="parent"><button id="child"><span id="inner">Go</span></button></div>
<a id="link" href="/home">Home</a>
```

```js
// 1
console.log(document.querySelector(".none"));

// 2
const live = document.getElementsByClassName("i");
const snap = document.querySelectorAll(".i");
const li = document.createElement("li"); li.className = "i";
document.getElementById("list").append(li);
console.log(live.length, snap.length);

// 3
const box = document.createElement("div");
console.log(box.classList.toggle("a"), box.classList.toggle("a"));

// 4
console.log(box.classList.toggle("b", false), box.className === "");

// 5  (click the button's text "Go")
parent.addEventListener("click", () => console.log("parent"));
child.addEventListener("click", () => console.log("child"));

// 6  (same setup, parent uses capture)
parent.addEventListener("click", () => console.log("parent"), { capture: true });
child.addEventListener("click", () => console.log("child"));

// 7  (click on the span inside the button; listener is on the button)
child.addEventListener("click", (e) => {
  console.log(e.target.tagName, e.currentTarget.tagName);
});

// 8
const btn = document.createElement("button");
btn.addEventListener("click", () => console.log("A"));
btn.removeEventListener("click", () => console.log("A"));
btn.click();

// 9
function hi() { console.log("hi"); }
btn.addEventListener("click", hi);
btn.addEventListener("click", hi);
btn.click();   // (btn already has the "A" listener from 8)

// 10
btn.onclick = () => console.log("one");
btn.onclick = () => console.log("two");
btn.click();   // ignore the earlier listeners; only the onclick result

// 11
const input = document.createElement("input"); input.value = "5";
console.log(typeof input.value, input.value + 1);

// 12
console.log(typeof document.querySelectorAll("li").map);

// 13
const a = document.getElementById("link");
console.log(a.getAttribute("href"), a.href === location.origin + "/home");
```

<details>
<summary>Answers</summary>

1. `null`
2. `3 2` (live collection grew, `NodeList` is a snapshot)
3. `true false`
4. `false true` (`toggle("b", false)` ensures it's removed and returns `false`)
5. Click the span: `child` then `parent` (bubbling)
6. `parent` then `child` (the capture listener on the parent runs first)
7. `SPAN BUTTON`
8. `A` still logs, because the `remove` call used a different function object
9. `A` and then `hi` **once** (the duplicate `hi` listener is ignored)
10. `two` only (the second `onclick` assignment overwrites the first; the other `addEventListener` listeners still run too, so you'd also see `A` and `hi`)
11. `"string"` and `"51"`
12. `"undefined"` (`NodeList` has no `map`)
13. `"/home"` and `true` on a page served from the site's origin (attribute is raw, the property is the absolute URL)

</details>

---

## 9. Coding practice

**1. Counter button**
```js
let count = 0;
const out = document.querySelector("#count");
document.querySelector("#inc").addEventListener("click", () => {
  count++;
  out.textContent = count;
});
```

**2. Dark mode toggle that remembers the choice**
```js
const root = document.documentElement;
if (localStorage.getItem("theme") === "dark") root.classList.add("dark");

document.querySelector("#theme-toggle").addEventListener("click", () => {
  const isDark = root.classList.toggle("dark");
  localStorage.setItem("theme", isDark ? "dark" : "light");
});
```

**3. Live character counter**
```js
const ta = document.querySelector("#bio");
const counter = document.querySelector("#counter");
ta.addEventListener("input", () => {
  counter.textContent = `${ta.value.length}/200`;
});
```

**4. Todo list: add and remove (event delegation)**
```js
form.addEventListener("submit", (e) => {
  e.preventDefault();
  const text = input.value.trim();
  if (!text) return;
  const li = document.createElement("li");
  li.textContent = text;
  list.append(li);
  input.value = "";
  input.focus();
});

list.addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (li && list.contains(li)) li.remove();
});
```

**5. Form submit with validation and `FormData`**
```js
form.addEventListener("submit", (e) => {
  e.preventDefault();
  const data = Object.fromEntries(new FormData(form));
  const errors = {};
  if (!/^\S+@\S+\.\S+$/.test(data.email ?? "")) errors.email = "Enter a valid email";
  if ((data.password ?? "").length < 8) errors.password = "Min 8 characters";

  form.querySelectorAll(".error").forEach(el => el.remove());
  for (const [name, msg] of Object.entries(errors)) {
    const p = document.createElement("p");
    p.className = "error";
    p.textContent = msg;
    form.elements[name].after(p);
  }
  if (Object.keys(errors).length === 0) console.log("submit", data);
});
```

**6. Toggle show/hide password**
```js
toggleBtn.addEventListener("click", () => {
  const show = pwd.type === "password";
  pwd.type = show ? "text" : "password";
  toggleBtn.textContent = show ? "Hide" : "Show";
});
```

**7. Close a modal with the Escape key (and clean up)**
```js
function openModal() {
  modal.hidden = false;
  const ac = new AbortController();
  document.addEventListener("keydown", (e) => {
    if (e.key === "Escape") { modal.hidden = true; ac.abort(); }
  }, { signal: ac.signal });
}
```

**8. Accordion (one open at a time)**
```js
const headers = document.querySelectorAll(".acc-header");
headers.forEach(h => h.addEventListener("click", () => {
  headers.forEach(x => x.nextElementSibling.classList.remove("open"));
  h.nextElementSibling.classList.add("open");
}));
```

**9. Copy to clipboard**
```js
btn.addEventListener("click", async () => {
  try {
    await navigator.clipboard.writeText(text);
    btn.textContent = "Copied!";
  } catch {
    btn.textContent = "Copy failed";
  }
});
```

**10. Highlight the clicked tab (single active)**
```js
tabs.addEventListener("click", (e) => {
  const tab = e.target.closest("[data-tab]");
  if (!tab) return;
  tabs.querySelectorAll(".active").forEach(t => t.classList.remove("active"));
  tab.classList.add("active");
});
```

---

## 10. Quick-recall cheat sheet

| Need | Use |
|---|---|
| One element by id | `getElementById("id")` |
| First / all by CSS selector | `querySelector` / `querySelectorAll` |
| Nearest ancestor match | `el.closest(".card")` |
| Set safe text | `el.textContent = value` |
| Set HTML (trusted only) | `el.innerHTML = html` |
| Add / remove / flip a class | `classList.add / remove / toggle` |
| Custom data on an element | `data-x="..."` → `el.dataset.x` |
| Create + insert | `createElement` → `append` |
| Remove element | `el.remove()` |
| Run on click | `addEventListener("click", fn)` |
| Run once | `{ once: true }` |
| Remove many listeners | `{ signal }` + `AbortController.abort()` |
| Which element was clicked | `e.target` (origin) / `e.currentTarget` (listener) |
| Stop reload / navigation | `e.preventDefault()` |
| Stop bubbling | `e.stopPropagation()` (use sparingly) |
| Form data as an object | `Object.fromEntries(new FormData(form))` |
| Typing in an input | `input` event (`change` = on commit) |
| Key pressed | `e.key` (`"Enter"`, `"Escape"`) |
| NodeList → array | `[...nodeList]` / `Array.from(nodeList)` |
| Debug quickly | `console.log` / `console.table` / `debugger;` |
| Find why a click fails | Console errors → element found? → listener attached? → breakpoint |

---

## Related notes
- [[JS 03 - Arrays & Objects]]
- [[JS 05 - Core Concepts (Scope, Closures, this)]]
- [[JS 08 - Async JavaScript]]
- [[JS 11 - Browser & DOM Advanced]]
