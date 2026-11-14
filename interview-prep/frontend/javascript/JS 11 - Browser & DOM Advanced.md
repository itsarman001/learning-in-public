# JS 11 — Browser & DOM Advanced

> [!info] Scope
> Event bubbling / capturing · Event delegation · `preventDefault` vs `stopPropagation` · Debounce & throttle · `localStorage` / `sessionStorage` / cookies · Observers (`IntersectionObserver`, `ResizeObserver`, `MutationObserver`)
> Previous: [[JS 10 - Error Handling]] · Next: [[JS 12 - Tooling Basics]]
> Builds on [[JS 04 - DOM & Events Basics]]. **Debounce, throttle and event delegation are asked in almost every frontend interview**, often as a live coding task.

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| Event flow | **Capture** (window → target) → **target** → **bubble** (target → window). Listeners run in the bubble phase by default. |
| Bubbling | Most events travel up the tree. Some don't: `focus`, `blur`, `mouseenter`, `mouseleave`, `load`, element `scroll`. |
| Delegation | One listener on a parent handles events from many (even future) children. Use `e.target.closest(selector)`. |
| `target` vs `currentTarget` | `target` = what was actually clicked. `currentTarget` = where the listener is attached. |
| `preventDefault()` | Cancels the browser's **default action** (form submit, link navigation). Doesn't stop bubbling. |
| `stopPropagation()` | Stops the event from reaching **other elements**. Doesn't cancel the default action. |
| `stopImmediatePropagation()` | Also stops the remaining listeners on the **same** element. |
| Debounce | Wait until calls **stop** for `n` ms, then run once (search box, autosave, resize end). |
| Throttle | Run at most **once every `n` ms** while calls keep coming (scroll, mousemove, resize). |
| `localStorage` | ~5 MB per origin, persists until cleared, strings only, **synchronous**, never sent to the server. |
| `sessionStorage` | Same API, but **per tab** and cleared when the tab closes. |
| Cookies | ~4 KB each, **sent with every request** to the domain, can be `HttpOnly`/`Secure`/`SameSite`. |
| Auth tokens | Prefer `HttpOnly; Secure; SameSite` cookies. Web storage is readable by any XSS. |
| `IntersectionObserver` | Tells you when an element enters/leaves the viewport (lazy loading, infinite scroll). |
| `ResizeObserver` | Tells you when an element's **size** changes. |
| `MutationObserver` | Tells you when the **DOM changes** (children, attributes, text). Runs as a microtask. |

---

## 1. Event propagation: bubbling and capturing

### Definition
When an event fires on an element, it doesn't just run that element's listeners. It travels through the DOM tree in **three phases**:

| Phase | `event.eventPhase` | Direction |
|---|---|---|
| **Capturing** | `1` | `window` → `document` → `html` → ... → parent of target |
| **Target** | `2` | The element that triggered the event |
| **Bubbling** | `3` | Target → parent → ... → `document` → `window` |

By default `addEventListener` registers for the **bubbling** phase. Pass `{ capture: true }` (or `true` as the third argument) to listen during capturing.

### Summary

```html
<div id="outer">
  <div id="inner">
    <button id="btn">Click</button>
  </div>
</div>
```

```js
const log = (name) => (e) => console.log(name, e.eventPhase);

outer.addEventListener("click", log("outer-capture"), { capture: true });
outer.addEventListener("click", log("outer-bubble"));
inner.addEventListener("click", log("inner-bubble"));
btn.addEventListener("click", log("btn"));

// Clicking the button prints:
// outer-capture 1   (capturing, top-down)
// btn 2             (target)
// inner-bubble 3    (bubbling, bottom-up)
// outer-bubble 3
```

**`target`, `currentTarget`, `composedPath()`**
```js
outer.addEventListener("click", (e) => {
  e.target;           // the deepest element clicked (e.g. the <button>)
  e.currentTarget;    // outer (where THIS listener lives; same as `this` in a regular function)
  e.composedPath();   // [button, inner, outer, body, html, document, window]
});
```

**Which events bubble?**

| Bubbles (default) | Does **not** bubble |
|---|---|
| `click`, `dblclick`, `mousedown`, `mouseup`, `mouseover`, `mouseout`, `mousemove` | `focus`, `blur` (use `focusin` / `focusout`, which do bubble) |
| `input`, `change`, `submit`, `reset` | `mouseenter`, `mouseleave` (use `mouseover` / `mouseout`) |
| `keydown`, `keyup` | `load`, `unload`, `error` (on elements like `<img>`) |
| `pointerdown`, `pointerup`, `pointermove` | `scroll` on elements (does bubble from `document` to `window`) |
| `focusin`, `focusout`, `contextmenu`, `wheel` | Media events (`play`, `pause`, `ended`, ...) |

Check at runtime with `event.bubbles`.

**Custom events**
```js
child.dispatchEvent(new CustomEvent("item:select", {
  detail: { id: 7 },        // payload
  bubbles: true,            // default is false!
  cancelable: true,         // allows preventDefault()
}));
parent.addEventListener("item:select", (e) => console.log(e.detail.id));   // 7
```

### Edge cases
- **Default is bubbling, not capturing.** Capturing is mainly for intercepting events before they reach the target (global shortcuts, analytics, "disable everything while loading").
- **Listener order:** on the same phase and element, listeners run in **registration order**. At the target itself, modern browsers run capture listeners before bubble listeners; older ones used registration order. Don't depend on it.
- **`target` can be a child of what you expected** (an `<svg>`/`<span>` inside a button). Use `e.target.closest("button")` or `e.currentTarget`.
- **`currentTarget` is `null` after the handler finishes** (e.g. read after `await`). Save it to a variable first.
- **`this` vs arrow functions:** `this` equals `currentTarget` only in regular functions.
- **Custom events don't bubble by default:** pass `bubbles: true`, otherwise ancestors never see them.
- **Non-bubbling events** still go through the capture phase, so a capturing listener on a parent catches `focus`/`blur`.
- **Same function + same type + same capture flag added twice** is registered once. To remove a listener, the function reference **and** the `capture` flag must match.
- **Shadow DOM:** events that are `composed` cross the shadow boundary, and `target` is *retargeted* to the host from the outside. `composedPath()` shows the full path (for open shadow roots).
- **`once: true`** removes the listener after the first run. **`passive: true`** promises not to call `preventDefault()` (enables smooth scrolling).
- **Listeners on `window`/`document` leak** if you add them in a component and never remove them. Use `removeEventListener` or `{ signal }` with `AbortController`.
- **Synthetic `element.click()`** dispatches a real `click` that bubbles.
- **Capturing can't be "stopped" by the target.** Only a listener earlier in the path calling `stopPropagation()` prevents later phases.

### Use cases
- Handling clicks on many children with one handler (delegation).
- Global shortcuts and "click outside to close" (listener on `document`).
- Intercepting events early with capturing (analytics, blocking input while a modal is open).
- Component communication with bubbling custom events.

---

## 2. Event delegation

### Definition
**Event delegation** means attaching **one** listener to a common ancestor instead of one listener per child. Because events bubble, the ancestor sees every child's event and uses `event.target` to figure out which child triggered it.

### Summary

```js
// Without delegation: one listener per item, and new items need their own
list.querySelectorAll("li").forEach(li => li.addEventListener("click", handle));

// With delegation: one listener, works for current AND future items
list.addEventListener("click", (e) => {
  const item = e.target.closest("li");           // find the item, even if a child was clicked
  if (!item || !list.contains(item)) return;     // ignore clicks outside any item
  console.log("clicked", item.dataset.id);
});
```

