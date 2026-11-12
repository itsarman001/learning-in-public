# JS 08 — Async JavaScript

> [!info] Scope
> Callbacks · Promises · `Promise.all` / `allSettled` / `race` / `any` · `async` / `await` · Event loop · Timers · `fetch` + `AbortController`
> Previous: [[JS 07 - ES6+ Features]] · Next: [[JS 09 - OOP in JS]]
> This is one of the **most-asked interview groups**, especially event loop ordering and promise combinators. Expect "predict the output" on every section.

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| JavaScript | Single-threaded. One call stack. Async work is handed to the browser/Node and comes back through **queues**. |
| Callback | A function called later. Easy to write, hard to compose (callback hell, no built-in error flow). |
| Promise | An object for a future value: `pending` → `fulfilled` or `rejected`. Settles **once**. |
| `.then()` | Returns a **new promise**. Return a value to continue the chain; throw to jump to `.catch`. |
| `.finally()` | Runs either way. Takes no argument and passes the original value/error through. |
| `Promise.all` | All must succeed. Rejects on the **first** failure. Results keep input order. |
| `Promise.allSettled` | Waits for all. Never rejects. Gives `{status, value/reason}` per item. |
| `Promise.race` | First to **settle** (success or failure) wins. |
| `Promise.any` | First to **fulfill** wins. All rejected → `AggregateError`. |
| `async` function | Always returns a promise. `await` pauses *that function*, not the whole program. |
| Parallel vs sequential | `await` inside a loop is sequential. `Promise.all(arr.map(...))` is parallel. |
| Event loop order | Sync code → **all microtasks** (promises, `queueMicrotask`) → one macrotask (timers, events) → repeat. |
| Timers | `setTimeout(fn, 0)` runs **after** the current code and all microtasks. Delay is a minimum. |
| `fetch` | Doesn't reject on `404`/`500`. Check `response.ok`. It rejects only on network failure or abort. |
| `AbortController` | `controller.abort()` cancels a fetch. It rejects with `AbortError`. |

**Mental model**

| Piece | What it does |
|---|---|
| **Call stack** | Runs synchronous code, one frame at a time |
| **Web APIs / Node APIs** | Timers, `fetch`, DOM events, I/O run *outside* the stack |
| **Microtask queue** | Promise reactions (`.then/.catch/.finally`), `await` continuations, `queueMicrotask`, `MutationObserver` |
| **Macrotask (task) queue** | `setTimeout`, `setInterval`, DOM events, I/O callbacks, `MessageChannel` |
| **Event loop** | When the stack is empty: run **all** microtasks, (maybe render), then take **one** macrotask |

---

## 1. Callbacks

### Definition
A **callback** is a function passed to another function to be called later: immediately (**synchronous**, like `arr.map(fn)`) or after something finishes (**asynchronous**, like `setTimeout(fn)` or an event handler).

### Summary

```js
// Synchronous callback
[1, 2, 3].forEach(n => console.log(n));

// Asynchronous callbacks
setTimeout(() => console.log("later"), 1000);
button.addEventListener("click", () => console.log("clicked"));

// Error-first callback (Node convention)
fs.readFile("data.json", "utf8", (err, data) => {
  if (err) return console.error(err);
  console.log(JSON.parse(data));
});
```

**Callback hell (pyramid of doom)**
```js
getUser(id, (err, user) => {
  if (err) return handle(err);
  getPosts(user.id, (err, posts) => {
    if (err) return handle(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return handle(err);
      render(comments);
    });
  });
});
```

**Why callbacks became painful**

| Problem | Explanation |
|---|---|
| Nesting | Sequential steps drift to the right; hard to read |
| Error handling | Every level must check `err` manually; `try/catch` can't catch async errors |
| Inversion of control | You hand your logic to someone else's code (it may call it twice, never, or too early) |
| Composition | Running things in parallel or in order needs hand-written bookkeeping |
| Inconsistent timing | A function that sometimes calls back synchronously and sometimes asynchronously ("Zalgo") |

**Fix: promisify** a callback API
```js
const promisify = fn => (...args) =>
  new Promise((resolve, reject) =>
    fn(...args, (err, data) => (err ? reject(err) : resolve(data))));

const readFileP = promisify(fs.readFile);
```

### Edge cases
- **`try/catch` doesn't catch async errors:**
  ```js
  try { setTimeout(() => { throw new Error("boom"); }, 0); } catch (e) { /* never runs */ }
  ```
- **Callback called more than once (or never)** is a bug source in third-party APIs. Promises settle only once.
- **Always async or always sync:** mixing causes ordering bugs. Don't `return cb(cached)` synchronously in one branch and call it later in another.
- **`this` is lost** when you pass a method as a callback (`setTimeout(obj.method)`). Use an arrow function or `bind` ([[JS 05 - Core Concepts (Scope, Closures, this)]]).
- **Calling instead of passing:** `setTimeout(run(), 1000)` runs `run` immediately. Pass `run` or `() => run()`.
- **Extra arguments:** array callbacks receive `(item, index, array)`. `["1","2"].map(parseInt)` → `[1, NaN]`.
- **Error-first convention:** the first argument is the error (`null` when fine). Don't forget `return` after handling the error.
- **Callbacks keep variables alive** (closures), which can cause memory leaks if listeners are never removed.

### Use cases
- Event handlers, timers, array methods, and older Node APIs.
- Anywhere a library hands you a hook (`onSuccess`, `onChange`).
- Still fine for simple, single-step async work. Prefer promises for sequences.

---

## 2. Promises

### Definition
A **Promise** is an object representing the eventual result (or failure) of an asynchronous operation. It's always in one of three states:

| State | Meaning |
|---|---|
| **pending** | Initial; not finished |
| **fulfilled** | Finished successfully with a value |
| **rejected** | Failed with a reason (error) |

*Settled* = fulfilled or rejected. Once settled, a promise **never changes** again.

### Summary

```js
const p = new Promise((resolve, reject) => {
  // executor runs SYNCHRONOUSLY, immediately
  setTimeout(() => resolve("done"), 1000);   // or reject(new Error("failed"))
});

p.then(value => console.log(value))          // runs if fulfilled
 .catch(err => console.error(err))           // runs if rejected (or if a then-callback threw)
 .finally(() => console.log("cleanup"));     // runs either way
```

**Chaining: each `.then` returns a new promise**
```js
fetch("/api/user")
  .then(res => res.json())                // returns a promise → chain waits for it
  .then(user => user.name.toUpperCase())  // returns a plain value → wrapped in a fulfilled promise
  .then(name => { throw new Error("oops"); })  // throwing → rejected promise
  .catch(err => "fallback")               // returning from catch → chain continues as fulfilled
  .then(v => console.log(v));             // "fallback"
```

