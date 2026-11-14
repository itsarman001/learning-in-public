# JS 10 — Error Handling

> [!info] Scope
> `try` / `catch` / `finally` · `throw` and the `Error` object · Custom errors · Handling async errors (callbacks, promises, `async`/`await`, `fetch`, global handlers, React error boundaries)
> Previous: [[JS 09 - OOP in JS]] · Next: [[JS 11 - Browser & DOM Advanced]]
> Async error flow builds on [[JS 08 - Async JavaScript]]. Custom errors build on classes from [[JS 09 - OOP in JS]].

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| `try/catch` | Catches **synchronous** exceptions thrown inside the `try` block (including from functions it calls). |
| `try/catch` and async | Does **not** catch errors in `setTimeout` callbacks or un-awaited promises. Use `await` or `.catch()`. |
| `finally` | Runs **always** (return, throw, break). A `return` inside `finally` overrides everything. |
| Optional catch binding | `catch { ... }` when you don't need the error variable (ES2019). |
| `throw` | Can throw anything, but **always throw `Error` objects** (they carry a stack trace). |
| Built-in types | `TypeError`, `ReferenceError`, `SyntaxError`, `RangeError`, `URIError`, `EvalError`, `AggregateError`. |
| Error properties | `name`, `message`, `stack` (non-standard but universal), `cause` (ES2022). |
| Custom errors | `class MyError extends Error`. Set `name`, add context fields, pass `{ cause }` to wrap. |
| Rethrow | Handle what you understand, **rethrow the rest**: `if (!(e instanceof X)) throw e;`. |
| Promises | A `throw` inside `.then` becomes a rejection. `.catch` handles it. Always end chains with `.catch`. |
| `async/await` | `try/catch` works with `await`. Without `await`, the rejection escapes. |
| `fetch` | Doesn't reject on `404`/`500`. Check `res.ok` and throw your own error. |
| Unhandled rejection | Browser: `unhandledrejection` event. Node 15+: crashes the process. |
| React | Error boundaries catch render/lifecycle errors only, **not** event handlers or async code. |

---

## 1. `try` / `catch` / `finally`

### Definition
`try/catch/finally` is JavaScript's structured way to handle runtime errors:
- **`try`**: code that might throw.
- **`catch (e)`**: runs if something in `try` throws; receives the thrown value.
- **`finally`**: runs afterwards **no matter what** (success, error, or early `return`).

### Summary

```js
try {
  const data = JSON.parse(text);        // may throw SyntaxError
  render(data);
} catch (err) {
  console.error("Invalid JSON:", err.message);
  showMessage("Could not read the data.");
} finally {
  hideSpinner();                         // always runs (cleanup)
}
```

**Valid forms**

| Form | Use |
|---|---|
| `try { } catch (e) { }` | Handle errors |
| `try { } catch { }` | Handle errors without using the variable (optional catch binding) |
| `try { } finally { }` | No handling, just guaranteed cleanup (the error still propagates) |
| `try { } catch (e) { } finally { }` | Handle + cleanup |

**Flow**

| What happens in `try` | `catch` | `finally` | Result |
|---|---|---|---|
| No error | skipped | runs | continues normally |
| Error thrown | runs | runs | continues after the statement (unless `catch` rethrows) |
| Error thrown, no `catch` | n/a | runs | error propagates up |
| `return` in `try` | skipped | runs **before** returning | returns the value |

**`finally` details**
```js
function a() { try { return "try"; } finally { console.log("finally runs"); } }
a();   // logs "finally runs", then returns "try"

function b() { try { return 1; } finally { return 2; } }
b();   // 2  (a return in finally overrides)

function c() {
  try { throw new Error("A"); }
  finally { return "swallowed"; }       // the error is silently discarded!
}
c();   // "swallowed"
```

**Selective handling and rethrowing**
```js
try {
  riskyOperation();
} catch (err) {
  if (err instanceof ValidationError) {
    showFieldError(err.field, err.message);   // known, expected → handle
  } else {
    throw err;                                // unknown → let it bubble up
  }
}
```

**Common real-world uses of `try/catch`**

| Operation | Why it can throw |
|---|---|
| `JSON.parse(str)` | `SyntaxError` on invalid JSON |
| `localStorage.setItem(...)` | `QuotaExceededError` / `SecurityError` (full storage, private mode) |
| `new URL(str)` | `TypeError` on an invalid URL |
| `structuredClone(obj)` | `DataCloneError` (functions, DOM nodes) |
| `decodeURIComponent(str)` | `URIError` on malformed input |
| `await somePromise` | Rejection becomes a thrown error |
| `BigInt("abc")`, `atob("%")` | `SyntaxError` / `InvalidCharacterError` |

### Edge cases
- **Only synchronous code inside `try` is covered.** A callback that runs later isn't:
  ```js
  try { setTimeout(() => { throw new Error("late"); }, 0); }
  catch (e) { /* never runs */ }
  ```
- **A promise created but not awaited isn't caught:**
  ```js
  try { Promise.reject(new Error("x")); } catch (e) { /* never runs */ }   // unhandled rejection
  ```