**Action-based delegation with `data-action`**
```html
<ul id="todos">
  <li data-id="1">Buy milk <button data-action="done">Done</button> <button data-action="delete">✕</button></li>
</ul>
```
```js
todos.addEventListener("click", (e) => {
  const btn = e.target.closest("[data-action]");
  if (!btn || !todos.contains(btn)) return;
  const li = btn.closest("li");
  switch (btn.dataset.action) {
    case "done":   li.classList.toggle("done"); break;
    case "delete": li.remove(); break;
  }
});
```

**Delegation vs direct listeners**

| | Delegation | Direct listeners |
|---|---|---|
| Number of listeners | 1 | One per element |
| Dynamically added elements | **Work automatically** | Must attach after each insert |
| Memory / setup cost | Low | Grows with the list |
| Cleanup | Remove one listener | Remove each |
| Needs event to bubble | **Yes** | No |
| Handler logic | Must filter `e.target` | Simple, target is known |
| Broken by `stopPropagation` in children | **Yes** | No |

**`target` filtering techniques**

| Technique | Notes |
|---|---|
| `e.target.closest(".item")` | **Best default.** Works when the click lands on a nested child (icon, span). |
| `e.target.matches(".btn")` | Only matches if the clicked element itself has the class; fails for children. |
| `e.target.dataset.action` | Fast, but breaks if a child element is the target. |
| `container.contains(match)` | Guards against matching an ancestor *outside* your container. |

### Edge cases
- **`matches` vs `closest`:** clicking a `<span>` inside `<button class="btn">` makes `e.target.matches(".btn")` `false`, while `e.target.closest(".btn")` finds the button.
- **Always check `container.contains(match)`:** `closest` walks up past your container, so it could match an unrelated outer element with the same selector.
- **Events that don't bubble can't be delegated:** use `focusin`/`focusout` for focus and `mouseover`/`mouseout` (with `relatedTarget`) for hover.
- **`stopPropagation()` in a child breaks delegation.** The parent never hears the event.
- **`event.currentTarget` is the delegate (container),** not the clicked item. Use the matched element for item-specific logic.
- **Heavy handlers:** the delegated handler runs for *every* matching event in the subtree (every mousemove, say). Filter early and avoid expensive work.
- **Delegating on `document`/`body` for everything** works but becomes hard to follow and can slow down hot events. Delegate on the nearest sensible container.
- **Elements removed during the event** (`li.remove()`) are still the `target` for the rest of dispatch, but `closest` on a detached node won't reach the container. Resolve what you need before removing.
- **`pointer-events: none`** on children makes the parent the target (sometimes used deliberately).
- **Inputs inside the container:** `change`/`input` bubble, so you can delegate form handling, but `focus`/`blur` need `focusin`/`focusout`.
- **React:** you rarely write delegation by hand. React attaches handlers at the root and does its own delegation (synthetic events).
- **Right-clicks and keyboard:** `click` also fires for Enter/Space on buttons. Use `contextmenu` or `e.button` for right click.

### Use cases
- Lists, tables, menus, tabs, accordions, and any UI that adds/removes children dynamically.
- Large lists where per-item listeners would be wasteful.
- Centralizing handlers with `data-action` attributes.

---

## 3. `preventDefault()` vs `stopPropagation()`

### Definition
- **`event.preventDefault()`** cancels the browser's **built-in behavior** for that event (if the event is cancelable).
- **`event.stopPropagation()`** stops the event from traveling to **other elements** (further capture/bubble targets). Other listeners on the **same** element still run.
- **`event.stopImmediatePropagation()`** does the same, **and** prevents the remaining listeners on the **same** element.

They are completely **independent**: using one doesn't imply the other.

### Summary

| | `preventDefault()` | `stopPropagation()` | `stopImmediatePropagation()` |
|---|---|---|---|
| Cancels default action | **Yes** | No | No |
| Stops other elements' listeners | No | **Yes** | **Yes** |
| Stops other listeners on the same element | No | No | **Yes** |
| Event keeps bubbling | Yes | No | No |
| Checked with | `event.defaultPrevented` | n/a | n/a |

**Typical default actions you cancel**

| Event | Default action | Typical reason to prevent it |
|---|---|---|
| `submit` (form) | Reloads/navigates | Send data with `fetch` instead |
| `click` on `<a>` | Navigates | SPA routing or custom handling |
| `keydown` | Types a char / shortcut / scroll | Custom shortcuts, block characters |
| `contextmenu` | Browser context menu | Custom menu |
| `dragover` / `drop` | Rejects the drop / opens the file | Enable drag-and-drop, handle files |
| `wheel`, `touchmove` | Scrolls | Custom scrolling (needs a non-passive listener) |
| `beforeunload` | Closes the tab silently | Show a "leave site?" prompt |
| `mousedown` | Starts text selection / focus | Prevent selection while dragging |

```js
form.addEventListener("submit", (e) => {
  e.preventDefault();                       // no page reload
  sendWithFetch(new FormData(form));
});

link.addEventListener("click", (e) => {
  if (e.metaKey || e.ctrlKey || e.shiftKey || e.button !== 0) return;   // let "open in new tab" work
  e.preventDefault();
  router.navigate(link.getAttribute("href"));
});

dropZone.addEventListener("dragover", (e) => e.preventDefault());       // required to allow dropping
dropZone.addEventListener("drop", (e) => { e.preventDefault(); handleFiles(e.dataTransfer.files); });

window.addEventListener("beforeunload", (e) => {
  if (hasUnsavedChanges) e.preventDefault();   // triggers the browser's confirmation dialog
});
```

**Using them together**
```js
child.addEventListener("click", (e) => { e.stopPropagation(); });        // parent won't hear it
a.addEventListener("click", (e) => { e.preventDefault(); });             // no navigation, but still bubbles
```

**Better than `stopPropagation` for "click outside"**
```js
function onOutsideClick(el, handler) {
  const listener = (e) => { if (!e.composedPath().includes(el)) handler(e); };
  document.addEventListener("pointerdown", listener);
  return () => document.removeEventListener("pointerdown", listener);
}
```

### Edge cases
- **Only cancelable events can be prevented** (`event.cancelable`). Calling `preventDefault()` on a non-cancelable event does nothing.
- **`passive: true` listeners can't prevent default** (the call is ignored and logs a warning). Browsers make `touchstart`, `touchmove` and `wheel` listeners on `window`/`document`/`body` passive by default. Use `{ passive: false }` when you must cancel scrolling.
- **`return false` only works for inline `onclick="return false"` and `el.onclick = () => false`.** In `addEventListener` it does nothing. (In jQuery handlers it means preventDefault **and** stopPropagation.)
- **`stopPropagation` hurts other code:** analytics, global "close dropdown on outside click", and event delegation all rely on events bubbling. Prefer checking `event.target` or `composedPath()`.
- **`stopPropagation` doesn't stop other listeners on the same element;** use `stopImmediatePropagation` for that.
- **Checkbox/radio quirk:** inside a `click` handler that calls `preventDefault()`, the box *appears* toggled, then reverts after the handler finishes.
- **`preventDefault` on `keydown`** stops the character from appearing; on `keypress`/`keyup` it's too late (and `keypress` is deprecated).
- **Capture-phase `stopPropagation` on `window`** prevents the event from reaching *anything* else. A very blunt tool.
- **`defaultPrevented`** lets other listeners check whether someone already handled the event.
- **Forms:** preventing `submit` doesn't stop native validation. Native validation runs *before* `submit`; add `novalidate` or use `checkValidity()`.
- **`beforeunload`:** modern browsers ignore custom text and only show a generic prompt. Add the listener only while there are unsaved changes.
- **Programmatic events** (`dispatchEvent`) support `preventDefault` only if created with `cancelable: true`; the return value of `dispatchEvent` is `false` if prevented.