**What a `.then` callback returns decides the next promise**

| Callback does | Next promise |
|---|---|
| Returns a value | Fulfilled with that value |
| Returns a promise/thenable | Adopts that promise's result (waits for it) |
| Returns nothing | Fulfilled with `undefined` |
| Throws | Rejected with the error |

**Creating and shortcuts**
```js
Promise.resolve(42);                  // already fulfilled
Promise.reject(new Error("no"));      // already rejected
const sleep = ms => new Promise(r => setTimeout(r, ms));

// ES2024 (check support): const { promise, resolve, reject } = Promise.withResolvers();
```

**`then`, `catch`, `finally`**

| Method | Receives | Return value effect |
|---|---|---|
| `then(onFulfilled, onRejected)` | The value / reason | Becomes the next value (or the next promise's result) |
| `catch(onRejected)` | The reason | Same as `then(undefined, onRejected)`; returning recovers the chain |
| `finally(fn)` | **Nothing** | Return value is **ignored** (passes the original result through), unless `fn` throws/rejects |

**Unhandled rejections**
- Browser: logs `Uncaught (in promise)` and fires the `unhandledrejection` event on `window`.
- Node 15+: an unhandled rejection **terminates the process** by default.
- Always end chains with `.catch(...)` (or use `try/catch` with `await`).

### Edge cases
- **Forgetting to `return` inside `.then`** breaks the chain: the next `.then` receives `undefined` and doesn't wait.
  ```js
  p.then(() => { fetchMore(); })        // not returned → next step doesn't wait
  p.then(() => fetchMore())             // returned → chain waits
  ```
- **`.then` callbacks are always asynchronous** (microtask), even if the promise is already fulfilled. The **executor** is the only part that runs synchronously.
  ```js
  new Promise(r => { console.log(1); r(); console.log(2); }).then(() => console.log(3));
  console.log(4);   // 1 2 4 3
  ```
- **`resolve`/`reject` only work once:** later calls are ignored. Code after `resolve()` still runs (use `return resolve()`).
- **Throwing inside the executor** rejects the promise. Throwing *after* it's settled is ignored.
- **Resolving with a promise** adopts it: `resolve(otherPromise)` follows `otherPromise`.
- **`catch` recovers:** after a `.catch` that returns normally, the chain is **fulfilled** again. Re-`throw` to keep it rejected.
- **`then(a, b)` vs `then(a).catch(b)`:** in the first form, `b` does *not* catch errors thrown by `a`.
- **Passing a call instead of a function:** `.then(doThing())` calls `doThing` immediately. Use `.then(doThing)` or `.then(() => doThing())`.
- **Nesting promises** (promise inside `.then` without returning) recreates callback hell. Flatten by returning.
- **`new Promise(async (resolve) => ...)`** is an anti-pattern: errors thrown inside the async executor are lost. Use an `async` function directly.
- **`finally` doesn't receive the value** and can't change it (unless it throws).
- **Rejecting with non-Errors** (`reject("oops")`) loses the stack trace. Reject with `new Error(...)`.
- **`Promise.resolve(p)` returns `p` itself** if it's already a native promise.
- **Promises are eager:** the work starts as soon as the promise is created, not when `.then` is attached.
- **Promises can't be cancelled** by themselves. Use `AbortController` (section 7).

### Use cases
- Any one-time async result: network requests, file reads, timers, animations.
- Composing sequences (`.then` chains) and parallel work (`Promise.all`).
- Wrapping callback APIs and building timeouts/retries.

---

## 3. Promise combinators: `all`, `allSettled`, `race`, `any`

### Definition
Static methods that take an iterable of promises (or plain values) and return **one** promise representing the group.

### Summary

| Method | Fulfills when | Rejects when | Result |
|---|---|---|---|
| `Promise.all` | **All** fulfill | **Any one** rejects (first rejection) | Array of values, **input order** |
| `Promise.allSettled` | **All** settle | Never | Array of `{ status, value }` / `{ status, reason }` |
| `Promise.race` | **First to settle** fulfills | **First to settle** rejects | That promise's value/reason |
| `Promise.any` | **First to fulfill** | **All** reject → `AggregateError` | First fulfilled value |

```js
// all: parallel, fail-fast
const [user, posts] = await Promise.all([getUser(1), getPosts(1)]);

// allSettled: collect every outcome
const results = await Promise.allSettled([p1, p2, p3]);
// [{status:"fulfilled", value:1}, {status:"rejected", reason:Error}, ...]
const ok = results.filter(r => r.status === "fulfilled").map(r => r.value);

// race: timeout pattern
const data = await Promise.race([
  fetch(url),
  new Promise((_, reject) => setTimeout(() => reject(new Error("timeout")), 5000)),
]);

// any: fastest successful mirror
const fastest = await Promise.any([fetch(mirror1), fetch(mirror2)]);
```

**Choosing one**

| Situation | Use |
|---|---|
| Need every result; one failure means the whole thing fails | `all` |
| Need to know what succeeded and what failed (dashboards, batch jobs) | `allSettled` |
| Timeouts, "first response wins" including errors | `race` |
| First *successful* result, ignore failures (mirrors, fallbacks) | `any` |

### Edge cases
- **`all` rejects immediately** on the first rejection, but the other promises **keep running** (they aren't cancelled); their results are ignored.
- **Order is preserved** in `all`/`allSettled` results regardless of completion order.
- **Empty iterable:** `all([])` → fulfills `[]`; `allSettled([])` → `[]`; `race([])` → **pending forever**; `any([])` → rejects with `AggregateError`.
- **Non-promise values are allowed:** `Promise.all([1, Promise.resolve(2)])` → `[1, 2]`.
- **`race` settles with the first settled**, even if it's a rejection. A fast failure beats a slower success. Use `any` if you only want success.
- **`any` rejection** is an `AggregateError` with `.errors` (array of reasons in input order).
- **`allSettled` never rejects**, so you must inspect `status` yourself. Forgetting this hides errors.
- **Sequential vs parallel:** `Promise.all(ids.map(fetchUser))` runs in parallel. A `for...of` with `await` runs one at a time.
- **Too much parallelism** (thousands of requests at once) can overwhelm servers or the browser. Limit concurrency (see coding practice).
- **Promises start immediately when created:** `const ps = ids.map(fetchUser)` already fires all requests before `Promise.all` is called.
- **`race` for timeouts** doesn't cancel the slow operation. Combine with `AbortController`.
- **Passing a non-iterable** throws a `TypeError`.

### Use cases
- `all`: load page data in parallel (user + settings + notifications).
- `allSettled`: uploading multiple files and reporting which failed.
- `race`: request timeouts, "whichever loads first".
- `any`: redundancy (multiple CDNs/endpoints), feature fallbacks.

---

## 4. `async` / `await`

### Definition
**`async`/`await`** is syntax built on promises that lets async code read like synchronous code.
- An **`async` function** always returns a **promise**.
- **`await`** pauses *that async function* until a promise settles, then returns its value (or throws its rejection). The rest of the program keeps running.

### Summary

```js
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const user = await res.json();
    return user;                        // wrapped in a fulfilled promise
  } catch (err) {
    console.error(err);                 // handles rejections AND thrown errors
    throw err;                          // rethrow to let the caller handle it
  } finally {
    hideSpinner();
  }
}

loadUser(1).then(console.log);          // an async function returns a promise
```

**Equivalent forms**
```js
// async/await
async function f() { const v = await p; return v + 1; }
// promises
function f() { return p.then(v => v + 1); }
```

**Sequential vs parallel**
```js
// Sequential (slow when the calls are independent): total = a + b
const a = await getA();
const b = await getB();

// Parallel: total ≈ max(a, b)
const [a2, b2] = await Promise.all([getA(), getB()]);

// Parallel start, await later
const pa = getA(), pb = getB();         // both started
const a3 = await pa, b3 = await pb;
```

**Loops**
```js
// Sequential (order matters / rate limits)
for (const id of ids) {
  results.push(await fetchUser(id));
}

// Parallel
const results = await Promise.all(ids.map(fetchUser));

// WRONG: forEach doesn't wait
ids.forEach(async id => { await save(id); });
console.log("done");   // runs before the saves finish

// Async iteration
for await (const chunk of asyncIterable) { /* ... */ }
```

**Where `await` is allowed**

| Place | Allowed? |
|---|---|
| Inside an `async` function / method / arrow | Yes |
| Top level of an **ES module** (top-level `await`) | Yes |
| Top level of a classic script / inside a regular function | No (SyntaxError) |
| Inside a class constructor | No (constructors can't be `async`) |

### Edge cases
- **An async function always returns a promise:** `return 1` → `Promise<1>`. Throwing → a rejected promise. Forgetting `await` gives you a promise instead of a value.
- **Forgetting `await`:** `const user = fetchUser(); user.name` → `undefined` (it's a promise). Also leaves a "floating" promise whose errors are unhandled.
- **`return promise` vs `return await promise` inside `try/catch`:**
  ```js
  async function a() { try { return failingCall(); }       catch { return "caught"; } }  // NOT caught
  async function b() { try { return await failingCall(); } catch { return "caught"; } }  // caught
  ```
  Without `await`, the function returns before the rejection happens, so `catch` doesn't run.
- **`await` always yields,** even on a non-promise: `await 5` still defers the rest of the function to a microtask.
- **`await` in `forEach`/`map` callbacks** doesn't pause the outer function. Use `for...of`, or `Promise.all(arr.map(async ...))`.
- **Unnecessary sequential awaits** are the most common performance bug. If calls don't depend on each other, run them in parallel.
- **Error handling:** one `try/catch` can wrap many awaits. `await Promise.all` throws the first rejection.
- **Parallel awaits and rejections:** `const pa = a(), pb = b(); await pa; await pb;`: if `pb` rejects while you're still awaiting `pa`, you can get an unhandled rejection warning. Prefer `Promise.all`.
- **`async` arrow in event handlers:** errors become unhandled rejections unless you `try/catch`.
- **`async` doesn't make code run in a thread.** A CPU-heavy loop inside an `async` function still blocks everything.
- **Top-level `await`** blocks the importing modules until it resolves.
- **`await` works on thenables** (any object with a `then` method), not just native promises.
- **Returning from `finally`** overrides the try/catch return value. Avoid.
- **`await` is not "blocking":** other tasks (events, timers) keep running while the function is suspended.

### Use cases
- Sequential steps that depend on each other (login → get profile → load data).
- Cleaner error handling with `try/catch/finally`.
- Any API-heavy code (data fetching, uploads, form submissions).

---

## 5. The event loop

### Definition
JavaScript runs on a **single thread** with one call stack. The **event loop** is the mechanism that moves finished async work (callbacks, promise reactions, events) from queues onto the stack **when the stack is empty**. It's what makes non-blocking behavior possible.

### Summary

**One turn of the loop (browser)**
1. Run the current script / one **macrotask** to completion (stack becomes empty).
2. Run **all microtasks** until the microtask queue is empty (including microtasks queued by microtasks).
3. If it's time to paint: run `requestAnimationFrame` callbacks, then style/layout/paint.
4. Take the **next macrotask** and repeat.

| Queue | Examples | Runs |
|---|---|---|
| **Call stack** | Synchronous code | Immediately |
| **Microtasks** | `.then/.catch/.finally`, code after `await`, `queueMicrotask`, `MutationObserver` | After the current task, **before** anything else |
| **Macrotasks (tasks)** | `setTimeout`, `setInterval`, DOM events (click…), I/O, `MessageChannel`, `setImmediate` (Node) | One per loop turn |
| **Render step** | `requestAnimationFrame`, style, layout, paint | Between tasks, when the browser decides to paint |

**The classic ordering**
```js
console.log("1");                                   // sync
setTimeout(() => console.log("2"), 0);              // macrotask
Promise.resolve().then(() => console.log("3"));     // microtask
console.log("4");                                   // sync
// 1 4 3 2
```

**A harder one**
```js
console.log("start");
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise1")).then(() => console.log("promise2"));
queueMicrotask(() => console.log("microtask"));
(async () => {
  console.log("async start");
  await null;
  console.log("async after await");
})();
console.log("end");
// start, async start, end, promise1, microtask, async after await, promise2, timeout
```

How to read it: all synchronous logs run first (`start`, `async start`, `end`). Microtasks then run in the order they were queued (`promise1`, `microtask`, `async after await`); `promise2` was queued by `promise1` so it goes to the back. The `setTimeout` macrotask runs last.

**Blocking the loop**
```js
button.addEventListener("click", () => {
  for (let i = 0; i < 1e9; i++) {}   // UI frozen: no rendering, no events, timers late
});
```
Fixes: split work into chunks (`setTimeout` / `requestIdleCallback`), use Web Workers, or move work to the server ([[JS 15 - Performance]]).

**Node.js specifics**

| | Order in Node |
|---|---|
| 1 | Synchronous code |
| 2 | `process.nextTick` queue (before promise microtasks) |
| 3 | Promise microtasks |
| 4 | Event-loop phases: timers → pending callbacks → poll (I/O) → check (`setImmediate`) → close callbacks |

### Edge cases
- **Microtasks run before the next macrotask,** so a promise `.then` always beats `setTimeout(..., 0)`.
- **Microtask starvation:** a microtask that keeps queuing new microtasks (e.g. an endless `.then` loop or recursive `queueMicrotask`) blocks timers, events, **and rendering** forever.
- **`await` splits a function into synchronous and microtask parts:** code before the first `await` runs synchronously; the rest runs as a microtask continuation.
- **`async` functions run synchronously until their first `await`:**
  ```js
  async function a() { console.log(1); await b(); console.log(2); }
  async function b() { console.log(3); }
  a(); console.log(4);   // 1 3 4 2
  ```
- **`requestAnimationFrame` isn't a task or microtask:** it runs right before the next paint (usually ~60 times per second) and pauses in background tabs.
- **`setTimeout(fn, 0)` isn't immediate:** it still waits for the current stack, all microtasks, and possibly rendering. It also has a minimum delay (~1 ms; nested timers are clamped to ≥ 4 ms after 5 levels).
- **Heavy synchronous work delays everything,** including timers and clicks. `async` doesn't help.
- **Event handlers are macrotasks;** a click handler's promises resolve in its microtask checkpoint right after the handler.
- **Rendering happens between tasks,** not between microtasks. Many DOM changes inside one task/microtask chain paint once at the end.
- **`setTimeout` vs `setImmediate` in Node's main module:** order isn't guaranteed. Inside an I/O callback, `setImmediate` always runs first.
- **Two `setTimeout`s with the same delay run in call order.** A shorter delay set later can still run first.
- **`queueMicrotask(fn)` vs `setTimeout(fn, 0)`:** the first is much earlier, but also more dangerous (it can starve the loop).

### Use cases
- Explaining output order in interviews and debugging "why did this log first?" issues.
- Deciding how to defer work (`queueMicrotask`, `setTimeout`, `requestAnimationFrame`, `requestIdleCallback`).
- Keeping UIs responsive by not blocking the stack.

---

## 6. Timers

### Definition
Timers schedule a callback to run later. They are **Web APIs** (not part of the language core): the browser/Node tracks the time, then queues your callback as a macrotask.

### Summary

| API | Purpose | Cancel with |
|---|---|---|
| `setTimeout(fn, ms, ...args)` | Run **once** after at least `ms` | `clearTimeout(id)` |
| `setInterval(fn, ms, ...args)` | Run **repeatedly** every ~`ms` | `clearInterval(id)` |
| `requestAnimationFrame(fn)` | Run before the next repaint (animations) | `cancelAnimationFrame(id)` |
| `requestIdleCallback(fn)` | Run when the browser is idle (low priority) | `cancelIdleCallback(id)` |
| `queueMicrotask(fn)` | Run in the microtask queue | n/a |

```js
const id = setTimeout(() => console.log("once"), 1000);
clearTimeout(id);                                   // cancel before it fires

const tick = setInterval(() => console.log("tick"), 1000);
setTimeout(() => clearInterval(tick), 5000);        // stop after 5s

setTimeout((a, b) => console.log(a + b), 100, 1, 2);   // extra args → 3

const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
await sleep(500);

// Recursive setTimeout: waits for the previous run to finish
async function poll() {
  const data = await fetchStatus();
  if (!data.done) setTimeout(poll, 2000);
}
```

**`setInterval` vs recursive `setTimeout`**

| | `setInterval` | Recursive `setTimeout` |
|---|---|---|
| Spacing | Fixed schedule from the start | Delay counted from when the previous run *finished* |
| If the callback is slow | Runs can bunch up / overlap in time | Never overlaps |
| Async callbacks | Doesn't wait for them | Can `await`, then schedule the next |
| Best for | Simple clocks/tickers | Polling, retries, variable delays |

### Edge cases
- **The delay is a *minimum*,** not a guarantee. A busy call stack, other tasks, or background-tab throttling make it later.
- **Minimum/clamped delays:** browsers clamp nested timers (after ~5 levels) to ≥ 4 ms; hidden tabs are throttled (often to once per second or less).
- **`setTimeout(fn(), 1000)`** calls `fn` immediately and schedules its *return value*. Pass `fn` or `() => fn()`.
- **`this`** inside a regular-function callback is `window`/`undefined`, not your object. Use an arrow function.
- **Maximum delay is `2^31 − 1` ms (~24.8 days);** larger values overflow and fire almost immediately.
- **`setInterval` doesn't wait for async callbacks;** if each run takes longer than the interval, runs pile up. Use recursive `setTimeout`.
- **Always clear timers on cleanup** (component unmount, page change). Forgotten timers leak memory and keep running (`useEffect` cleanup in React).
- **Timer IDs:** browsers return a number; Node returns a `Timeout` object (`typeof` differs). Passing `undefined` to `clearTimeout` is safe.
- **`setTimeout(fn, 0)` vs `Promise.resolve().then(fn)`:** the promise runs first (microtask beats macrotask).
- **Don't use timers for exact timing:** compute elapsed time with `Date.now()` / `performance.now()` for countdowns, not by counting ticks.
- **`setTimeout` with a string** (`setTimeout("code()", 100)`) evaluates code like `eval`. Never do this.
- **Animations:** prefer `requestAnimationFrame` (or CSS) over `setInterval` for smooth 60 fps and battery use.
- **A closure in a timer callback keeps its variables alive** until the timer fires or is cleared.

### Use cases
- Debounce/throttle (see [[JS 11 - Browser & DOM Advanced]]), delayed UI messages (toasts), auto-save.
- Polling an API, retry backoff, timeouts (`Promise.race` + timer).
- Animation loops (`requestAnimationFrame`), idle-time background work.

---

## 7. `fetch` + `AbortController`

### Definition
- **`fetch(url, options)`** makes an HTTP request and returns a **promise that resolves to a `Response`** as soon as the response **headers** arrive. The body is read with a second promise (`res.json()`, `res.text()`, ...).
- **`AbortController`** lets you cancel one or more async operations (fetch, event listeners, streams) through its **`signal`**.

### Summary

**Basic GET with proper error handling**
```js
async function getJson(url, { signal } = {}) {
  const res = await fetch(url, { signal });
  if (!res.ok) throw new Error(`HTTP ${res.status} ${res.statusText}`);   // fetch doesn't throw on 404/500
  return res.json();
}
```

**POST with JSON**
```js
const res = await fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Asha" }),
  credentials: "include",            // send cookies cross-origin (server must allow)
});
const created = await res.json();
```

**Useful `Response` members**

| Member | Meaning |
|---|---|
| `res.ok` | `true` for status 200–299 |
| `res.status` / `res.statusText` | e.g. `404` / `"Not Found"` |
| `res.headers.get("content-type")` | Read a header |
| `res.json()` / `res.text()` / `res.blob()` / `res.formData()` | Read the body (each returns a promise; body can be read **once**) |
| `res.clone()` | Copy the response so you can read the body twice |

**What `fetch` rejects on, and what it doesn't**

| Situation | Promise |
|---|---|
| Network failure, DNS error, CORS block | **Rejects** (`TypeError`) |
| `abort()` called | **Rejects** (`AbortError`) |
| HTTP `404`, `500`, any status | **Resolves** (check `res.ok`) |
| Invalid JSON body in `res.json()` | `res.json()` rejects (`SyntaxError`) |

**Cancelling with `AbortController`**
```js
const controller = new AbortController();

fetch("/api/slow", { signal: controller.signal })
  .then(res => res.json())
  .catch(err => {
    if (err.name === "AbortError") console.log("cancelled");
    else console.error(err);
  });

controller.abort();                      // cancel
// controller.abort(new Error("reason"));  // fetch rejects with this reason instead
```

**Timeout**
```js
try {
  const res = await fetch(url, { signal: AbortSignal.timeout(5000) });
} catch (err) {
  if (err.name === "TimeoutError") console.log("took too long");
  else if (err.name === "AbortError") console.log("cancelled");
  else throw err;
}
```

**Cancel stale requests (search-as-you-type / React `useEffect`)**
```js
let controller;
async function search(query) {
  controller?.abort();                          // cancel the previous request
  controller = new AbortController();
  try {
    const data = await getJson(`/api/search?q=${encodeURIComponent(query)}`, { signal: controller.signal });
    render(data);
  } catch (err) {
    if (err.name === "AbortError") return;      // ignore: it was replaced
    showError(err);
  }
}

// React
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal }).then(r => r.json()).then(setData).catch(() => {});
  return () => controller.abort();              // cleanup on unmount / dependency change
}, [url]);
```

**One controller, many listeners**
```js
const ac = new AbortController();
window.addEventListener("resize", onResize, { signal: ac.signal });
window.addEventListener("scroll", onScroll, { signal: ac.signal });
ac.abort();   // removes both
```

### Edge cases
- **HTTP errors don't reject:** a `404` or `500` still **resolves**. Always check `res.ok` (or `res.status`).
- **The body can be read only once:** calling `res.json()` then `res.text()` throws `TypeError: body stream already read`. Use `res.clone()` first.
- **`res.json()` throws on an empty or invalid body** (e.g. `204 No Content`, HTML error pages). Check status/`content-type`, or read `res.text()` and parse safely.
- **`AbortController` is single-use:** once aborted, `signal.aborted` stays `true`. Create a new controller for each request.
- **Abort error types:** `abort()` → `AbortError`; `AbortSignal.timeout(ms)` → `TimeoutError`; `abort(reason)` → rejects with your `reason`. Check `err.name`.
- **Aborting after the response arrives** also cancels reading the body (`res.json()` rejects).
- **Abort doesn't stop the server's work,** only the client's wait. The request may still complete on the server.
- **Aborted requests aren't "errors":** don't show an error UI for `AbortError`.
- **Race conditions:** without cancelling (or ignoring) old responses, a slow earlier request can overwrite a newer result. Abort the old one or compare a request id.
- **CORS:** cross-origin requests need server headers. Blocked requests reject with a generic `TypeError: Failed to fetch` (details only in DevTools). `mode: "no-cors"` gives an unreadable *opaque* response.
- **Cookies:** `credentials` defaults to `"same-origin"`. Use `"include"` for cross-origin cookies.
- **`Content-Type` for `FormData`:** don't set it manually. The browser adds the multipart boundary.
- **Query strings:** build with `new URLSearchParams({ q })` or `encodeURIComponent`.
- **Retries:** `fetch` doesn't retry. Retrying non-idempotent requests (`POST`) can duplicate actions.
- **`AbortSignal.timeout` / `AbortSignal.any`** are newer. Check browser support or use `setTimeout(() => controller.abort(), ms)` with cleanup.
- **Large downloads:** use `res.body` (a `ReadableStream`) for progress or streaming; `res.json()` buffers everything.
- **Node:** `fetch` is built in from Node 18, with the same semantics.

### Use cases
- Loading data for pages/components, form submissions, file uploads, search suggestions.
- Cancelling outdated requests (typing, route changes, component unmount).
- Timeouts and robust API clients (error handling, retry, auth headers).

---

## 8. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### Callbacks

> [!question]- What is a callback? Sync vs async callbacks?
> A function passed to another function to be called later. Synchronous callbacks run during the call (`arr.map(fn)`); asynchronous callbacks run after something completes (`setTimeout`, events, I/O).

> [!question]- What is callback hell and how do you avoid it?
> Deeply nested callbacks for sequential async steps, making code hard to read and error handling repetitive. Avoid with promises/async-await, named functions, and splitting logic into smaller functions.

> [!question]- What is inversion of control with callbacks?
> You hand your continuation to third-party code, trusting it to call it correctly (once, with the right args, at the right time). Promises fix this because they settle once and you control the `then`.

> [!question]- Why doesn't `try/catch` catch errors in a `setTimeout` callback?
> The callback runs later, from the event loop, after the `try` block has already finished. Handle errors inside the callback (or use promises with `.catch`/`await`).

> [!question]- What is an error-first callback?
> A Node convention where the first argument is the error (or `null`) and the second is the result: `(err, data) => {}`.

### Promises

> [!question]- What is a promise? What are its states?
> An object representing a future value. States: pending, fulfilled, rejected. Once settled (fulfilled/rejected) it never changes.

> [!question]- What does `.then()` return?
> A **new promise**. Its result depends on the callback: a returned value fulfills it, a returned promise is adopted, a thrown error rejects it.

> [!question]- How does promise chaining work?
> Each `.then` returns a new promise, so you can chain steps. Return values/promises flow to the next `.then`; errors skip to the nearest `.catch`.

> [!question]- What's the difference between `.then(a, b)` and `.then(a).catch(b)`?
> In `.then(a, b)`, `b` handles rejections from the original promise only, not errors thrown in `a`. `.catch(b)` after `.then(a)` handles both.

> [!question]- What does `finally` do?
> Runs a callback whether the promise fulfills or rejects, with no arguments, and passes the original value/error through (its return value is ignored unless it throws).

> [!question]- Does the promise executor run synchronously?
> Yes, immediately. Only `.then/.catch/.finally` callbacks are asynchronous (microtasks).

> [!question]- What happens if you forget to `return` inside a `.then`?
> The next `.then` gets `undefined` and doesn't wait for the inner async work, so the chain continues too early.

> [!question]- What is an unhandled promise rejection?
> A rejection with no handler. Browsers log it and fire `unhandledrejection`; modern Node terminates the process by default. Always add `.catch` or use `try/catch` with `await`.

> [!question]- Can you cancel a promise?
> Not natively. Use `AbortController` (for `fetch` and other supporting APIs), or ignore the result with a flag.

> [!question]- How do you convert a callback-based function into a promise?
> Wrap it in `new Promise`, calling `resolve(data)` or `reject(err)` from the callback (promisify). See coding practice.

### Combinators

> [!question]- `Promise.all` vs `Promise.allSettled`?
> `all` rejects as soon as any promise rejects (fail-fast) and resolves to an array of values. `allSettled` waits for all and never rejects, giving `{status, value|reason}` for each.

> [!question]- `Promise.race` vs `Promise.any`?
> `race` settles with the first promise to settle (success or failure). `any` fulfills with the first **fulfilled** promise and rejects with `AggregateError` only if all reject.

> [!question]- What happens to the other promises when `Promise.all` rejects?
> They keep running; their results are ignored. Promises can't be cancelled.

> [!question]- What do the combinators return for an empty array?
> `all([])` → `[]`, `allSettled([])` → `[]`, `race([])` → pending forever, `any([])` → rejects with `AggregateError`.

> [!question]- How would you implement a request timeout?
> `Promise.race([request, timeoutPromise])`, ideally with `AbortController`/`AbortSignal.timeout` so the request is actually cancelled.

> [!question]- How do you limit concurrency (e.g. 5 requests at a time)?
> Start N worker loops that each take the next task until none remain (promise pool), instead of firing everything with `Promise.all`.

### async/await

> [!question]- What does an `async` function return?
> Always a promise. A returned value fulfills it; a thrown error rejects it.

> [!question]- What does `await` do?
> Pauses the surrounding async function until the promise settles, then returns the fulfilled value (or throws the rejection). It doesn't block the thread; other code keeps running.

> [!question]- How do you handle errors with async/await?
> `try/catch/finally` around the awaits, or `.catch()` on the returned promise. `await Promise.all` throws the first rejection.

> [!question]- Sequential vs parallel awaits?
> Consecutive `await`s run one after another. For independent work, start all promises first and `await Promise.all([...])`.

> [!question]- Why doesn't `forEach` work with async/await?
> `forEach` ignores returned promises, so it doesn't wait. Use `for...of` with `await` (sequential) or `await Promise.all(arr.map(async ...))` (parallel).

> [!question]- `return promise` vs `return await promise` inside `try/catch`?
> Without `await`, a rejection happens after the function has returned, so the local `catch` doesn't run. With `await`, the rejection is thrown inside the `try` and is caught.

> [!question]- Is `async/await` just syntactic sugar?
> Yes, for promises. An `await` is roughly `.then(...)` for the rest of the function, with the continuation scheduled as a microtask.

> [!question]- Can you use `await` at the top level?
> In ES modules, yes (top-level await). In classic scripts and regular functions, no.

> [!question]- What is a "floating promise"?
> A promise that's neither awaited nor handled. Its errors become unhandled rejections and its timing is unpredictable.

### Event loop & timers

> [!question]- Explain the event loop.
> JS has one call stack. Async APIs run outside it and queue callbacks. When the stack is empty, the loop runs **all** microtasks, may render, then takes one macrotask, and repeats.

> [!question]- Microtasks vs macrotasks?
> Microtasks: promise reactions, `await` continuations, `queueMicrotask`, `MutationObserver`; run right after the current task, all of them. Macrotasks: `setTimeout`, `setInterval`, events, I/O; one per loop turn.

> [!question]- Why does a promise `.then` run before `setTimeout(fn, 0)`?
> Microtasks are drained after the current task, before the next macrotask (the timer callback).

> [!question]- Can microtasks block the UI?
> Yes. If microtasks keep queueing more microtasks, the loop never reaches rendering or the next macrotask (starvation).

> [!question]- Is JavaScript asynchronous or multi-threaded?
> The language runs on a single thread; asynchrony comes from the host environment (Web APIs/Node) plus the event loop. Real parallelism is possible with Web Workers/worker threads.

> [!question]- Is `setTimeout(fn, 0)` immediate?
> No. It runs after the current code, all microtasks, and possibly a render, with a minimum delay (and ≥ 4 ms clamping when nested).

> [!question]- `setInterval` vs recursive `setTimeout`?
> `setInterval` fires on a fixed schedule regardless of how long the callback takes (and ignores async completion). Recursive `setTimeout` schedules the next run after the current one finishes, so runs never overlap.

> [!question]- Where does `requestAnimationFrame` fit?
> It runs before the next repaint, after microtasks and between tasks; it pauses in background tabs. Best for animations.

> [!question]- How do you keep the UI responsive during a heavy computation?
> Break the work into chunks scheduled with `setTimeout`/`requestIdleCallback`, move it to a Web Worker, or reduce/optimize the work.

### fetch & AbortController

> [!question]- Does `fetch` reject on a 404 or 500?
> No. It only rejects on network errors, CORS failures, or abort. Check `res.ok`/`res.status` and throw yourself.

> [!question]- How do you send JSON with `fetch`?
> `method: "POST"`, header `"Content-Type": "application/json"`, and `body: JSON.stringify(data)`. Read the reply with `await res.json()`.

> [!question]- How do you cancel a `fetch` request?
> Pass `signal: controller.signal` and call `controller.abort()`. The promise rejects with an `AbortError`.

> [!question]- How do you add a timeout to `fetch`?
> `fetch(url, { signal: AbortSignal.timeout(ms) })` (rejects with `TimeoutError`), or an `AbortController` aborted by `setTimeout` (clear the timer when done).

> [!question]- How do you avoid race conditions in search-as-you-type?
> Abort the previous request before sending a new one (or ignore responses that aren't for the latest query), and treat `AbortError` as a non-error.

> [!question]- Why can't you call `res.json()` twice?
> The body is a stream that can be consumed once. Use `res.clone()` before reading if you need it twice.

> [!question]- What does "Failed to fetch" usually mean?
> A network-level failure: offline, DNS, server down, or a CORS block. The details are in the DevTools Network/Console tabs, not the error message.

> [!question]- Is an aborted request an error?
> Technically the promise rejects, but it's an intentional cancel; filter by `err.name === "AbortError"` and don't show an error message.

---

## 9. Predict the output

```js
// 1
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");

// 2
console.log("start");
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise1")).then(() => console.log("promise2"));
queueMicrotask(() => console.log("microtask"));
(async () => { console.log("async start"); await null; console.log("async after await"); })();
console.log("end");

// 3
new Promise((resolve) => { console.log(1); resolve(2); console.log(3); }).then(console.log);
console.log(4);

// 4
async function a() { console.log(1); await b(); console.log(2); }
async function b() { console.log(3); }
a();
console.log(4);

// 5
Promise.resolve(1)
  .then(x => x + 1)
  .then(() => { throw new Error("x"); })
  .catch(() => 10)
  .then(x => console.log(x));

// 6
Promise.resolve(5).finally(() => 10).then(console.log);

// 7
Promise.all([1, Promise.resolve(2), new Promise(r => setTimeout(r, 10, 3))]).then(console.log);

// 8
Promise.all([Promise.resolve(1), Promise.reject("err")]).catch(console.log);

// 9
Promise.allSettled([Promise.resolve(1), Promise.reject("e")]).then(console.log);

// 10
Promise.race([
  new Promise(r => setTimeout(r, 50, "slow")),
  new Promise(r => setTimeout(r, 10, "fast")),
]).then(console.log);

// 11
Promise.any([Promise.reject(1), Promise.reject(2)])
  .catch(e => console.log(e.constructor.name, e.errors));

// 12
async function f1() { try { return Promise.reject(new Error("x")); } catch { return "caught"; } }
async function f2() { try { return await Promise.reject(new Error("x")); } catch { return "caught"; } }
(async () => {
  console.log(await f1().catch(() => "f1 rejected"));
  console.log(await f2());
})();

// 13
const wait = (ms, v) => new Promise(r => setTimeout(() => r(v), ms));
[1, 2, 3].forEach(async n => { await wait(10); console.log(n); });
console.log("done");

// 14
setTimeout(() => console.log("a"), 0);
setTimeout(() => console.log("b"), 0);
Promise.resolve().then(() => console.log("c"));

// 15 (the server returns 404 for /missing)
fetch("/missing")
  .then(r => console.log(r.ok, r.status))
  .catch(() => console.log("rejected"));

// 16
const ac = new AbortController();
ac.abort();
fetch("/api/data", { signal: ac.signal }).catch(e => console.log(e.name));

// 17
Promise.resolve().then(() => console.log("a"));
(async () => { await 1; console.log("b"); })();
Promise.resolve().then(() => console.log("c"));

// 18
Promise.resolve(1).then(x => Promise.resolve(x + 1)).then(console.log);

// 19
Promise.reject("e").catch(x => x).then(x => console.log("then", x));

// 20
setTimeout(() => console.log("timer"), 0);
Promise.resolve().then(() => {
  console.log("micro 1");
  Promise.resolve().then(() => console.log("micro 2"));
});
console.log("sync");
```

<details>
<summary>Answers</summary>

1. `1 4 3 2`
2. `start, async start, end, promise1, microtask, async after await, promise2, timeout`
3. `1 3 4 2` (the executor is synchronous; `.then` is a microtask)
4. `1 3 4 2` (an async function runs synchronously up to its first `await`)
5. `10` (`catch` recovers, so the chain continues fulfilled)
6. `5` (`finally`'s return value is ignored)
7. `[1, 2, 3]` (non-promises are allowed; order is preserved)
8. `"err"`
9. `[{ status: "fulfilled", value: 1 }, { status: "rejected", reason: "e" }]`
10. `"fast"`
11. `AggregateError [1, 2]`
12. `"f1 rejected"` then `"caught"` (without `await`, `f1`'s `catch` never runs)
13. `done`, then `1 2 3` after about 10 ms (`forEach` doesn't wait)
14. `c a b` (the microtask first, then timers in order)
15. `false 404` (`fetch` resolves on HTTP errors, so `catch` doesn't run)
16. `"AbortError"`
17. `a b c` (microtasks run in the order they were queued)
18. `2` (a returned promise is adopted)
19. `then e` (`catch` returned a value, so the chain is fulfilled)
20. `sync, micro 1, micro 2, timer` (all microtasks, including newly queued ones, run before the timer)

</details>

---

## 10. Coding practice

**1. `sleep` and a delayed result**
```js
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
const delay = (value, ms) => sleep(ms).then(() => value);
```

**2. Promisify a callback API**
```js
const promisify = fn => (...args) =>
  new Promise((resolve, reject) =>
    fn(...args, (err, data) => (err ? reject(err) : resolve(data))));
```

**3. `Promise.all` polyfill**
```js
Promise.myAll = function (iterable) {
  return new Promise((resolve, reject) => {
    const items = [...iterable];
    if (items.length === 0) return resolve([]);
    const results = new Array(items.length);
    let remaining = items.length;
    items.forEach((item, i) => {
      Promise.resolve(item).then(value => {
        results[i] = value;                         // keep input order
        if (--remaining === 0) resolve(results);
      }, reject);                                    // first rejection wins
    });
  });
};
```

**4. `allSettled`, `race`, `any` polyfills**
```js
Promise.myAllSettled = iterable =>
  Promise.all([...iterable].map(p =>
    Promise.resolve(p).then(
      value => ({ status: "fulfilled", value }),
      reason => ({ status: "rejected", reason }))));

Promise.myRace = iterable =>
  new Promise((resolve, reject) => {
    for (const p of iterable) Promise.resolve(p).then(resolve, reject);
  });

Promise.myAny = iterable =>
  new Promise((resolve, reject) => {
    const items = [...iterable];
    if (items.length === 0) return reject(new AggregateError([], "All promises were rejected"));
    const errors = new Array(items.length);
    let remaining = items.length;
    items.forEach((p, i) =>
      Promise.resolve(p).then(resolve, err => {
        errors[i] = err;
        if (--remaining === 0) reject(new AggregateError(errors, "All promises were rejected"));
      }));
  });
```

**5. Timeout wrapper (with timer cleanup)**
```js
function withTimeout(promise, ms) {
  let id;
  const timeout = new Promise((_, reject) => {
    id = setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms);
  });
  return Promise.race([promise, timeout]).finally(() => clearTimeout(id));
}
```

**6. Retry with exponential backoff**
```js
async function retry(fn, { retries = 3, delay = 300, factor = 2 } = {}) {
  let lastError;
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await fn(attempt);
    } catch (err) {
      lastError = err;
      if (attempt < retries) await sleep(delay * factor ** attempt);
    }
  }
  throw lastError;
}
```

**7. Limit concurrency (promise pool)**
```js
async function pool(tasks, limit) {            // tasks: array of () => Promise
  const results = new Array(tasks.length);
  let next = 0;
  async function worker() {
    while (next < tasks.length) {
      const i = next++;
      results[i] = await tasks[i]();
    }
  }
  await Promise.all(Array.from({ length: Math.min(limit, tasks.length) }, worker));
  return results;
}
```

**8. Sequential vs parallel**
```js
// Sequential
const out = [];
for (const id of ids) out.push(await getUser(id));

// Parallel
const users = await Promise.all(ids.map(getUser));

// Sequential with reduce
await tasks.reduce((chain, task) => chain.then(task), Promise.resolve());
```

**9. Robust fetch wrapper (ok check, JSON, timeout, external abort)**
```js
async function request(url, { method = "GET", body, headers, timeout = 8000, signal } = {}) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(new Error("Request timed out")), timeout);
  signal?.addEventListener("abort", () => controller.abort(signal.reason), { once: true });

  try {
    const res = await fetch(url, {
      method,
      headers: { ...(body ? { "Content-Type": "application/json" } : {}), ...headers },
      body: body ? JSON.stringify(body) : undefined,
      signal: controller.signal,
    });
    if (!res.ok) throw new Error(`HTTP ${res.status} ${res.statusText}`);
    return res.status === 204 ? null : await res.json();
  } finally {
    clearTimeout(timer);
  }
}
```

**10. Search with stale-request cancellation**
```js
let controller;
async function search(q) {
  controller?.abort();
  controller = new AbortController();
  try {
    const data = await request(`/api/search?q=${encodeURIComponent(q)}`, { signal: controller.signal });
    render(data);
  } catch (err) {
    if (err.name !== "AbortError") showError(err);
  }
}
```

**11. Polling until done (with a stop function)**
```js
function poll(fn, { interval = 1000, until = () => false } = {}) {
  let stopped = false;
  (async function tick() {
    while (!stopped) {
      const result = await fn();
      if (until(result)) return;
      await sleep(interval);
    }
  })();
  return () => { stopped = true; };
}
```

**12. Run async functions in order and collect results**
```js
async function mapSeries(items, fn) {
  const results = [];
  for (const [i, item] of items.entries()) results.push(await fn(item, i));
  return results;
}
const mapParallel = (items, fn) => Promise.all(items.map(fn));
```

**13. A simplified Promise (interview favorite, not fully spec-compliant)**
```js
class MyPromise {
  #state = "pending";
  #value;
  #handlers = [];

  constructor(executor) {
    const resolve = v => this.#settle("fulfilled", v);
    const reject = e => this.#settle("rejected", e);
    try { executor(resolve, reject); } catch (e) { reject(e); }
  }

  #settle(state, value) {
    if (this.#state !== "pending") return;                     // settles once
    if (state === "fulfilled" && value && typeof value.then === "function") {
      value.then(v => this.#settle("fulfilled", v), e => this.#settle("rejected", e));  // adopt thenables
      return;
    }
    this.#state = state;
    this.#value = value;
    queueMicrotask(() => this.#handlers.forEach(h => this.#run(h)));
  }

  #run({ onFulfilled, onRejected, resolve, reject }) {
    const fulfilled = this.#state === "fulfilled";
    const cb = fulfilled ? onFulfilled : onRejected;
    if (typeof cb !== "function") return (fulfilled ? resolve : reject)(this.#value);  // pass through
    try { resolve(cb(this.#value)); } catch (e) { reject(e); }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const handler = { onFulfilled, onRejected, resolve, reject };
      if (this.#state === "pending") this.#handlers.push(handler);
      else queueMicrotask(() => this.#run(handler));           // always async
    });
  }

  catch(onRejected) { return this.then(undefined, onRejected); }
}
```

**14. Debounced async function that ignores outdated results**
```js
function debounceAsync(fn, ms) {
  let timer, latest = 0;
  return (...args) => new Promise((resolve, reject) => {
    clearTimeout(timer);
    const id = ++latest;
    timer = setTimeout(async () => {
      try {
        const result = await fn(...args);
        if (id === latest) resolve(result);        // drop stale results
      } catch (err) { if (id === latest) reject(err); }
    }, ms);
  });
}
```

---

## 11. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Run something later | `setTimeout` (once) / `setInterval` (repeat) / recursive `setTimeout` (no overlap) |
| Wait in async code | `await sleep(ms)` |
| Wrap a callback API | `new Promise((res, rej) => cb((err, d) => err ? rej(err) : res(d)))` |
| Continue a chain with a value | Return it from `.then` (don't forget `return`) |
| Recover from an error | `.catch(() => fallback)` |
| Always run cleanup | `.finally(fn)` / `finally { }` |
| All must succeed, run in parallel | `Promise.all` |
| Need every outcome | `Promise.allSettled` |
| First result of any kind / timeout | `Promise.race` |
| First *success* | `Promise.any` (`AggregateError` if all fail) |
| Independent calls | Start together, `await Promise.all([...])` |
| Calls that depend on each other / order matters | Sequential `await` (or `for...of` + `await`) |
| Loop with async work | `for...of` + `await`, or `Promise.all(arr.map(async ...))`, never `forEach` |
| Catch rejection inside `try` when returning a promise | `return await` |
| Output-order puzzle | Sync → all microtasks → one macrotask → repeat |
| Run soon, before timers | `queueMicrotask` / `.then` |
| Run before next paint | `requestAnimationFrame` |
| Heavy work without freezing UI | Chunk it, `requestIdleCallback`, or a Web Worker |
| HTTP error handling in `fetch` | `if (!res.ok) throw new Error(res.status)` |
| Send JSON | `headers: {"Content-Type": "application/json"}` + `JSON.stringify` |
| Cancel a request | `AbortController` → `signal` → `abort()` → catch `AbortError` |
| Request timeout | `AbortSignal.timeout(ms)` → `TimeoutError` |
| Stale search results | Abort the previous request before the next |
| React cleanup | `return () => controller.abort()` / `clearTimeout` / `clearInterval` |

---

## Related notes
- [[JS 07 - ES6+ Features]]
- [[JS 09 - OOP in JS]]
- [[JS 10 - Error Handling]]
- [[JS 11 - Browser & DOM Advanced]]
- [[JS 15 - Performance]]
- [[JS 18 - Utility Implementations]]