- **Syntax errors can't be caught in the same script** (they fail at parse time). You can catch them from `eval`, `new Function`, or `JSON.parse`.
- **`return`, `break`, `continue` in `finally` override** whatever `try`/`catch` was doing, including swallowing a thrown error. Avoid them.
- **An error thrown in `finally` replaces** the earlier one (the original error is lost).
- **An error thrown inside `catch`** propagates up (and `finally` still runs first).
- **Scope:** variables declared with `let`/`const` inside `try` aren't visible in `catch`/`finally`. Declare them above the `try`.
- **Empty `catch` blocks hide bugs.** At minimum log, or comment why it's safe to ignore.
- **`catch (e)` catches everything,** including your own programming bugs (typos causing `TypeError`). Keep `try` blocks small and rethrow what you don't expect.
- **Performance:** `try/catch` has negligible cost in modern engines. Throwing is the slower part, so don't use exceptions for ordinary control flow.
- **Loops:** `try/catch` *inside* a loop lets you continue to the next item; *outside* it stops at the first error.
- **`finally` runs even on `return`/`throw` but not when the process dies,** the tab closes, or an infinite loop/`process.exit()` happens.
- **`catch` binding is block-scoped** (`e` isn't visible after the block).

### Use cases
- Parsing untrusted input (`JSON.parse`, URLs, user files).
- Cleanup that must always happen (hide a spinner, release a lock, close a file/connection).
- Turning low-level failures into friendly messages or fallbacks.
- Converting exceptions into values at boundaries (the `[err, data]` pattern).

---

## 2. `throw` and the `Error` object

### Definition
`throw` stops normal execution and passes a value to the nearest enclosing `catch` (unwinding the call stack). The standard thing to throw is an **`Error` object**, which records a message and a **stack trace**.

### Summary

```js
function divide(a, b) {
  if (typeof a !== "number" || typeof b !== "number") throw new TypeError("Both arguments must be numbers");
  if (b === 0) throw new RangeError("Division by zero");
  return a / b;
}

try { divide(1, 0); }
catch (e) {
  console.log(e.name);      // "RangeError"
  console.log(e.message);   // "Division by zero"
  console.log(e.stack);     // "RangeError: Division by zero\n    at divide (...)"
}
```

**Built-in error types**

| Type | Typical cause |
|---|---|
| `Error` | Generic base type |
| `TypeError` | Wrong type / calling a non-function / reading a property of `null` or `undefined` / assigning to a `const` |
| `ReferenceError` | Using an undeclared variable or a `let`/`const` in the TDZ |
| `SyntaxError` | Invalid code or invalid input to `JSON.parse`, `eval`, `RegExp` |
| `RangeError` | Value out of range (`new Array(-1)`, `toFixed(101)`, infinite recursion → "Maximum call stack size exceeded") |
| `URIError` | Malformed URI in `decodeURIComponent` etc. |
| `EvalError` | Legacy (rarely thrown) |
| `AggregateError` | Several errors at once (e.g. `Promise.any` when all reject; has `.errors`) |
| `DOMException` | Browser APIs (`AbortError`, `QuotaExceededError`, `NotAllowedError`) |

**Common messages**

| Message | Meaning |
|---|---|
| `Cannot read properties of undefined (reading 'x')` | You accessed `.x` on `undefined`: a missing value upstream |
| `x is not a function` | Calling something that isn't callable (typo, `NodeList.map`, wrong import) |
| `x is not defined` | Undeclared variable / wrong scope / script order |
| `Assignment to constant variable` | Reassigning a `const` |
| `Unexpected token < in JSON at position 0` | The "JSON" was actually HTML (often an error page) |
| `Maximum call stack size exceeded` | Runaway recursion |

**Error properties**

| Property | Meaning |
|---|---|
| `name` | Error type, e.g. `"TypeError"` |
| `message` | Human-readable description |
| `stack` | Call stack as a string (non-standard but available everywhere; format varies by engine) |
| `cause` | The underlying error you wrapped (ES2022) |
| `toString()` | `"Name: message"` |

**Wrapping with `cause` (keep the original error)**
```js
try {
  await loadProfile();
} catch (err) {
  throw new Error("Could not load your profile", { cause: err });
}
// later: error.cause is the original error, with its own stack
```

**Exceptions vs returned values**

| Use an exception when | Return a value (result/`null`/`{ ok, error }`) when |
|---|---|
| Something unexpected or unrecoverable happened (bug, I/O failure) | A failure is a normal, expected outcome (validation: "email invalid") |
| The caller is several levels up and should decide | The immediate caller always handles it |
| You must not continue | The program should carry on with a fallback |

### Edge cases
- **You can throw anything** (`throw "oops"`, `throw 42`, `throw { code: 1 }`), but non-`Error` values have **no stack trace** and no `.message`. `catch (e)` may receive a string, so check `e instanceof Error`.
- **`Error()` works without `new`** (`Error("x")` still creates an error), as do the built-in subclasses. Custom classes need `new`.
- **`JSON.stringify(error)` gives `{}`** because `message` and `stack` are non-enumerable. Serialize manually: `{ name: e.name, message: e.message, stack: e.stack }`.
- **`console.log(error)`** prints the stack in browsers, but `console.log({ error })` can show only `{}` or a collapsed object depending on the tool.
- **`stack` is engine-specific** and computed when the error is *created* (not thrown). Create errors where they happen, so the trace points to the cause.
- **Re-throwing the same error keeps its original stack,** while `throw new Error(e.message)` loses it. Wrap with `{ cause: e }` instead.
- **`instanceof Error` fails across realms** (iframes, some test environments). `Object.prototype.toString.call(e) === "[object Error]"` or checking `e?.name` helps.
- **Errors that cross `postMessage` / `structuredClone`** lose custom class identity and extra properties (browsers clone `Error` types but not custom prototype data).
- **`throw` inside a function unwinds all frames** until a matching `catch`. If none exists: browser logs "Uncaught", Node prints the trace and exits.
- **Unreachable code after `throw`** never runs.
- **`TypeError` for `null.x`** is the most common production error. Fix the cause (missing data) rather than wrapping everything in `try/catch`.
- **Don't parse `error.message` strings** for logic. Messages change and are localized. Use types, `name`, `code`, or custom properties.

### Use cases
- Validating arguments and invariants (fail fast with a clear message).
- Signaling failures from low-level utilities to the layer that can handle them.
- Wrapping low-level errors with context (`cause`) as they bubble up.

---

## 3. Custom errors

### Definition
A **custom error** is a subclass of `Error` that represents a specific failure in your domain (`ValidationError`, `NotFoundError`, `HttpError`). It lets you identify errors with `instanceof` and attach structured data (status, field, code).

### Summary

```js
class AppError extends Error {
  constructor(message, options) {
    super(message, options);             // options may include { cause }
    this.name = "AppError";
  }
}

class ValidationError extends AppError {
  constructor(message, field, options) {
    super(message, options);
    this.name = "ValidationError";
    this.field = field;
  }
}

class HttpError extends AppError {
  constructor(status, statusText, body, options) {
    super(`HTTP ${status} ${statusText}`, options);
    this.name = "HttpError";
    this.status = status;
    this.body = body;
  }
}

class NotFoundError extends HttpError {
  constructor(body) { super(404, "Not Found", body); this.name = "NotFoundError"; }
}
```

**Using them**
```js
try {
  validateEmail(input);
} catch (err) {
  if (err instanceof ValidationError) {
    form.showError(err.field, err.message);
  } else if (err instanceof HttpError && err.status === 401) {
    redirectToLogin();
  } else {
    throw err;                           // unexpected → bubble up / log
  }
}

const e = new ValidationError("Email is invalid", "email");
e instanceof ValidationError;   // true
e instanceof Error;             // true
e.name;                         // "ValidationError"
String(e);                      // "ValidationError: Email is invalid"
e.stack;                        // includes the name and message
```

**Why bother?**

| Benefit | Example |
|---|---|
| Selective handling | `catch` only `ValidationError`, let others bubble |
| Structured data | `status`, `field`, `code`, `retryable` |
| Cleaner UI logic | Map error classes to messages/toasts |
| Better logging | Group by `name`, include context |
| Error hierarchies | `NotFoundError` is-an `HttpError` is-an `AppError` |

**Serializing an error (for logs and APIs)**
```js
function serializeError(err) {
  if (!(err instanceof Error)) return { message: String(err) };
  return {
    name: err.name,
    message: err.message,
    stack: err.stack,
    ...(err.cause !== undefined && { cause: serializeError(err.cause) }),
    ...Object.fromEntries(Object.entries(err)),   // custom own enumerable fields
  };
}
```

**Alternatives to exceptions: result objects**
```js
async function to(promise) {
  try { return [null, await promise]; }
  catch (err) { return [err, null]; }
}
const [err, user] = await to(fetchUser(1));
if (err) return showError(err);
```

### Edge cases
- **Always call `super(message)`** first in the constructor, and call `super()` before using `this`.
- **Set `name` explicitly.** Without it, `e.name` is just `"Error"`. `this.name = this.constructor.name` works but breaks when a minifier renames classes. A string literal is safer.
- **`name` as an own property is enumerable,** so `JSON.stringify(customError)` includes `name` and your custom fields but still omits `message` and `stack`.
- **Pass `cause`** (`super(message, { cause: err })`) so the original error and its stack aren't lost when wrapping.
- **Old transpiled code (TypeScript → ES5)** breaks `instanceof` for subclasses of `Error`. Fix with `Object.setPrototypeOf(this, new.target.prototype)` in the constructor. Modern targets don't need it.
- **Stack trace** is captured when the error is created. V8 also has `Error.captureStackTrace(this, this.constructor)` to omit the constructor frame (non-standard).
- **Deep hierarchies** get unwieldy. Two or three levels are usually enough; prefer a `code` property (`"E_NOT_FOUND"`) for fine-grained distinctions.
- **`instanceof` breaks across bundles/realms** (two copies of your error module). A `code` or `name` check is more robust at API boundaries.
- **Don't include secrets** (tokens, passwords, PII) in `message` or custom fields. Errors get logged and sent to monitoring tools.
- **User-facing vs developer-facing messages:** keep `message` technical; map error types to friendly text in the UI layer.
- **Errors sent over the network** (`postMessage`, JSON) need explicit serialization and rebuilding.
- **`AggregateError(errors, message)`** is the built-in way to report multiple failures at once (e.g. collecting every validation error).

### Use cases
- API clients (`HttpError` with status/body, `NetworkError`, `TimeoutError`).
- Form/domain validation (`ValidationError` with `field`).
- App-specific failures (`AuthError`, `PermissionError`, `NotFoundError`) mapped to UI states and redirects.
- Wrapping lower-level errors with context before they reach the UI.

---

## 4. Handling async errors

### Definition
Async code reports errors differently from synchronous code: a thrown error inside a callback, promise handler, or `async` function can't be caught by a surrounding synchronous `try/catch`. Each async style has its own channel: **error-first callbacks**, **rejected promises**, and **`await` + `try/catch`**.

### Summary

**1. Callbacks (error-first)**
```js
fs.readFile(path, "utf8", (err, data) => {
  if (err) return handle(err);       // must check manually; always return after handling
  use(data);
});
```

**2. Promises: `.catch`**
```js
fetchUser(1)
  .then(user => fetchPosts(user.id))
  .then(render)
  .catch(err => showError(err))      // catches rejections AND errors thrown in any earlier .then
  .finally(() => hideSpinner());

new Promise((resolve, reject) => {
  throw new Error("oops");           // a throw in the executor rejects the promise
}).catch(console.error);
```

**3. `async/await` + `try/catch`**
```js
async function loadDashboard() {
  try {
    const [user, stats] = await Promise.all([getUser(), getStats()]);
    render(user, stats);
  } catch (err) {
    showError(err);
  } finally {
    hideSpinner();
  }
}
```

**`fetch`: handle network errors and HTTP errors**
```js
async function getJson(url, { signal } = {}) {
  let res;
  try {
    res = await fetch(url, { signal });               // rejects on network failure / CORS / abort
  } catch (err) {
    if (err.name === "AbortError") throw err;         // cancelled on purpose: let callers ignore it
    throw new NetworkError("Network request failed", { cause: err });
  }
  if (!res.ok) {                                       // 404/500 do NOT reject
    const body = await res.text().catch(() => "");
    throw new HttpError(res.status, res.statusText, body);
  }
  return res.json();                                   // can reject on invalid JSON
}
```

**Where each kind of error comes from**

| Source | How you handle it |
|---|---|
| Sync code | `try/catch` |
| Promise rejection | `.catch()` or `await` inside `try/catch` |
| `async` function throw | Returned promise rejects → `await` in `try/catch` or `.catch()` |
| Callback API | Check the `err` argument |
| Event handler | `try/catch` inside the handler (errors there aren't returned to anyone) |
| `setTimeout` / `setInterval` callback | `try/catch` inside the callback |
| Anything left over | Global handlers (`error`, `unhandledrejection`) for logging |

**Promise-chain behavior**

| Situation | Result |
|---|---|
| `throw` inside `.then` | Next promise rejects; skips to the nearest `.catch` |
| `.catch` returns a value | Chain **recovers** (continues fulfilled) |
| `.catch` re-throws | Chain stays rejected |
| No `.catch` anywhere | **Unhandled rejection** |
| `.finally(fn)` | Runs always; passes the result/error through |

**Multiple async operations**

| Need | Use | Error behavior |
|---|---|---|
| All must succeed | `Promise.all` | Rejects with the **first** error |
| Report every outcome | `Promise.allSettled` | Never rejects; inspect each `status` |
| First success wins | `Promise.any` | Rejects with `AggregateError` if all fail |
| Timeout | `Promise.race` / `AbortSignal.timeout` | Rejects with the timeout error |

```js
const results = await Promise.allSettled(uploads.map(upload));
const failures = results.filter(r => r.status === "rejected").map(r => r.reason);
if (failures.length) showError(new AggregateError(failures, `${failures.length} uploads failed`));
```

**Global safety nets (log, don't rely on them)**
```js
// Uncaught synchronous errors (and resource load errors with capture: true)
window.addEventListener("error", (event) => {
  logError(event.error ?? event.message);
});

// Unhandled promise rejections
window.addEventListener("unhandledrejection", (event) => {
  logError(event.reason);
  // event.preventDefault();   // suppress the default console "Uncaught (in promise)" message
});

// Node.js
process.on("unhandledRejection", (reason) => { /* log */ });
process.on("uncaughtException", (err) => { /* log, then exit */ });
```

**React: error boundaries**
```jsx
class ErrorBoundary extends React.Component {
  state = { error: null };
  static getDerivedStateFromError(error) { return { error }; }           // switch to the fallback UI
  componentDidCatch(error, info) { logError(error, info.componentStack); } // report it
  render() {
    return this.state.error ? (this.props.fallback ?? <p>Something went wrong.</p>) : this.props.children;
  }
}

<ErrorBoundary fallback={<ErrorPage />}><Dashboard /></ErrorBoundary>
```

| React error source | Caught by an error boundary? |
|---|---|
| Errors while **rendering** child components | Yes |
| Lifecycle methods / constructors of children | Yes |
| **Event handlers** (`onClick`) | **No** (use `try/catch`) |
| **Async code** (`fetch`, `setTimeout`, promises) | **No** (catch and set state, or rethrow during render) |
| Server-side rendering | No |
| Errors in the boundary itself | No (a parent boundary catches it) |

**Best practices**

| Do | Avoid |
|---|---|
| Catch where you can **do something** (retry, fallback, show a message) | Empty `catch {}` blocks |
| Rethrow what you can't handle | Catching, logging, then rethrowing at every layer (duplicate logs) |
| Throw `Error` objects with context (`cause`, custom fields) | Throwing strings; matching on `message` text |
| Show friendly UI messages; log the technical details | Showing raw stack traces/messages to users |
| Always show **loading / error / empty** states for requests | Leaving a spinner running after a failure (use `finally`) |
| Treat `AbortError` as a non-error | Reporting cancelled requests as failures |
| Retry only transient failures (network, 5xx, 429) with backoff | Retrying 4xx errors or non-idempotent `POST`s blindly |
| Validate inputs early (guard clauses) | Wrapping huge blocks of code in one `try` |
| Send errors to monitoring (Sentry etc.) with context and source maps | Logging secrets/PII |

### Edge cases
- **`try { promise } catch` without `await` does nothing:** the rejection happens later and becomes unhandled.
- **`return promise` vs `return await promise` inside `try/catch`:** without `await`, a rejection escapes the `catch` (see [[JS 08 - Async JavaScript]]).
- **`forEach(async ...)` swallows errors:** the callbacks' rejections are unhandled and the outer `try/catch` never sees them. Use `for...of` with `await` or `Promise.all(arr.map(...))`.
- **Unhandled rejections are fatal in Node 15+** (process exits with an error). In browsers they only log, which makes them easy to miss.
- **`.catch(() => {})`** silences a rejection completely. Do it only when failure truly doesn't matter.
- **A `.catch` in the middle of a chain recovers:** later `.then` callbacks run with the fallback value. Rethrow if the chain should still fail.
- **`.then(a, b)` vs `.then(a).catch(b)`:** `b` in the first form doesn't handle errors thrown by `a`.
- **`Promise.all` rejection doesn't cancel** the other promises; they keep running and their later rejections are ignored (but not "unhandled").
- **Fire-and-forget async calls** (`saveAnalytics();` without `await`/`.catch`) can produce unhandled rejections. Add `.catch(...)` explicitly.
- **Event-handler errors don't propagate to callers.** In `button.onclick = async () => {...}` a failure just becomes an unhandled rejection; wrap the body in `try/catch`.
- **`fetch` doesn't reject on HTTP errors,** and `res.json()` can still fail on an empty/HTML body, so catch parse errors too.
- **Network errors look generic** (`TypeError: Failed to fetch`); the real cause (CORS, offline, DNS) is in DevTools, not in the message.
- **CORS script errors:** uncaught errors from cross-origin scripts appear as `"Script error."` with no details unless the script is served with CORS headers and `crossorigin` is set.
- **`window.onerror` doesn't catch promise rejections** (use `unhandledrejection`), and `unhandledrejection` doesn't catch sync errors.
- **React boundaries don't catch async/event errors;** store the error in state (`setError(err)`) and render the fallback.
- **Error in an error handler:** a `throw` inside `catch`/`.catch` creates a *new* rejection/exception. Make handlers simple and safe.
- **Retry loops need a limit and backoff,** or you create a retry storm. Never retry on `AbortError`.
- **Race conditions after failures:** a late error from an old request can overwrite newer UI state. Abort or ignore stale requests ([[JS 08 - Async JavaScript]]).
- **Mid-flight errors with streams/generators** need `try/finally` to release resources.

### Use cases
- Robust API calls (network + HTTP + parse errors, with retries and cancellation).
- Form submission flows (validation errors vs server errors vs unexpected failures).
- Global logging and monitoring in production.
- React apps with error boundaries around routes/widgets and per-request error state.

---

## 5. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### try / catch / finally

> [!question]- What do `try`, `catch` and `finally` do?
> `try` contains code that may throw; `catch` handles the thrown value; `finally` always runs afterwards (success, error, or early `return`), making it ideal for cleanup.

> [!question]- Does `finally` run if `try` has a `return`?
> Yes. The return value is computed, `finally` runs, then the function returns, unless `finally` itself returns or throws, which overrides it.

> [!question]- What happens if you `return` inside `finally`?
> It overrides any `return` or thrown error from `try`/`catch`, silently swallowing exceptions. Avoid it.

> [!question]- Can `try/catch` catch errors inside a `setTimeout` callback?
> No. The callback runs later, after the `try` block has finished. Put the `try/catch` inside the callback.

> [!question]- Does `try/catch` catch a rejected promise?
> Only if you `await` it inside the `try`. Without `await`, the rejection escapes and becomes unhandled.

> [!question]- What is optional catch binding?
> `catch { ... }` without `(e)` when you don't need the error value (ES2019).

> [!question]- Can you use `try` without `catch`?
> Yes, `try/finally`. The error still propagates after `finally` runs.

> [!question]- Can `try/catch` catch syntax errors?
> Not in the same script (they fail at parse time). It can catch them from `eval`, `new Function`, and `JSON.parse`.

> [!question]- Is `try/catch` slow?
> Not in modern engines. Throwing and catching an error is relatively expensive, so don't use exceptions for normal control flow.

> [!question]- What if the `catch` block itself throws?
> The new error propagates upward (after `finally` runs). The original error is lost unless you attach it as `cause`.

### throw & Error

> [!question]- What can you `throw` in JavaScript?
> Any value, but you should throw `Error` objects (or subclasses) so you get a stack trace, `name` and `message`.

> [!question]- Name the built-in error types.
> `Error`, `TypeError`, `ReferenceError`, `SyntaxError`, `RangeError`, `URIError`, `EvalError`, and `AggregateError`. Browser APIs also throw `DOMException`.

> [!question]- When do you get a `TypeError` vs a `ReferenceError`?
> `ReferenceError`: using an undeclared variable (or a `let`/`const` in the TDZ). `TypeError`: a value is the wrong type or an operation isn't allowed (`null.x`, calling a non-function, assigning to `const`).

> [!question]- What properties does an `Error` have?
> `name`, `message`, `stack` (non-standard but universal), and `cause` (ES2022).

> [!question]- What is `error.cause`?
> A way to chain errors: `new Error("high-level message", { cause: lowLevelError })` keeps the original error and its stack attached.

> [!question]- Why does `JSON.stringify(new Error("x"))` return `{}`?
> `message` and `stack` are non-enumerable own properties, so `JSON.stringify` skips them. Serialize manually (`{ name, message, stack }`).

> [!question]- How do you rethrow an error selectively?
> Check its type in `catch` and `throw err` again for anything you don't handle: `if (!(err instanceof ValidationError)) throw err;`.

> [!question]- Exceptions vs return values for errors?
> Use exceptions for unexpected failures; use return values (`null`, `{ ok, error }`, results) for expected outcomes like validation failures.

> [!question]- Why is throwing a string a bad idea?
> Strings have no stack trace and no `.message`/`.name`, so debugging and `instanceof` checks are harder. Throw `Error` objects.

### Custom errors

> [!question]- How do you create a custom error?
> `class MyError extends Error { constructor(message) { super(message); this.name = "MyError"; } }`, plus any extra fields.

> [!question]- Why create custom error classes?
> To distinguish error kinds with `instanceof`, attach structured data (status, field, code), build hierarchies, and handle each case differently.

> [!question]- Why set `this.name`?
> Otherwise `err.name` is `"Error"`, which makes logs and `toString()` less useful. Use a string literal rather than `this.constructor.name`, which breaks under minification.

> [!question]- Why avoid matching on `error.message`?
> Messages are free text that change and may be localized. Match on error classes, `name`, or a stable `code`.

> [!question]- What is `AggregateError`?
> An error wrapping multiple errors (`.errors`). `Promise.any` rejects with it when all promises fail, and you can create one to report several failures together.

> [!question]- How do you wrap an error while keeping the original?
> `throw new AppError("Context message", { cause: originalError })`.

> [!question]- Why might `instanceof` fail for custom errors?
> Old transpilation to ES5 breaks subclassing of `Error` (fix with `Object.setPrototypeOf(this, new.target.prototype)`), and duplicated bundles/iframes create different class identities. A `code` property is more robust at boundaries.

### Async error handling

> [!question]- How do you handle errors with `async/await`?
> Wrap the `await`s in `try/catch/finally`, or attach `.catch()` to the returned promise. A thrown error inside an `async` function becomes a rejection.

> [!question]- Why doesn't `try { fetch(url) } catch {}` catch a failed request?
> `fetch` returns a promise and the code doesn't `await` it, so the rejection happens after the `try` has finished. Use `await fetch(url)` inside the `try`.

> [!question]- Does `fetch` reject on a 404 or 500?
> No. It only rejects on network errors/CORS/abort. Check `res.ok` and throw your own error.

> [!question]- How do errors propagate in a promise chain?
> A rejection (or a throw inside `.then`) skips subsequent `.then` callbacks until a `.catch`. If a `.catch` returns normally, the chain recovers; re-throw to keep it failing.

> [!question]- What is an unhandled promise rejection and how do you handle it?
> A rejection with no handler. Browsers fire `unhandledrejection` (listen for it to log); Node 15+ terminates the process by default. Always end chains with `.catch` or use `try/catch` with `await`.

> [!question]- `Promise.all` vs `Promise.allSettled` for errors?
> `all` rejects with the first error (others keep running, results ignored). `allSettled` never rejects and reports every outcome, so you inspect failures yourself.

> [!question]- How do you handle errors in event handlers?
> Wrap the handler body in `try/catch` (especially `async` handlers). Errors in handlers don't propagate to any caller, and in React they aren't caught by error boundaries.

> [!question]- What do React error boundaries catch and not catch?
> They catch errors during rendering, lifecycle methods, and constructors of descendants. They don't catch event handlers, async code, SSR errors, or errors in the boundary itself.

> [!question]- `window.onerror` vs `unhandledrejection`?
> `error` events catch uncaught synchronous errors (and resource-load errors when captured). `unhandledrejection` catches promise rejections with no handler. Use both for logging.

> [!question]- How do you handle async errors inside `forEach`?
> You can't with `forEach` (it ignores promises). Use `for...of` with `await` and `try/catch`, or `Promise.all`/`allSettled` over `map`.

> [!question]- How would you design retry logic?
> Retry only transient failures (network errors, 5xx, 429) with exponential backoff and a maximum attempts limit; never retry on `AbortError` or client errors (4xx); be careful with non-idempotent requests.

> [!question]- How should you treat `AbortError`?
> As an intentional cancellation, not a failure: ignore it (don't show an error UI, don't log it as an error).

> [!question]- How do you log errors in production?
> Send them to a monitoring service (e.g. Sentry) from `error`/`unhandledrejection` handlers and error boundaries, with context (user action, route, release), using source maps, and without sensitive data.

> [!question]- What UI states should a data-fetching component handle?
> Loading, success, empty, and error (with a retry option), and it should always clear the loading state in `finally`.

---

## 6. Predict the output

```js
// 1
function f1() { try { return "try"; } finally { console.log("finally"); } }
console.log(f1());

// 2
function f2() { try { return 1; } finally { return 2; } }
console.log(f2());

// 3
function f3() {
  try { throw new Error("x"); }
  catch (e) { return "catch"; }
  finally { console.log("cleanup"); }
}
console.log(f3());

// 4
try {
  try { throw new Error("A"); }
  finally { throw new Error("B"); }
} catch (e) { console.log(e.message); }

// 5
try { setTimeout(() => { throw new Error("late"); }, 0); }
catch (e) { console.log("caught"); }
console.log("after");

// 6
async function f6() {
  try { Promise.reject(new Error("x")); }
  catch { console.log("caught"); }
  console.log("end");
}
f6();

// 7
(async () => {
  try { await Promise.reject(new Error("boom")); }
  catch (e) { console.log("caught", e.message); }
})();

// 8
async function f8() {
  try { return await Promise.resolve("ok"); }
  finally { console.log("done"); }
}
f8().then(console.log);

// 9
try { null.x; } catch (e) { console.log(e.name); }
try { undefinedVar; } catch (e) { console.log(e.name); }
try { const c = 1; c = 2; } catch (e) { console.log(e.name); }
try { JSON.parse("{bad"); } catch (e) { console.log(e.name); }
try { new Array(-1); } catch (e) { console.log(e.name); }

// 10
class ValidationError extends Error {
  constructor(m) { super(m); this.name = "ValidationError"; }
}
const ve = new ValidationError("bad");
console.log(ve instanceof ValidationError, ve instanceof Error, ve.name, String(ve));

// 11
console.log(JSON.stringify(new Error("x")));
console.log(JSON.stringify(ve));

// 12
try { throw "oops"; } catch (e) { console.log(typeof e, e instanceof Error); }

// 13
Promise.reject(new Error("x")).catch(() => "recovered").then(console.log);

// 14
Promise.all([
  Promise.resolve(1),
  Promise.reject(new Error("fail")),
  new Promise(r => setTimeout(r, 10, 3)),
]).catch(e => console.log(e.message));

// 15
const outer = new Error("outer", { cause: new Error("inner") });
console.log(outer.cause.message);

// 16
try { JSON.parse("{"); } catch { console.log("invalid"); }

// 17
try {
  try { null.f(); }
  catch (e) {
    if (!(e instanceof RangeError)) throw e;
    console.log("handled");
  }
} catch (e) { console.log("outer", e.name); }

// 18
Promise.any([Promise.reject(1), Promise.reject(2)])
  .catch(e => console.log(e.name, e.errors));

// 19
console.log(new Error("x").toString(), Error("y") instanceof Error);

// 20
(async () => {
  for (const n of [1, 2, 3]) {
    try {
      if (n === 2) throw new Error("skip");
      console.log("ok", n);
    } catch (e) { console.log("failed", n); }
  }
})();
```

<details>
<summary>Answers</summary>

1. `finally`, then `try`
2. `2` (a `return` in `finally` wins)
3. `cleanup`, then `catch`
4. `B` (the error from `finally` replaces the first)
5. `after`, and then an uncaught "late" error. `"caught"` never prints.
6. `end`. The `catch` never runs because the rejection isn't awaited, so it becomes an unhandled rejection.
7. `caught boom`
8. `done`, then `ok`
9. `TypeError`, `ReferenceError`, `TypeError`, `SyntaxError`, `RangeError`
10. `true true "ValidationError" "ValidationError: bad"`
11. `{}`, then `{"name":"ValidationError"}` (`message` and `stack` are non-enumerable; `name` was assigned as an own property)
12. `string false`
13. `recovered`
14. `fail`
15. `inner`
16. `invalid`
17. `outer TypeError` (the `TypeError` wasn't a `RangeError`, so it was rethrown)
18. `AggregateError [1, 2]`
19. `Error: x true` (`Error()` without `new` still works)
20. `ok 1`, `failed 2`, `ok 3` (`try/catch` inside the loop continues with the next item)

</details>

---

## 7. Coding practice

**1. Safe JSON parse with a fallback**
```js
function safeParse(text, fallback = null) {
  try { return JSON.parse(text); }
  catch { return fallback; }
}
```

**2. Go-style `[error, data]` wrapper**
```js
async function to(promise) {
  try { return [null, await promise]; }
  catch (err) { return [err, null]; }
}
const [err, user] = await to(fetchUser(1));
```

**3. Custom error hierarchy**
```js
class AppError extends Error {
  constructor(message, options) { super(message, options); this.name = "AppError"; }
}
class ValidationError extends AppError {
  constructor(message, field, options) { super(message, options); this.name = "ValidationError"; this.field = field; }
}
class HttpError extends AppError {
  constructor(status, statusText, body, options) {
    super(`HTTP ${status} ${statusText}`, options);
    this.name = "HttpError"; this.status = status; this.body = body;
  }
}
class NetworkError extends AppError {
  constructor(message, options) { super(message, options); this.name = "NetworkError"; }
}
```

**4. `fetch` wrapper with network + HTTP + parse handling**
```js
async function request(url, options = {}) {
  let res;
  try {
    res = await fetch(url, options);
  } catch (err) {
    if (err.name === "AbortError") throw err;
    throw new NetworkError("Network request failed", { cause: err });
  }
  if (!res.ok) {
    const body = await res.text().catch(() => "");
    throw new HttpError(res.status, res.statusText, body);
  }
  try {
    return res.status === 204 ? null : await res.json();
  } catch (err) {
    throw new AppError("Response was not valid JSON", { cause: err });
  }
}
```

**5. Retry only retryable errors, with exponential backoff**
```js
const sleep = ms => new Promise(r => setTimeout(r, ms));

const isRetryable = err =>
  err instanceof NetworkError || (err instanceof HttpError && (err.status >= 500 || err.status === 429));

async function retry(fn, { retries = 3, delay = 300 } = {}) {
  for (let attempt = 0; ; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (err.name === "AbortError" || !isRetryable(err) || attempt >= retries) throw err;
      await sleep(delay * 2 ** attempt);
    }
  }
}
```

**6. Global error logging**
```js
function setupErrorLogging(send) {
  window.addEventListener("error", e => send({ type: "error", error: serializeError(e.error ?? e.message) }));
  window.addEventListener("unhandledrejection", e => send({ type: "unhandledrejection", error: serializeError(e.reason) }));
}
```

**7. React error boundary**
```jsx
class ErrorBoundary extends React.Component {
  state = { error: null };
  static getDerivedStateFromError(error) { return { error }; }
  componentDidCatch(error, info) { logError(error, info.componentStack); }
  reset = () => this.setState({ error: null });
  render() {
    if (this.state.error) {
      return this.props.fallback?.(this.state.error, this.reset) ?? <button onClick={this.reset}>Try again</button>;
    }
    return this.props.children;
  }
}
```

**8. Handle async errors in event handlers (and surface them to an error boundary)**
```jsx
function SaveButton() {
  const [state, setState] = React.useState({ saving: false, error: null });

  async function handleClick() {
    setState({ saving: true, error: null });
    try {
      await save();
      setState({ saving: false, error: null });
    } catch (err) {
      setState({ saving: false, error: err });
    }
  }
  // To let an ErrorBoundary catch it instead:
  //   setState(() => { throw err; })
  return <button onClick={handleClick} disabled={state.saving}>{state.error ? "Retry" : "Save"}</button>;
}
```

**9. Collect all validation errors, then throw once**
```js
function validate(form) {
  const errors = [];
  if (!/^\S+@\S+\.\S+$/.test(form.email ?? "")) errors.push(new ValidationError("Invalid email", "email"));
  if ((form.password ?? "").length < 8) errors.push(new ValidationError("Password too short", "password"));
  if (errors.length) throw new AggregateError(errors, "Validation failed");
  return form;
}
try { validate(data); }
catch (e) { if (e instanceof AggregateError) e.errors.forEach(err => showFieldError(err.field, err.message)); else throw e; }
```

**10. Cleanup that always runs (`finally`)**
```js
async function submit(form) {
  setLoading(true);
  try {
    await api.save(form);
    toast("Saved!");
  } catch (err) {
    toast(err instanceof ValidationError ? err.message : "Something went wrong", { type: "error" });
    throw err;                          // let callers know it failed
  } finally {
    setLoading(false);                  // spinner always stops
  }
}
```

**11. Wrap async handlers so errors never go unhandled**
```js
const safe = (fn, onError = console.error) => async (...args) => {
  try { return await fn(...args); }
  catch (err) { onError(err); }
};
button.addEventListener("click", safe(handleSave, showErrorToast));
```

**12. Report failures from `Promise.allSettled`**
```js
async function uploadAll(files) {
  const results = await Promise.allSettled(files.map(upload));
  const succeeded = results.filter(r => r.status === "fulfilled").map(r => r.value);
  const failed = results.flatMap((r, i) => r.status === "rejected" ? [{ file: files[i].name, reason: r.reason }] : []);
  return { succeeded, failed };
}
```

**13. `assert` helper (fail fast)**
```js
function assert(condition, message = "Assertion failed") {
  if (!condition) throw new Error(message);
}
assert(Array.isArray(items), "items must be an array");
```

**14. Safe `localStorage` (private mode / quota)**
```js
const storage = {
  get(key, fallback = null) {
    try { const v = localStorage.getItem(key); return v === null ? fallback : JSON.parse(v); }
    catch { return fallback; }
  },
  set(key, value) {
    try { localStorage.setItem(key, JSON.stringify(value)); return true; }
    catch { return false; }          // QuotaExceededError, SecurityError, circular JSON...
  },
};
```

**15. Timeout that distinguishes timeout from cancel**
```js
async function getWithTimeout(url, ms = 5000) {
  try {
    const res = await fetch(url, { signal: AbortSignal.timeout(ms) });
    if (!res.ok) throw new HttpError(res.status, res.statusText);
    return await res.json();
  } catch (err) {
    if (err.name === "TimeoutError") throw new AppError(`Request timed out after ${ms}ms`, { cause: err });
    throw err;
  }
}
```

---

## 8. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Handle a sync error | `try { } catch (e) { }` |
| Always run cleanup | `finally { }` (never `return` inside it) |
| Ignore the error variable | `catch { }` |
| Throw properly | `throw new Error("message")` (never strings) |
| Keep the original error when wrapping | `new Error("context", { cause: err })` |
| Handle only known errors | `if (!(e instanceof X)) throw e;` |
| Custom error | `class X extends Error { constructor(m){ super(m); this.name = "X"; } }` |
| Several errors at once | `new AggregateError(errors, message)` |
| Serialize an error | `{ name, message, stack, cause }`, since `JSON.stringify(e)` gives `{}` |
| Catch a rejected promise | `await` inside `try`, or `.catch()` |
| `try` + un-awaited promise | Not caught. Add `await`. |
| Return a promise inside `try/catch` | `return await` so the `catch` sees the rejection |
| Chain recovery | `.catch(() => fallback)`; rethrow to keep failing |
| HTTP errors in `fetch` | `if (!res.ok) throw new HttpError(...)` |
| Several async tasks, need all | `Promise.all` (first error wins) |
| Several async tasks, need every outcome | `Promise.allSettled` |
| Event handlers / timers | `try/catch` inside the callback |
| Log leftovers | `window "error"` + `"unhandledrejection"` (Node: `process.on`) |
| React render errors | Error boundary (class with `getDerivedStateFromError` + `componentDidCatch`) |
| React async/event errors | `try/catch` + `setState` |
| Retry | Only network/5xx/429, with backoff and a limit; never on `AbortError` |
| Cancelled requests | `AbortError`: ignore, don't show an error |
| Show users | Friendly message and retry; log technical details separately |
| Avoid | Empty catch, throwing strings, matching on `message`, exceptions as control flow |

---

## Related notes
- [[JS 08 - Async JavaScript]]
- [[JS 09 - OOP in JS]]
- [[JS 11 - Browser & DOM Advanced]]
- [[JS 17 - Patterns, Testing & TypeScript]]