### Use cases
- `preventDefault`: AJAX forms, SPA links, drag and drop, custom shortcuts, custom context menus.
- `stopPropagation`: isolating a widget so a parent click handler doesn't fire (use sparingly).
- `stopImmediatePropagation`: ensuring only the first of several same-element handlers runs.

---

## 4. Debounce and throttle

### Definition
Both **limit how often a function runs** when an event fires rapidly.

- **Debounce:** delays execution until there has been **no new call for `wait` ms**. Every new call resets the timer. Result: runs **once, after the burst ends**.
- **Throttle:** guarantees the function runs **at most once per `interval` ms** while calls keep coming. Result: runs **regularly during the burst**.

### Summary

| | Debounce | Throttle |
|---|---|---|
| Runs | After calls **stop** for `wait` ms | At most once per `interval` ms |
| During continuous events | Never runs (until a pause) | Runs periodically |
| Analogy | Elevator door waits for people to stop entering | Machine gun with a fixed fire rate |
| Typical events | `input` (search), `resize` end, autosave, validation | `scroll`, `mousemove`, `resize` (live), drag, infinite scroll |
| Variants | Leading / trailing, `maxWait`, `cancel`, `flush` | Leading / trailing, `cancel` |
| Timeline (calls at 0, 10, 20 ms; `n` = 100) | One run at ~120 ms | Runs at 0 ms (and a trailing run at 100 ms) |

**Debounce**
```js
function debounce(fn, wait = 300) {
  let timer;
  function debounced(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = undefined;
      fn.apply(this, args);               // keep `this` and arguments
    }, wait);
  }
  debounced.cancel = () => { clearTimeout(timer); timer = undefined; };
  return debounced;
}

input.addEventListener("input", debounce((e) => search(e.target.value), 300));
```

**Debounce with `leading` / `trailing` options**
```js
function debounce(fn, wait = 300, { leading = false, trailing = true } = {}) {
  let timer, lastArgs, lastThis;
  function debounced(...args) {
    lastArgs = args; lastThis = this;
    const callNow = leading && !timer;                   // first call of a burst
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = undefined;
      if (trailing && lastArgs) fn.apply(lastThis, lastArgs);
      lastArgs = lastThis = undefined;
    }, wait);
    if (callNow) { fn.apply(this, args); lastArgs = lastThis = undefined; }
  }
  debounced.cancel = () => { clearTimeout(timer); timer = lastArgs = lastThis = undefined; };
  return debounced;
}
```

**Throttle (leading + trailing)**
```js
function throttle(fn, interval = 200) {
  let last = 0, timer, lastArgs, lastThis;
  function throttled(...args) {
    const now = Date.now();
    const remaining = interval - (now - last);
    lastArgs = args; lastThis = this;
    if (remaining <= 0) {                                // enough time has passed: run now
      clearTimeout(timer); timer = undefined;
      last = now;
      fn.apply(this, args);
    } else if (!timer) {                                 // otherwise schedule one trailing run
      timer = setTimeout(() => {
        last = Date.now(); timer = undefined;
        fn.apply(lastThis, lastArgs);
      }, remaining);
    }
  }
  throttled.cancel = () => { clearTimeout(timer); timer = undefined; last = 0; };
  return throttled;
}

window.addEventListener("scroll", throttle(updateProgressBar, 100), { passive: true });
```

**Simplest throttle (leading only)**
```js
const throttle = (fn, ms) => {
  let waiting = false;
  return (...args) => {
    if (waiting) return;
    fn(...args);
    waiting = true;
    setTimeout(() => { waiting = false; }, ms);
  };
};
```

**Animation-frame throttle (for visual updates)**
```js
function rafThrottle(fn) {
  let ticking = false;
  return (...args) => {
    if (ticking) return;
    ticking = true;
    requestAnimationFrame(() => { ticking = false; fn(...args); });
  };
}
window.addEventListener("mousemove", rafThrottle(drawCursor));
```

**In React**
```jsx
// Debounce a VALUE (most common)
function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = React.useState(value);
  React.useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);            // restart on every change, clean up on unmount
  }, [value, delay]);
  return debounced;
}

const debouncedQuery = useDebouncedValue(query, 300);
React.useEffect(() => { fetchResults(debouncedQuery); }, [debouncedQuery]);
```

**Which one to use?**

| Situation | Choice |
|---|---|
| Search-as-you-type, auto-save, form validation on typing | **Debounce** |
| "Run after the user finishes resizing" | **Debounce** |
| Scroll position tracking, parallax, progress bars | **Throttle** (or `rAF`) |
| Mouse-move / drag handlers | **Throttle** (or `rAF`) |
| Prevent double-click submits | Leading **debounce** or disable the button |
| Infinite scroll check | **Throttle** (or `IntersectionObserver`, better) |
| Smooth visual updates | `requestAnimationFrame` |

### Edge cases
- **Forward `this` and arguments:** use `fn.apply(this, args)` inside a regular (not arrow) wrapper function, or event handlers and methods will break.
- **Create the debounced function once.** Calling `debounce(...)` inside a React render or inside the event handler makes a new timer each time, so nothing is ever debounced. Use `useMemo`/`useRef`/`useCallback`, or debounce a **value** with the hook above.
- **Clean up on unmount:** call `.cancel()` (or `clearTimeout` in the effect cleanup) so a pending call doesn't run after the component is gone.
- **The wrapper returns `undefined`,** not the result of `fn`. To get results, return a promise or use a callback.
- **Stale responses:** debouncing reduces requests but doesn't prevent out-of-order responses. Combine with `AbortController` ([[JS 08 - Async JavaScript]]).
- **Trailing vs leading:** pure trailing debounce feels laggy for the first keystroke; `leading: true` runs immediately then ignores the burst. Throttle with only leading drops the **last** event (the final scroll position may never be processed). Use trailing too.
- **Wait of `0`:** still defers to the next macrotask.
- **Don't pick a `wait` that's too long:** 200 to 500 ms for search; 100 to 200 ms for scroll/resize; ~16 ms equals one animation frame.
- **Timer accuracy:** `setTimeout` is a minimum delay, and timers in background tabs are throttled.
- **`flush`:** sometimes you must run the pending call immediately (before navigation or on blur). Libraries like Lodash provide `.flush()`.
- **Lodash:** `_.debounce(fn, wait, { leading, trailing, maxWait })` and `_.throttle(fn, wait, { leading, trailing })` (throttle is debounce with `maxWait = wait`).
- **Testing:** use fake timers (`jest.useFakeTimers()`) and advance time.
- **Scroll/touch listeners:** pair throttled scroll handlers with `{ passive: true }`. For visibility checks prefer `IntersectionObserver`.
- **Debounce vs disabling a button:** for submits, disabling the button after the first click (and re-enabling in `finally`) is often more reliable.

### Use cases
- **Debounce:** search suggestions, autosave drafts, validating usernames, window resize recalculations, analytics "stopped typing".
- **Throttle:** scroll-based UI (sticky headers, progress), drag/resize handlers, mousemove tracking, rate-limiting button spam.

---

## 5. `localStorage`, `sessionStorage`, and cookies

### Definition
Three ways to keep data in the browser:
- **Web Storage** (`localStorage`, `sessionStorage`): simple key-value string stores, synchronous, accessible from JavaScript only, never sent automatically to the server.
- **Cookies**: small key-value strings that the browser **sends to the server with every matching request**, and that can be created by the server (`Set-Cookie`) or by JavaScript (`document.cookie`).

### Summary

| | `localStorage` | `sessionStorage` | Cookies |
|---|---|---|---|
| Capacity | ~5 MB per origin | ~5 MB per origin | ~4 KB per cookie (about 50+ per domain) |
| Lifetime | Until cleared (by code or user) | Until the **tab/window** closes | Session, or until `Expires`/`Max-Age` |
| Scope | **Origin** (scheme + host + port), shared across tabs | Origin **and tab** (not shared between tabs) | Domain + path (not port-specific); can include subdomains |
| Sent to server | **Never** | **Never** | **Automatically with every request** to the domain/path |
| Readable by JS | Yes | Yes | Yes, unless `HttpOnly` |
| Set by | JS only | JS only | Server (`Set-Cookie`) or JS (`document.cookie`) |
| API | Synchronous `getItem`/`setItem` | Same | String parsing of `document.cookie` |
| Data type | Strings only | Strings only | Strings only |
| Security flags | None | None | `HttpOnly`, `Secure`, `SameSite`, `Path`, `Domain` |
| Cross-tab events | `storage` event in **other** tabs | n/a | none |
| Typical use | Theme, preferences, drafts, cached non-sensitive data | Per-tab wizard step, temporary form state | Sessions/auth, CSRF tokens, consent, server-visible prefs |

**Web Storage API**
```js
localStorage.setItem("theme", "dark");
localStorage.getItem("theme");        // "dark"  (returns null if missing)
localStorage.removeItem("theme");
localStorage.clear();                 // removes everything for this origin
localStorage.length;                  // number of keys
localStorage.key(0);                  // name of the first key

// Only strings are stored: serialize objects
localStorage.setItem("user", JSON.stringify({ id: 1, name: "Asha" }));
const user = JSON.parse(localStorage.getItem("user") ?? "null");

// sessionStorage: identical API
sessionStorage.setItem("step", "2");
```

**Cookies from JavaScript**
```js
document.cookie = "theme=dark; max-age=31536000; path=/; SameSite=Lax; Secure";
document.cookie;                       // "theme=dark; lang=en"  (only name=value pairs; no attributes)
document.cookie = "theme=; max-age=0; path=/";   // delete (same path/domain as when set)
```

**Cookie attributes**

| Attribute | Meaning |
|---|---|
| `Max-Age` / `Expires` | Lifetime in seconds / absolute date. Without either it's a **session cookie**. (Browsers cap lifetime, e.g. ~400 days in Chrome.) |
| `Path` | URL path prefix the cookie is sent to (default: the current page's directory) |
| `Domain` | Cookie also sent to subdomains (`Domain=example.com`). Omit it to restrict to the exact host |
| `Secure` | Only sent over HTTPS |
| `HttpOnly` | **Not accessible from JavaScript** (set by the server). Protects against XSS stealing it |
| `SameSite=Strict` | Sent only for same-site requests |
| `SameSite=Lax` | Also sent on top-level navigations (clicking a link). Default in modern browsers |
| `SameSite=None` | Sent on cross-site requests (requires `Secure`); needed for embedded/third-party use |

**Cookie helpers**
```js
const cookies = {
  set(name, value, { days = 7, path = "/", sameSite = "Lax", secure = location.protocol === "https:" } = {}) {
    let c = `${encodeURIComponent(name)}=${encodeURIComponent(value)}; max-age=${days * 86400}; path=${path}; SameSite=${sameSite}`;
    if (secure) c += "; Secure";
    document.cookie = c;
  },
  get(name) {
    const prefix = encodeURIComponent(name) + "=";
    for (const part of document.cookie.split("; ")) {
      if (part.startsWith(prefix)) return decodeURIComponent(part.slice(prefix.length));
    }
    return null;
  },
  remove(name, path = "/") {
    document.cookie = `${encodeURIComponent(name)}=; max-age=0; path=${path}`;
  },
};
```

**Safe storage wrapper (JSON, errors, optional expiry)**
```js
const store = {
  set(key, value, ttlMs) {
    const record = { value, expires: ttlMs ? Date.now() + ttlMs : null };
    try { localStorage.setItem(key, JSON.stringify(record)); return true; }
    catch { return false; }                      // QuotaExceededError, SecurityError, circular JSON
  },
  get(key, fallback = null) {
    try {
      const raw = localStorage.getItem(key);
      if (raw === null) return fallback;
      const { value, expires } = JSON.parse(raw);
      if (expires && Date.now() > expires) { localStorage.removeItem(key); return fallback; }
      return value;
    } catch { return fallback; }
  },
  remove(key) { try { localStorage.removeItem(key); } catch { /* ignore */ } },
};
```

**Syncing tabs with the `storage` event**
```js
window.addEventListener("storage", (e) => {      // fires in OTHER tabs of the same origin
  if (e.storageArea !== localStorage) return;
  if (e.key === "theme") applyTheme(e.newValue);          // e.key is null after clear()
  if (e.key === "logout") location.href = "/login";
});
function logoutEverywhere() { localStorage.setItem("logout", String(Date.now())); }
```

**Choosing a storage mechanism**

| Need | Use |
|---|---|
| UI preferences (theme, language), non-sensitive cache | `localStorage` |
| Temporary per-tab state (wizard step, unsaved form text) | `sessionStorage` |
| Session/auth identifier the **server** needs | **Cookie** (`HttpOnly; Secure; SameSite`) |
| Large or structured data, offline apps, binary data | **IndexedDB** |
| Cached network responses / PWA assets | **Cache API** + service workers |
| Messaging between tabs | `storage` event or `BroadcastChannel` |

### Edge cases
- **Everything is a string:** `localStorage.setItem("n", 5)` stores `"5"`; `setItem("o", {a:1})` stores `"[object Object]"`; `setItem("b", false)` stores `"false"`, which is **truthy**. Serialize with `JSON.stringify` and parse with `JSON.parse`.
- **`getItem` returns `null` when missing:** `JSON.parse(null)` returns `null`, but `JSON.parse(undefined)` throws. Guard with `?? "null"`.
- **Can throw:** `setItem` throws `QuotaExceededError` when full; accessing `localStorage` at all can throw `SecurityError` (blocked cookies/storage, sandboxed iframes, some privacy modes). Wrap in `try/catch`.
- **Synchronous and blocking:** large reads/writes block the main thread. Keep values small; use IndexedDB for big data.
- **`storage` event doesn't fire in the tab that made the change,** only in other tabs/windows on the same origin.
- **Origin isolation:** `http://a.com`, `https://a.com`, and `https://sub.a.com` are **different origins** with separate `localStorage`. Cookies can be shared across subdomains with `Domain`.
- **`sessionStorage` is per tab:** opening the same site in a new tab starts empty (a duplicated tab or `window.open` may copy it). It survives reloads but not closing the tab.
- **Not available in Web Workers/service workers** (use IndexedDB or the Cache API there).
- **Security:** any XSS can read all Web Storage. **Don't store auth tokens, secrets, or sensitive personal data there.** Use `HttpOnly; Secure; SameSite` cookies for sessions; add CSRF protection (SameSite, CSRF tokens) because cookies are sent automatically.
- **`HttpOnly` cookies are invisible to `document.cookie`** (and can't be set from JS).
- **Cookies are sent on every request** to the matching domain/path (images, scripts, API calls), costing bandwidth. Keep them small.
- **Deleting a cookie** requires the same `path` (and `domain`) it was created with, otherwise you create a different cookie.
- **`document.cookie` setting is additive:** assigning writes **one** cookie, it doesn't overwrite the whole string. Reading returns all visible cookies as `a=1; b=2`.
- **Always encode cookie values** (`encodeURIComponent`) to avoid `;`, `,` and spaces breaking them.
- **`SameSite=None` requires `Secure`;** third-party cookies are increasingly blocked/restricted, so don't rely on them.
- **Cookie name prefixes** `__Host-` and `__Secure-` enforce `Secure` (and `Path=/`, no `Domain` for `__Host-`).
- **Browser privacy features** (e.g. Safari's tracking prevention) may cap the lifetime of script-written cookies/storage. Treat client storage as **ephemeral**; the user can clear it anytime.
- **No transactions or locking:** two tabs writing the same key race; last write wins.
- **Persistence is best-effort:** storage can be cleared under disk pressure or in private/incognito mode.
- **GDPR/consent:** non-essential cookies/storage (analytics, tracking) generally need user consent.
- **Key naming:** namespace keys (`app:v1:theme`) to avoid collisions and allow versioned migrations.

### Use cases
- `localStorage`: theme, language, collapsed sidebar, last-used filters, cached non-sensitive API data, drafts.
- `sessionStorage`: multi-step form progress, temporary UI state, one-time banners per session.
- Cookies: login sessions (HttpOnly), CSRF tokens, consent choices, server-side personalization, A/B bucket ids.

---

## 6. Observers

### Definition
**Observer APIs** let the browser notify your code **asynchronously** when something changes, instead of you polling or listening to noisy events like `scroll`.

| Observer | Watches | Replaces |
|---|---|---|
| **`IntersectionObserver`** | Whether an element is visible in the viewport/ancestor (and how much) | `scroll` + `getBoundingClientRect()` |
| **`ResizeObserver`** | Size changes of an element | `window.resize` + manual measuring |
| **`MutationObserver`** | DOM changes: children, attributes, text | Deprecated "mutation events", polling |
| `PerformanceObserver` | Performance entries (LCP, CLS, long tasks, resources) | Manual timing (see [[JS 15 - Performance]]) |

### 6.1 `IntersectionObserver`

```js
const io = new IntersectionObserver((entries, observer) => {
  for (const entry of entries) {
    entry.isIntersecting;        // is it (partly) visible?
    entry.intersectionRatio;     // 0..1 fraction visible
    entry.target;                // the observed element
    entry.boundingClientRect;    // its rect
  }
}, {
  root: null,                    // null = the browser viewport (or pass a scrollable ancestor)
  rootMargin: "200px 0px",       // grow/shrink the root box (like CSS margin): trigger early
  threshold: [0, 0.5, 1],        // ratios at which the callback fires (default 0)
});

io.observe(element);
io.unobserve(element);           // stop watching one element
io.disconnect();                 // stop watching everything
```

**Lazy-load images**
```js
const io = new IntersectionObserver((entries, obs) => {
  for (const entry of entries) {
    if (!entry.isIntersecting) continue;
    const img = entry.target;
    img.src = img.dataset.src;
    obs.unobserve(img);          // one-time work
  }
}, { rootMargin: "200px" });
document.querySelectorAll("img[data-src]").forEach(img => io.observe(img));
// Modern alternative for plain images: <img loading="lazy">
```

**Infinite scroll with a sentinel**
```js
let loading = false;
const io = new IntersectionObserver(async ([entry]) => {
  if (!entry.isIntersecting || loading) return;
  loading = true;
  try { await loadMore(); }
  finally {
    loading = false;
    io.unobserve(sentinel); io.observe(sentinel);   // re-check if the sentinel is still visible
  }
}, { rootMargin: "300px" });
io.observe(sentinel);
```

**Scroll-spy (highlight the nav link of the visible section)**
```js
const links = new Map([...document.querySelectorAll("nav a")].map(a => [a.hash.slice(1), a]));
const io = new IntersectionObserver((entries) => {
  for (const entry of entries) links.get(entry.target.id)?.classList.toggle("active", entry.isIntersecting);
}, { rootMargin: "-40% 0px -55% 0px" });            // only the band around the middle counts
document.querySelectorAll("section[id]").forEach(s => io.observe(s));
```

### 6.2 `ResizeObserver`

```js
const ro = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;     // content box size
    // entry.borderBoxSize[0].inlineSize / blockSize for the border box
    resizeChart(entry.target, width, height);
  }
});
ro.observe(container);          // options: { box: "border-box" | "content-box" | "device-pixel-content-box" }
// ro.unobserve(container); ro.disconnect();
```

### 6.3 `MutationObserver`

```js
const mo = new MutationObserver((records) => {
  for (const r of records) {
    r.type;                      // "childList" | "attributes" | "characterData"
    r.addedNodes; r.removedNodes;
    r.attributeName; r.oldValue; // with attributeOldValue: true
    r.target;
  }
});
mo.observe(targetNode, {
  childList: true,               // children added/removed
  attributes: true,              // attribute changes
  attributeFilter: ["class", "data-state"],   // only these attributes (optional)
  characterData: true,           // text node content changes
  subtree: true,                 // include all descendants
  attributeOldValue: true,
});
mo.disconnect();
mo.takeRecords();                // grab pending records immediately and clear the queue
```

**Wait for an element to appear**
```js
function waitForElement(selector, { root = document, timeout = 5000 } = {}) {
  return new Promise((resolve, reject) => {
    const existing = root.querySelector(selector);
    if (existing) return resolve(existing);

    const observer = new MutationObserver(() => {
      const el = root.querySelector(selector);
      if (el) { cleanup(); resolve(el); }
    });
    const timer = setTimeout(() => { cleanup(); reject(new Error(`Timed out waiting for ${selector}`)); }, timeout);
    function cleanup() { observer.disconnect(); clearTimeout(timer); }

    observer.observe(root, { childList: true, subtree: true });
  });
}
```

**Comparison**

| | `IntersectionObserver` | `ResizeObserver` | `MutationObserver` |
|---|---|---|---|
| Triggered by | Visibility changes | Element size changes | DOM tree/attribute/text changes |
| Callback timing | Async, batched (after layout) | After layout, before paint | **Microtask** after the mutation |
| Fires immediately on `observe()` | **Yes** (initial entry with current state) | Yes (if the size is non-zero) | No |
| Main options | `root`, `rootMargin`, `threshold` | `box` | `childList`, `attributes`, `subtree`, `characterData`, `attributeFilter` |
| Clean up | `unobserve` / `disconnect` | same | `disconnect` |
| Typical uses | Lazy loading, infinite scroll, animations on scroll, ad/analytics impressions | Responsive components, charts, auto-grow textareas | Reacting to third-party DOM changes, waiting for elements, extensions |

### Edge cases
- **Callbacks are asynchronous and batched:** they get an **array** of entries/records, and `forEach`/`for...of` over all of them.
- **`IntersectionObserver` fires an initial callback** right after `observe()` with the current state (visible or not). Handle `isIntersecting: false` entries.
- **It reports *changes* only:** if you load more content and the sentinel is still visible, no new callback fires. Re-observe (`unobserve` then `observe`) as shown above.
- **`threshold: 1` may never fire** for an element taller than the viewport (it can't be 100% visible). Use a lower threshold.
- **`display: none` elements never intersect** and have no size for `ResizeObserver` (they fire when shown).
- **`rootMargin` uses CSS margin syntax** (`"10px 20px"`) and requires units. Negative values shrink the root box.
- **Unobserve after one-time work** (lazy loading, "animate once") and always `disconnect()` on cleanup (React `useEffect` return), or you leak.
- **`ResizeObserver` loop warning:** if your callback changes the observed element's size, the browser may log `ResizeObserver loop completed with undelivered notifications`. Usually harmless, but avoid size-changing feedback loops (or defer with `requestAnimationFrame`).
- **`ResizeObserver` doesn't report position changes,** only size.
- **`MutationObserver` loops:** changing the observed DOM inside the callback triggers the callback again (infinite loop). Disconnect first, filter by `attributeFilter`, or guard with a flag.
- **`MutationObserver` must observe something:** passing none of `childList`, `attributes`, `characterData` throws a `TypeError`.
- **`subtree: true` on `document.body`** can be expensive on busy pages. Observe the smallest node and use `attributeFilter`.
- **`MutationObserver` runs as a microtask,** so it fires before any `setTimeout` and right after the current script (even before rendering).
- **Observers hold references** to targets, so detached/removed elements can't be garbage-collected until unobserved.
- **Not for precise scroll positions:** `IntersectionObserver` tells you *whether* something is visible, not exact pixel offsets.
- **Cross-origin iframes:** `rootMargin` and intersection details are limited for security reasons.
- **Older browsers** may need polyfills (mostly irrelevant for modern targets).
- **Don't use `scroll` + `getBoundingClientRect` for visibility.** It forces layout on every event and janks the page.

### Use cases
- **IntersectionObserver:** lazy-load images/components, infinite scroll, scroll-triggered animations, sticky-header toggles, "mark as read" when visible, video autoplay/pause, impression tracking.
- **ResizeObserver:** re-render charts/canvas on container resize, element-level responsive behavior, auto-resizing textareas, virtualized list measurements.
- **MutationObserver:** reacting to DOM changes you don't control (third-party widgets, browser extensions, CMS content), waiting for elements, accessibility tooling, undo history in editors.

---

## 7. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### Bubbling, capturing & delegation

> [!question]- What is event bubbling?
> After an event fires on the target, it propagates up through its ancestors (parent, grandparent, ... `document`, `window`), triggering their listeners for that event type.

> [!question]- What is event capturing?
> The phase before the target phase, where the event travels from `window` down to the target. Listeners registered with `{ capture: true }` run during this phase.

> [!question]- Describe the three phases of event propagation.
> Capturing (window → target's parent), target (the element itself), bubbling (target → window). Listeners are bubble-phase by default.

> [!question]- Which events don't bubble?
> `focus`, `blur`, `mouseenter`, `mouseleave`, `load`, `unload`, element-level `scroll`, and media events. Use `focusin`/`focusout` and `mouseover`/`mouseout` as bubbling alternatives.

> [!question]- What is event delegation and why use it?
> Attaching one listener to a parent and using `event.target` to identify the child. It reduces listeners/memory, handles dynamically added children automatically, and simplifies cleanup.

> [!question]- What is the difference between `event.target` and `event.currentTarget`?
> `target` is the element that originated the event (deepest). `currentTarget` is the element whose listener is currently running (the one it's attached to).

> [!question]- Why use `closest()` instead of checking `e.target` directly in delegation?
> The click may land on a child (icon, span) of the element you care about. `e.target.closest(selector)` walks up to the matching ancestor; `matches` only tests the target itself.

> [!question]- What are the downsides of event delegation?
> It requires bubbling events, breaks if a child calls `stopPropagation()`, needs `target` filtering, and the handler runs for every event in the subtree.

> [!question]- How would you delegate `focus` or `mouseenter`?
> Use their bubbling counterparts: `focusin`/`focusout` and `mouseover`/`mouseout` (with `relatedTarget`), or use capturing on the parent.

> [!question]- How do you create and dispatch a custom event so that parents can hear it?
> `el.dispatchEvent(new CustomEvent("name", { detail, bubbles: true }))`. `bubbles` defaults to `false`.

> [!question]- Does React use event delegation?
> Yes. React attaches handlers at the root container and dispatches synthetic events to components, so you rarely delegate manually.

### preventDefault vs stopPropagation

> [!question]- Difference between `preventDefault` and `stopPropagation`?
> `preventDefault` cancels the browser's default action (submit, navigation). `stopPropagation` stops the event from reaching other elements. They're independent.

> [!question]- What does `stopImmediatePropagation` add?
> It also prevents remaining listeners on the same element from running.

> [!question]- Does `preventDefault` stop bubbling?
> No. The event still bubbles to ancestors (with `defaultPrevented` set to `true`).

> [!question]- Does `return false` prevent default in `addEventListener`?
> No. It only works for inline handlers/`onclick` properties (and jQuery handlers, where it also stops propagation).

> [!question]- Why is `stopPropagation` considered harmful?
> It breaks delegation, analytics, and "click outside" logic that rely on bubbling. Prefer checking `event.target` or `composedPath()`.

> [!question]- What is a passive event listener?
> A listener that promises not to call `preventDefault()`, letting the browser scroll without waiting for the handler. Pass `{ passive: true }`; `touchstart`/`touchmove`/`wheel` on `window`/`document`/`body` are passive by default.

> [!question]- How do you stop a form from reloading the page?
> Call `event.preventDefault()` in the form's `submit` handler, then send data with `fetch`.

> [!question]- How do you build drag and drop for files?
> Call `preventDefault()` in both `dragover` (to allow dropping) and `drop` (to stop the browser from opening the file), then read `e.dataTransfer.files`.

### Debounce & throttle

> [!question]- What is debouncing?
> Delaying a function until no new calls have happened for `wait` ms. Every call resets the timer, so it runs once after the burst ends.

> [!question]- What is throttling?
> Limiting a function to run at most once per interval while events keep firing.

> [!question]- Debounce vs throttle: when do you use each?
> Debounce when only the final value matters (search input, autosave, resize end). Throttle when you need regular updates during continuous events (scroll, mousemove, drag).

> [!question]- Implement `debounce`.
> `clearTimeout(timer); timer = setTimeout(() => fn.apply(this, args), wait);` inside a returned regular function that forwards `this` and arguments. See coding practice.

> [!question]- What are leading and trailing options?
> `leading` runs on the first call of a burst; `trailing` runs after the burst ends. For debounce, trailing is the default. A leading-only throttle can drop the last event, so use both when the final value matters.

> [!question]- Why does debouncing inside a React component often not work?
> Creating the debounced function during render makes a new timer every render. Create it once (`useMemo`/`useRef`/`useCallback`) or debounce a value with `useDebouncedValue` and clean up on unmount.

> [!question]- Does debouncing solve race conditions in search?
> No. It reduces requests, but an older slower response can still arrive last. Cancel previous requests with `AbortController` or ignore stale results.

> [!question]- How is `requestAnimationFrame` related to throttling?
> `rAF` throttling runs the handler at most once per frame, which is ideal for visual updates like cursor following or scroll-driven animation.

> [!question]- What would you use instead of throttled scroll listeners for visibility checks?
> `IntersectionObserver`, because it doesn't fire on every scroll event or force layout.

### Storage & cookies

> [!question]- Difference between `localStorage`, `sessionStorage` and cookies?
> `localStorage`: ~5 MB, persists until cleared, shared across tabs of an origin, never sent to the server. `sessionStorage`: same API but per tab and cleared when the tab closes. Cookies: ~4 KB, sent to the server with each request, with expiry and security flags (`HttpOnly`, `Secure`, `SameSite`).

> [!question]- What data types can Web Storage hold?
> Only strings. Serialize objects with `JSON.stringify` and parse with `JSON.parse`; `setItem("n", 5)` stores `"5"`.

> [!question]- What does `localStorage.getItem("missing")` return?
> `null`.

> [!question]- What is the `storage` event?
> It fires in *other* tabs/windows of the same origin when `localStorage` changes (not in the tab that changed it), useful for syncing state like theme or logout across tabs.

> [!question]- Is it safe to store JWTs in `localStorage`?
> It's risky: any XSS can read it. Prefer `HttpOnly; Secure; SameSite` cookies for session/auth tokens (and add CSRF protection), or keep tokens in memory with refresh via cookie.

> [!question]- What do `HttpOnly`, `Secure` and `SameSite` do?
> `HttpOnly`: hidden from JavaScript (mitigates XSS theft). `Secure`: sent only over HTTPS. `SameSite`: controls cross-site sending (`Strict`/`Lax`/`None`), mitigating CSRF.

> [!question]- Why can `localStorage.setItem` throw?
> `QuotaExceededError` when storage is full, or `SecurityError` when storage is blocked (privacy settings, sandboxed iframes). Wrap it in `try/catch`.

> [!question]- How do you delete a cookie?
> Set it again with the same name, `path` and `domain` and `max-age=0` (or a past `expires`).

> [!question]- Why are cookies a performance concern?
> They're sent with every request to the matching domain/path, including static assets, adding bandwidth and latency.

> [!question]- Where would you store a theme preference? A login session?
> Theme: `localStorage` (maybe also a cookie if the server renders it). Session: a server-set `HttpOnly; Secure; SameSite` cookie.

> [!question]- When would you choose IndexedDB over `localStorage`?
> For large, structured, or binary data, offline support, indexed queries, and asynchronous access (non-blocking). It's also available in workers.

> [!question]- Are `localStorage` values shared between subdomains?
> No. Subdomains are different origins. Cookies can be shared with the `Domain` attribute.

### Observers

> [!question]- What is `IntersectionObserver` and what is it used for?
> An API that notifies you asynchronously when an element enters/leaves the viewport (or another root) and by how much. Used for lazy loading, infinite scroll, scroll animations, and impression tracking.

> [!question]- What do `root`, `rootMargin` and `threshold` mean?
> `root`: the scrollable ancestor (default viewport). `rootMargin`: CSS-style margin that grows/shrinks the root box (trigger early with positive values). `threshold`: visibility ratio(s) (0 to 1) at which the callback fires.

> [!question]- Why use `IntersectionObserver` instead of a `scroll` listener?
> It's asynchronous, batched, runs off the hot path, and avoids forcing layout with `getBoundingClientRect` on every scroll event.

> [!question]- What is `ResizeObserver` for?
> Detecting size changes of individual elements (not just the window), e.g. resizing charts, responsive components, or auto-growing textareas.

> [!question]- What is `MutationObserver`?
> An API to react to DOM changes: child additions/removals, attribute changes, and text changes. Callbacks are batched and run as microtasks.

> [!question]- What can go wrong with `MutationObserver` callbacks?
> If the callback modifies the observed DOM, it triggers itself again (infinite loop). Disconnect first, narrow the observation, or add a guard.

> [!question]- How do you stop an observer?
> `unobserve(target)` for one element or `disconnect()` for all. Do this in cleanup (React `useEffect` return) to avoid leaks.

> [!question]- Why might an `IntersectionObserver` not fire again for infinite scroll?
> It only fires on **changes**. If the sentinel remains visible after loading, nothing changes. Re-observe it or check manually.

> [!question]- What is the `ResizeObserver loop` error?
> A warning when a resize callback changes sizes in a way that triggers more notifications in the same frame. Usually benign; avoid feedback loops or defer changes with `requestAnimationFrame`.

> [!question]- Is the `IntersectionObserver` callback invoked initially?
> Yes. After `observe()`, you get an initial entry with the element's current intersection state.

---

## 8. Predict the output

```html
<div id="outer"><div id="inner"><button id="btn"><span id="label">Go</span></button></div></div>
```

```js
// 1  (click the button)
outer.addEventListener("click", () => console.log("outer"));
inner.addEventListener("click", () => console.log("inner"));
btn.addEventListener("click", () => console.log("btn"));

// 2  (click the button)
outer.addEventListener("click", () => console.log("outer-capture"), { capture: true });
outer.addEventListener("click", () => console.log("outer-bubble"));
inner.addEventListener("click", () => console.log("inner"));
btn.addEventListener("click", () => console.log("btn"));

// 3  (click the button)
inner.addEventListener("click", (e) => { console.log("inner"); e.stopPropagation(); });
outer.addEventListener("click", () => console.log("outer"));
btn.addEventListener("click", () => console.log("btn"));

// 4  (click the button)
btn.addEventListener("click", (e) => { console.log("first"); e.stopImmediatePropagation(); });
btn.addEventListener("click", () => console.log("second"));
outer.addEventListener("click", () => console.log("outer"));

// 5  (click the span inside the button; listener is on outer)
outer.addEventListener("click", (e) => console.log(e.target.id, e.currentTarget.id));

// 6  (click the button)
outer.addEventListener("click", (e) => console.log(e.eventPhase), { capture: true });
btn.addEventListener("click", (e) => console.log(e.eventPhase));
outer.addEventListener("click", (e) => console.log(e.eventPhase));

// 7  (focus an <input> inside #outer)
outer.addEventListener("focus", () => console.log("focus"));
outer.addEventListener("focusin", () => console.log("focusin"));

// 8
const cb = document.createElement("input");
cb.type = "checkbox";
cb.addEventListener("click", (e) => e.preventDefault());
cb.click();
console.log(cb.checked);

// 9  (a link inside #outer)
const a = document.createElement("a");
a.href = "https://example.com";
a.addEventListener("click", (e) => { e.preventDefault(); console.log("a"); });
outer.addEventListener("click", (e) => console.log("outer", e.defaultPrevented));
outer.append(a);
a.click();

// 10
const log = debounce((x) => console.log(x), 100);
log(1); log(2);
setTimeout(() => log(3), 50);
setTimeout(() => log(4), 300);

// 11  (leading-only throttle, 100 ms)
const t = throttle((x) => console.log(x), 100);
t("a"); t("b");
setTimeout(() => t("c"), 50);
setTimeout(() => t("d"), 150);

// 12
localStorage.setItem("n", 5);
localStorage.setItem("b", false);
localStorage.setItem("o", { a: 1 });
console.log(typeof localStorage.getItem("n"),
            localStorage.getItem("b") ? "truthy" : "falsy",
            localStorage.getItem("o"));

// 13
console.log(localStorage.getItem("missing"), JSON.parse(localStorage.getItem("missing")));

// 14
document.cookie = "a=1; path=/";
document.cookie = "b=2; path=/";
console.log(document.cookie);

// 15  (#target is already visible in the viewport)
new IntersectionObserver((entries) => {
  console.log(entries[0].isIntersecting ? "visible" : "hidden");
}).observe(document.querySelector("#target"));

// 16
const mo = new MutationObserver(() => console.log("mutation"));
mo.observe(document.body, { childList: true });
document.body.append(document.createElement("div"));
console.log("sync");
Promise.resolve().then(() => console.log("promise"));
setTimeout(() => console.log("timeout"), 0);

// 17  (click the span inside the button)
btn.addEventListener("click", (e) => console.log(e.target.matches("button"), !!e.target.closest("button")));

// 18
const parent = document.createElement("div");
const child = document.createElement("span");
parent.append(child);
parent.addEventListener("x", () => console.log("parent"));
child.dispatchEvent(new CustomEvent("x"));
child.dispatchEvent(new CustomEvent("x", { bubbles: true }));
```

<details>
<summary>Answers</summary>

1. `btn`, `inner`, `outer` (bubbling, bottom-up)
2. `outer-capture`, `btn`, `inner`, `outer-bubble`
3. `btn`, `inner` (`inner` stops propagation, so `outer` never runs)
4. `first`, `outer` (`stopImmediatePropagation` skips `second` and also stops bubbling)
5. `label outer` (`target` is the span; `currentTarget` is `outer`)
6. `1`, `2`, `3` (capture on outer, target on the button, bubble on outer)
7. `focusin` only (`focus` doesn't bubble)
8. `false` (`preventDefault` in the click handler reverts the checkbox)
9. `a`, then `outer true` (the event still bubbles; `defaultPrevented` is `true`)
10. `3`, then `4` (the timer restarts at 50 ms, so `3` fires at ~150 ms; `4` fires at ~400 ms)
11. `a`, then `d` (`b` and `c` fall inside the first 100 ms window and are dropped)
12. `string truthy [object Object]`
13. `null null`
14. `a=1; b=2` (the order may vary by browser)
15. `visible` (an initial callback fires right after `observe()`)
16. `sync`, `mutation`, `promise`, `timeout` (the mutation microtask was queued before the promise)
17. `false true` (`target` is the span; `closest` finds the button)
18. `parent` once (only the second dispatch bubbles)

</details>

---

## 9. Coding practice

**1. `debounce` with `cancel` (and leading/trailing)**: see section 4 for the full version.
```js
function debounce(fn, wait = 300) {
  let timer;
  function debounced(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => { timer = undefined; fn.apply(this, args); }, wait);
  }
  debounced.cancel = () => { clearTimeout(timer); timer = undefined; };
  return debounced;
}
```

**2. `throttle` (leading + trailing)**: see section 4.

**3. `requestAnimationFrame` throttle**
```js
const rafThrottle = (fn) => {
  let ticking = false;
  return (...args) => {
    if (ticking) return;
    ticking = true;
    requestAnimationFrame(() => { ticking = false; fn(...args); });
  };
};
```

**4. Delegated todo list with `data-action`**
```js
todos.addEventListener("click", (e) => {
  const btn = e.target.closest("[data-action]");
  if (!btn || !todos.contains(btn)) return;
  const li = btn.closest("li");
  if (btn.dataset.action === "done") li.classList.toggle("done");
  if (btn.dataset.action === "delete") li.remove();
});
```

**5. Delegated table: edit/delete rows (works for rows added later)**
```js
table.addEventListener("click", (e) => {
  const row = e.target.closest("tr[data-id]");
  if (!row || !table.contains(row)) return;
  const id = row.dataset.id;
  if (e.target.closest(".edit")) openEditor(id);
  else if (e.target.closest(".delete")) { deleteRecord(id); row.remove(); }
});
```

**6. "Click outside" without `stopPropagation`**
```js
function onOutsideClick(el, handler) {
  const listener = (e) => { if (!e.composedPath().includes(el)) handler(e); };
  document.addEventListener("pointerdown", listener);
  return () => document.removeEventListener("pointerdown", listener);   // cleanup
}
const off = onOutsideClick(dropdown, () => dropdown.classList.remove("open"));
```

**7. Event bus with `EventTarget` and `CustomEvent`**
```js
const bus = new EventTarget();
const on = (type, fn) => { bus.addEventListener(type, fn); return () => bus.removeEventListener(type, fn); };
const emit = (type, detail) => bus.dispatchEvent(new CustomEvent(type, { detail }));

const off = on("cart:add", (e) => console.log("added", e.detail));
emit("cart:add", { id: 1 });
off();
```

**8. Safe storage wrapper with expiry**: see section 5 (`store.set / get / remove`).

**9. Cookie helpers**: see section 5 (`cookies.set / get / remove`).

**10. Sync state across tabs**
```js
window.addEventListener("storage", (e) => {
  if (e.storageArea !== localStorage) return;
  if (e.key === "theme") applyTheme(e.newValue);
  if (e.key === "logout") location.href = "/login";
});
```

**11. Lazy-load images with `IntersectionObserver`**: see section 6.1.

**12. Infinite scroll with a sentinel**: see section 6.1.

**13. `useInView` and `useDebouncedValue` hooks (React)**
```jsx
function useInView(options) {
  const ref = React.useRef(null);
  const [inView, setInView] = React.useState(false);
  React.useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const io = new IntersectionObserver(([entry]) => setInView(entry.isIntersecting), options);
    io.observe(el);
    return () => io.disconnect();
  }, [options?.root, options?.rootMargin]);       // keep `options` stable (primitive deps or memoized)
  return [ref, inView];
}

function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = React.useState(value);
  React.useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

**14. Resize a chart/canvas with `ResizeObserver`**
```js
const ro = new ResizeObserver(([entry]) => {
  const { width, height } = entry.contentRect;
  canvas.width = width;
  canvas.height = height;
  redraw();
});
ro.observe(container);
// later: ro.disconnect();
```

**15. `waitForElement` with `MutationObserver`**: see section 6.3.

**16. Scroll-spy navigation with `IntersectionObserver`**: see section 6.1.

**17. Debounced search that cancels stale requests**
```js
let controller;
const search = debounce(async (q) => {
  controller?.abort();
  controller = new AbortController();
  try {
    const data = await getJson(`/api/search?q=${encodeURIComponent(q)}`, { signal: controller.signal });
    render(data);
  } catch (err) {
    if (err.name !== "AbortError") showError(err);
  }
}, 300);

input.addEventListener("input", (e) => search(e.target.value.trim()));
```

**18. Remove all listeners at once with `AbortController`**
```js
const ac = new AbortController();
const { signal } = ac;
window.addEventListener("resize", onResize, { signal });
document.addEventListener("keydown", onKey, { signal });
button.addEventListener("click", onClick, { signal });
// cleanup:
ac.abort();
```

---

## 10. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Event phases | Capture (1) → target (2) → bubble (3) |
| Listen during capture | `{ capture: true }` |
| Which element was clicked / where is the listener | `e.target` / `e.currentTarget` |
| Full path of the event | `e.composedPath()` |
| One handler for many (future) children | Delegation: `e.target.closest(sel)` + `container.contains(match)` |
| Delegate `focus` / `mouseenter` | `focusin` / `mouseover` (these bubble) |
| Custom event that bubbles | `new CustomEvent("x", { detail, bubbles: true })` |
| Cancel browser default | `e.preventDefault()` (needs `cancelable`, not `passive`) |
| Stop reaching other elements | `e.stopPropagation()` (use sparingly) |
| Stop other listeners on the same element too | `e.stopImmediatePropagation()` |
| Click outside to close | `document` listener + `composedPath().includes(el)` |
| Run once after the user stops | **Debounce** |
| Run at most every N ms | **Throttle** (or `requestAnimationFrame`) |
| Debounce in React | `useDebouncedValue` / memoized function + cleanup |
| Stale search responses | Debounce **plus** `AbortController` |
| Small non-sensitive persistent data | `localStorage` (strings only → `JSON`) |
| Per-tab temporary data | `sessionStorage` |
| Server needs it / auth session | Cookie: `HttpOnly; Secure; SameSite` |
| Big/structured/offline data | IndexedDB |
| Cross-tab sync | `storage` event (other tabs only) or `BroadcastChannel` |
| Storage can fail | Wrap in `try/catch` (`QuotaExceededError`, `SecurityError`) |
| Delete a cookie | Same name + path/domain, `max-age=0` |
| Visible on screen? | `IntersectionObserver` (`rootMargin` to preload) |
| Element size changed? | `ResizeObserver` |
| DOM changed? | `MutationObserver` (disconnect before mutating inside its callback) |
| Stop observing | `unobserve(el)` / `disconnect()` |
| Many listeners to clean up | `{ signal }` + `AbortController.abort()` |

---

## Related notes
- [[JS 04 - DOM & Events Basics]]
- [[JS 08 - Async JavaScript]]
- [[JS 10 - Error Handling]]
- [[JS 12 - Tooling Basics]]
- [[JS 15 - Performance]]
- [[JS 16 - Security]]
