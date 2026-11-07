# JS 06 — Functional Concepts

> [!info] Scope
> Higher-order functions · Pure functions · Currying · Recursion · IIFE
> Previous: [[JS 05 - Core Concepts (Scope, Closures, this)]] · Next: [[JS 07 - ES6+ Features]]
> Closures power almost everything here. If a section feels shaky, revisit the closures section of the previous note first.

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| Higher-order function (HOF) | Takes a function as an argument **and/or returns a function**. `map`, `filter`, `reduce`, `setTimeout` are all HOFs. |
| Callback | A function passed to another function to be called later. |
| Pure function | Same input → same output, and **no side effects** (no mutation, no I/O, no hidden state). |
| Immutability | Don't change inputs. Return new values (`[...arr]`, `{...obj}`). |
| Currying | `f(a, b, c)` → `f(a)(b)(c)`. One argument per call. |
| Partial application | Fix some arguments now, supply the rest later (`bind`, closures). |
| Composition | `compose(f, g)(x)` = `f(g(x))`. `pipe` runs left to right. |
| Recursion | A function calling itself. Needs a **base case** and a step that moves toward it. |
| Stack overflow | Too-deep recursion throws `RangeError`. JS engines (except Safari) don't optimize tail calls. |
| IIFE | A function expression that runs immediately: `(function () {})()` or `(() => {})()`. |

---

## 1. Higher-order functions (HOFs)

### Definition
A **higher-order function** is a function that does at least one of these:
1. **Takes** one or more functions as arguments, or
2. **Returns** a function.

This works because JavaScript functions are **first-class values**: they can be stored in variables, passed around, and returned.

### Summary

| Kind | Examples |
|---|---|
| Built-in, takes a function | `map`, `filter`, `reduce`, `forEach`, `sort`, `find`, `some`, `every`, `setTimeout`, `addEventListener`, `Promise.then` |
| Custom, takes a function | `repeat(n, fn)`, `retry(fn)`, `measure(fn)` |
| Returns a function | `multiplier(2)`, `debounce(fn, ms)`, `once(fn)`, `curry(fn)` |
| Both | Decorators/wrappers: `withLogging(fn)`, `memoize(fn)`, `compose(f, g)` |

```js
// Takes a function
function repeat(n, action) {
  for (let i = 0; i < n; i++) action(i);
}
repeat(3, i => console.log("run", i));

// Returns a function
const multiplier = factor => number => number * factor;
const double = multiplier(2);
double(5);   // 10

// Takes AND returns a function (a decorator / wrapper)
function withLogging(fn) {
  return function (...args) {
    console.log(`calling ${fn.name} with`, args);
    const result = fn.apply(this, args);
    console.log("result:", result);
    return result;
  };
}
const loggedAdd = withLogging((a, b) => a + b);
loggedAdd(2, 3);
```

**`reduce`: the HOF that can build anything**
```js
[1, 2, 3, 4].reduce((acc, n) => acc + n, 0);          // 10
["a", "b", "a"].reduce((counts, ch) => {
  counts[ch] = (counts[ch] ?? 0) + 1;
  return counts;
}, {});                                                // { a: 2, b: 1 }
```

**Composition with HOFs**
```js
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);  // right → left
const pipe    = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);       // left → right

const trim = s => s.trim();
const lower = s => s.toLowerCase();
const slug = s => s.replace(/\s+/g, "-");

pipe(trim, lower, slug)("  Hello World  ");   // "hello-world"
```

**Callback styles**

| Style | Example | Notes |
|---|---|---|
| Synchronous callback | `arr.map(fn)` | Runs immediately, during the call |
| Asynchronous callback | `setTimeout(fn, 0)`, `fetch().then(fn)` | Runs later (event loop) |
| Error-first callback | `fs.readFile(path, (err, data) => {})` | Node convention |

### Edge cases
- **Passing vs calling:** `btn.addEventListener("click", handle())` calls `handle` immediately and passes its **return value**. Pass `handle`.
- **Extra arguments:** array methods pass `(item, index, array)`. `["1","2","3"].map(parseInt)` → `[1, NaN, NaN]`. Use `.map(Number)` or wrap: `.map(s => parseInt(s, 10))`.
- **Lost `this`:** `arr.forEach(obj.method)` detaches the method. Use `bind`, an arrow wrapper, or the `thisArg` parameter ([[JS 05 - Core Concepts (Scope, Closures, this)]]).
- **Wrappers must forward `this` and all arguments:** use `function (...args) { return fn.apply(this, args); }`. An arrow wrapper loses the caller's `this`.
- **Wrappers change metadata:** the returned function has a different `name` and `length` than the original.
- **`reduce` on an empty array without an initial value** throws `TypeError`. Always pass an initial value.
- **`reduce` with objects:** returning `acc` is required each iteration. Forgetting the `return` makes `acc` `undefined` on the next step.
- **Callback timing:** an async callback runs after the surrounding code finishes, so variables it reads may have changed (see closures).
- **Errors in callbacks** inside `setTimeout` or promise handlers aren't caught by a surrounding `try/catch`.
- **Callback hell:** deeply nested callbacks. Prefer promises and `async/await` ([[JS 08 - Async JavaScript]]).
- **`sort` comparators must be consistent.** An inconsistent comparator gives implementation-defined order.
- **Performance:** chained `map().filter().map()` creates intermediate arrays. Fine for normal sizes, but consider a single `reduce` or loop for very large data.

### Use cases
- Transforming data (`map`/`filter`/`reduce`), event handling, timers, promise chains.
- Reusable behavior wrappers: logging, timing, caching, retry, debounce/throttle, auth checks.
- Middleware pipelines and React patterns (HOCs, custom hooks that accept callbacks).

---

## 2. Pure functions

### Definition
A **pure function** satisfies two rules:
1. **Deterministic:** given the same arguments, it always returns the same result.
2. **No side effects:** it doesn't change anything outside itself (no mutating arguments, no writing to outer variables, no I/O).

Related terms:
- **Side effect:** any observable change beyond returning a value (mutation, DOM update, network call, `console.log`, writing to storage, throwing to an outside handler).
- **Referential transparency:** a call can be replaced by its result without changing the program.
- **Immutability:** treating data as unchangeable; "changes" produce new copies.

### Summary

```js
// Pure
const add = (a, b) => a + b;
const addItem = (list, item) => [...list, item];                    // new array
const rename = (user, name) => ({ ...user, name });                 // new object

// Impure
let total = 0;
const addToTotal = n => (total += n);                               // changes outer state
const addItemBad = (list, item) => { list.push(item); return list; }; // mutates argument
const now = () => Date.now();                                       // non-deterministic
const roll = () => Math.random();                                   // non-deterministic
const save = data => localStorage.setItem("k", JSON.stringify(data)); // I/O side effect
```

**Common sources of impurity**

| Source | Why impure |
|---|---|
| Mutating arguments (`push`, `sort`, `splice`, `obj.x = 1`) | Caller's data changes |
| Reading/writing outer mutable variables | Result depends on hidden state |
| `Date.now()`, `Math.random()`, `new Date()` | Different result each call |
| DOM, network, storage, `console.log` | Observable effect outside the function |
| Throwing for control flow tied to external state | Depends on the environment |

**Pure vs impure, at a glance**

| | Pure | Impure |
|---|---|---|
| Output depends on | Arguments only | Arguments + hidden state |
| Changes outside world | Never | Possibly |
| Easy to test | Yes (input → output) | Needs mocks/setup |
| Safe to cache (memoize) | Yes | No |
| Safe to run in any order/parallel | Yes | Not always |

**Pure functions made practical: "functional core, imperative shell"**
```js
// pure core: decides what to do
const calcTotal = items => items.reduce((sum, i) => sum + i.price * i.qty, 0);

// impure shell: does the I/O
async function checkout(items) {
  const total = calcTotal(items);          // pure
  await fetch("/api/pay", { method: "POST", body: JSON.stringify({ total }) });  // side effect at the edge
  document.querySelector("#total").textContent = total;                         // side effect at the edge
}
```

**Immutable update patterns**
```js
const state = { user: { name: "A" }, items: [1, 2, 3] };

const next = {
  ...state,
  user: { ...state.user, name: "B" },        // update nested
  items: state.items.filter(n => n !== 2),   // remove
};
const added = [...state.items, 4];           // add
const sortedCopy = [...state.items].sort((a, b) => b - a);  // sort a copy
```

### Edge cases
- **Hidden mutation:** `arr.sort()`, `arr.reverse()`, `arr.splice()` and `arr.push()` mutate. Copy first or use `toSorted()`, `toReversed()`, `toSpliced()`.
- **Reading a constant is fine,** but reading a *mutable* outer variable makes a function impure.
- **Shallow copies share nested objects:** `{ ...obj }` followed by `copy.nested.x = 1` mutates the original. Copy each level you change ([[JS 05 - Core Concepts (Scope, Closures, this)]]).
- **`Object.freeze` is shallow** and only throws in strict mode (silent failure in sloppy mode).
- **`console.log` is technically a side effect,** but is usually tolerated for debugging.
- **Randomness and time:** pass them in as arguments (`fn(items, random)`) to keep the function pure and testable.
- **Pure ≠ idempotent:** idempotent means calling twice has the same effect as once (e.g. `PUT`), while pure means no effects and deterministic output.
- **Not everything can be pure:** UI, networking and storage are side effects. The goal is to **isolate** them.
- **React relies on purity:** components should be pure with respect to props/state, and reducers (`useReducer`, Redux) must be pure, or you'll get missed updates and double-render bugs (StrictMode intentionally double-invokes to expose impurity).
- **Memoization is only safe for pure functions.**
- **Array callbacks** (`map`, `filter`, `sort` comparators) should be pure. Mutating inside `map` is a common bug.
- **Immutability has a cost:** copying large structures repeatedly can be slow. Structural sharing libraries (Immer) help.

### Use cases
- Business logic (calculations, validation, formatting) that's easy to unit test.
- React render logic, reducers, selectors, derived data.
- Memoization and caching; parallel/async-safe code.

---

## 3. Currying and partial application

### Definition
- **Currying** transforms a function that takes multiple arguments into a chain of functions that each take **one** argument: `f(a, b, c)` → `f(a)(b)(c)`.
- **Partial application** fixes **some** arguments of a function and returns a new function that takes the remaining ones.

### Summary

| | Currying | Partial application |
|---|---|---|
| Shape | `f(a)(b)(c)`: one argument per call | `g = fix(f, a)` then `g(b, c)`: any number of remaining args |
| Result of each call | Always a function until the last | One new function |
| Tool | Manual arrows / `curry()` helper | `bind`, closures, `partial()` helper |

```js
// Manual currying
const add = a => b => c => a + b + c;
add(1)(2)(3);          // 6
const add1 = add(1);   // partially applied via currying
add1(2)(3);            // 6

// Partial application with bind
function volume(l, w, h) { return l * w * h; }
const withLength10 = volume.bind(null, 10);
withLength10(2, 3);    // 60

// Partial application with a closure
const partial = (fn, ...preset) => (...later) => fn(...preset, ...later);
const greet = (greeting, name) => `${greeting}, ${name}!`;
const hello = partial(greet, "Hello");
hello("Asha");         // "Hello, Asha!"
```

**Generic `curry` (uses `fn.length`)**
```js
const curry = fn =>
  function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...next) => curried.apply(this, [...args, ...next]);
  };

const sum3 = curry((a, b, c) => a + b + c);
sum3(1)(2)(3);     // 6
sum3(1, 2)(3);     // 6
sum3(1)(2, 3);     // 6
```

**Classic interview: `sum(1)(2)(3)()`**
```js
const sum = a => b => (b === undefined ? a : sum(a + b));
sum(1)(2)(3)();    // 6

// Version that works without the final () by overriding valueOf
function add(a) {
  const next = b => add(a + b);
  next.valueOf = () => a;
  return next;
}
add(1)(2)(3) + 0;  // 6  (coerced to a primitive)
+add(1)(2)(3);     // 6
```

**Practical curried helpers**
```js
const log = level => message => console.log(`[${level}] ${message}`);
const warn = log("WARN");
warn("Disk almost full");

const get = key => obj => obj[key];
users.map(get("name"));

const fetchFrom = baseUrl => path => fetch(`${baseUrl}${path}`).then(r => r.json());
const api = fetchFrom("https://api.example.com");
api("/users");
```

### Edge cases
- **`fn.length` limits the generic `curry`:** it ignores default and rest parameters (`(a, b = 1) => {}` has `length` 1; `(...args) => {}` has `0`). For variadic functions pass the arity explicitly: `curry(fn, 3)`.
- **Extra/fewer arguments:** a curried function called with no arguments (`add()`) returns the same function (or a result, depending on the implementation). Define the behavior clearly.
- **Argument order matters:** put the data last so partial application is useful (`map(fn)(data)`, not `map(data)(fn)`).
- **`this` in curried functions:** arrow-based currying loses `this`. The generic helper above forwards it with `apply`.
- **Each partial call allocates a closure.** Fine normally, but avoid it in tight loops.
- **`sum(1)(2)(3)` without a terminator** can't return both a function and a number. Use a terminating `()` call or the `valueOf`/`toString` trick.
- **`bind` for partial application** also fixes `this`. Pass `null` if you don't need it. A bound function's `length` is reduced by the number of bound arguments.
- **Currying vs default parameters:** defaults can't be supplied from the left like partial application.
- **Debugging:** stack traces and logs show anonymous inner functions. Name them if needed.

### Use cases
- Configuration-first helpers (`logger("auth")`, `fetchFrom(baseUrl)`, `validator(min)`).
- Event handlers that need extra data (`onClick={handle(id)}` returning a function).
- Building pipelines with `compose` / `pipe` from small single-argument functions.
- Functional libraries (Ramda, lodash/fp) are curried by default.

---

## 4. Recursion

### Definition
**Recursion** is when a function calls itself to solve a smaller version of the same problem. Every recursive function needs:
1. A **base case**: a condition where it returns without calling itself.
2. A **recursive case**: a call on a smaller or simpler input that moves toward the base case.

Each call is pushed onto the **call stack**, and results combine as the calls return.

### Summary

```js
function factorial(n) {
  if (n <= 1) return 1;          // base case
  return n * factorial(n - 1);   // recursive case
}
factorial(5);   // 120
```

**How the stack unwinds**
```
factorial(3)
 → 3 * factorial(2)
        → 2 * factorial(1)
               → 1                  (base case)
        → 2 * 1 = 2
 → 3 * 2 = 6
```

**Common recursive patterns**

| Problem | Idea |
|---|---|
| Factorial, sum to n | `f(n) = n op f(n-1)` |
| Fibonacci | `f(n) = f(n-1) + f(n-2)` (needs memoization) |
| Flatten nested arrays | If item is an array, recurse; else keep it |
| Traverse trees (DOM, menus, comments, file systems, JSON) | Handle the node, then recurse into children |
| Deep clone / deep equal | Recurse into nested objects |
| Binary search, merge sort | Divide the problem in half |
| Permutations, subsets, backtracking | Choose, recurse, undo |

```js
// Sum of digits
const sumDigits = n => (n < 10 ? n : (n % 10) + sumDigits(Math.floor(n / 10)));

// Fast exponent (O(log n))
const power = (x, n) => n === 0 ? 1 : n % 2 === 0 ? power(x * x, n / 2) : x * power(x, n - 1);

// Flatten any depth
const flatten = arr =>
  arr.reduce((acc, x) => acc.concat(Array.isArray(x) ? flatten(x) : x), []);
flatten([1, [2, [3, [4]]]]);   // [1, 2, 3, 4]

// Tree: count all nested comments
const countComments = c => 1 + (c.replies ?? []).reduce((n, r) => n + countComments(r), 0);

// Binary search (sorted array)
function search(arr, target, lo = 0, hi = arr.length - 1) {
  if (lo > hi) return -1;
  const mid = Math.floor((lo + hi) / 2);
  if (arr[mid] === target) return mid;
  return arr[mid] < target ? search(arr, target, mid + 1, hi) : search(arr, target, lo, mid - 1);
}
```

**Memoized Fibonacci**
```js
function fib(n, memo = {}) {
  if (n < 2) return n;
  return memo[n] ??= fib(n - 1, memo) + fib(n - 2, memo);
}
fib(50);   // fast; the naive version is O(2^n)
```

**Recursion vs loop**

| | Recursion | Loop |
|---|---|---|
| Readability for trees/nested data | Excellent | Needs an explicit stack |
| Memory | Uses call stack (O(depth)) | O(1) typically |
| Stack overflow risk | Yes | No |
| Speed | Function-call overhead | Usually faster |

**Converting to iteration with an explicit stack**
```js
function sumTree(root) {
  let total = 0;
  const stack = [root];
  while (stack.length) {
    const node = stack.pop();
    total += node.value;
    stack.push(...(node.children ?? []));
  }
  return total;
}
```

### Edge cases
- **Missing or wrong base case → infinite recursion** → `RangeError: Maximum call stack size exceeded`. Check every path reaches the base case (e.g. negative or non-integer input to `factorial` never hits `n === 0`).
- **Stack depth is limited** (roughly 10,000+ frames, varies by engine and frame size). A recursive sum over 100,000 items will crash. Use a loop or an explicit stack.
- **No tail-call optimization in most engines:** ES2015 specified it, but only Safari (JavaScriptCore) implements it. Tail-recursive style does not prevent overflow in Chrome/Node/Firefox. A **trampoline** can emulate it:
  ```js
  const trampoline = fn => (...args) => {
    let r = fn(...args);
    while (typeof r === "function") r = r();
    return r;
  };
  const sumTo = trampoline(function f(n, acc = 0) {
    return n === 0 ? acc : () => f(n - 1, acc + n);
  });
  sumTo(100000);   // 5000050000
  ```
- **Naive Fibonacci is exponential** (`fib(40)` takes seconds). Memoize or iterate.
- **Circular references** (objects pointing to themselves) cause infinite recursion when traversing or cloning. Track visited nodes with a `Set`/`WeakMap`.
- **Mutating shared state across calls** (a shared array in a default parameter, or an outer variable) gives bugs. Pass accumulators explicitly or return values.
- **`arr.concat` in recursion** is O(n²) for large inputs. Use `push` into a shared accumulator when performance matters.
- **Mutual recursion** (`isEven` ↔ `isOdd`) works, but is harder to reason about.
- **Debugging:** the call stack in DevTools shows each frame. Use breakpoints and watch the arguments at each level.
- **Async recursion** (`await` inside) doesn't grow the synchronous stack the same way, but still needs a base case.

### Use cases
- Nested data: comment threads, menus, folder trees, JSON of unknown depth, DOM traversal.
- Algorithms: divide and conquer (merge/quick sort, binary search), backtracking.
- Deep clone, deep equality, deep flatten, deep merge.

---

## 5. IIFE (Immediately Invoked Function Expression)

### Definition
An **IIFE** is a function expression that is **defined and executed immediately**. The wrapping parentheses turn the `function` keyword into an *expression* (otherwise it parses as a declaration, which needs a name and can't be called right away).

### Summary

```js
(function () {
  const secret = "private";
  console.log("runs once");
})();

(function () { /* ... */ }());        // alternative placement of ()

(() => { /* arrow IIFE */ })();

(async () => {                        // async IIFE: use await without top-level await
  const res = await fetch("/api/data");
  console.log(await res.json());
})();

const counter = (function () {        // module pattern: returns a public API
  let count = 0;
  return {
    inc: () => ++count,
    get: () => count,
  };
})();
counter.inc();
counter.get();   // 1
```

**With arguments**
```js
(function (win, $) {
  // win and $ are local aliases
})(window, jQuery);
```

**Why IIFEs were used**

| Purpose | Notes |
|---|---|
| Avoid polluting the global scope | Variables inside don't leak |
| Create private state | Closure over variables (module pattern) |
| Run initialization code once | Setup that needs temporary variables |
| Fix the `var` loop closure bug | `(function (i) { ... })(i)` captures each value |
| Async setup in scripts | `(async () => { ... })()` |

**Modern replacements**

| Old IIFE use | Modern alternative |
|---|---|
| Private scope | Block scope `{ let x; }` / ES modules |
| Loop closure fix | `let` in the loop |
| Module pattern | ES modules, classes with `#private` fields |
| Top-level `await` | ES modules support it directly |

### Edge cases
- **Without wrapping parentheses it's a SyntaxError:** `function () {}()` fails, since a function *statement* needs a name. Use `(function () {})()`, or `!function () {}()` / `void function () {}()`.
- **Missing semicolon before an IIFE** can merge with the previous line:
  ```js
  const a = 1
  (function () {})()      // parsed as: 1(function(){})()  → TypeError: 1 is not a function
  ```
  End statements with `;` (or start the IIFE with a leading `;`).
- **Arrow IIFEs have no own `this`/`arguments`.** A regular IIFE has `this` = `undefined` (strict) or `globalThis` (sloppy).
- **Async IIFE errors:** a rejected promise inside is unhandled unless you add `.catch(...)` or a `try/catch`.
- **Return value:** an IIFE returns what the function returns, so `const x = (() => 42)();` works.
- **Named IIFEs** (`(function init() {})()`) show up better in stack traces. The name isn't visible outside.
- **A block `{ ... }` with `let/const` replaces most IIFE use** for scoping, but can't return a value (the IIFE can).
- **Implicit globals inside an IIFE:** `var a = b = 5;` makes `a` local but `b` a global in sloppy mode.
- **An IIFE runs once.** To reuse the logic, use a regular function.
- Overusing IIFEs makes code harder to test, since inner pieces aren't accessible.

### Use cases
- Legacy scripts/libraries (jQuery plugins, UMD bundles), small inline scripts.
- Async initialization in non-module scripts and DevTools snippets.
- One-off computed values: `const config = (() => { ... return result; })();`.
- Interview classics: scope/closure/loop questions.

---

## 6. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### Higher-order functions & callbacks

> [!question]- What is a higher-order function?
> A function that takes another function as an argument, returns a function, or both. Examples: `map`, `filter`, `reduce`, `setTimeout`, and wrappers like `debounce` or `memoize`.

> [!question]- What is a callback function?
> A function passed to another function to be invoked later, either synchronously (`arr.map(fn)`) or asynchronously (`setTimeout(fn)`, event handlers, promise `.then`).

> [!question]- Why are functions "first-class citizens" in JavaScript?
> They can be assigned to variables, passed as arguments, returned from functions, and stored in data structures, just like any other value. That is what makes HOFs possible.

> [!question]- What is the difference between `map`, `filter` and `reduce`?
> `map` transforms each item (same length), `filter` keeps items that pass a test (fewer items), and `reduce` folds the array into a single value (number, object, array...) using an accumulator.

> [!question]- What happens if you call `reduce` on an empty array without an initial value?
> It throws `TypeError: Reduce of empty array with no initial value`. With an initial value it returns that value.

> [!question]- Why does `["1","2","3"].map(parseInt)` return `[1, NaN, NaN]`?
> `map` passes `(value, index, array)` and `parseInt`'s second parameter is the radix, so it parses with radix 0, 1, 2. Use `.map(Number)` or `.map(s => parseInt(s, 10))`.

> [!question]- How would you write a wrapper that logs every call to a function?
> Return a new function that logs the arguments, calls the original with `fn.apply(this, args)`, logs and returns the result.

> [!question]- What is function composition? `compose` vs `pipe`?
> Combining functions so the output of one is the input to the next. `compose(f, g)(x)` = `f(g(x))` (right to left); `pipe(f, g)(x)` = `g(f(x))` (left to right).

> [!question]- What is the difference between a callback and a promise?
> A callback is a function you hand over to be called later; nesting them leads to callback hell and awkward error handling. A promise is an object representing a future value, with chaining (`then`) and unified error handling (`catch`).

> [!question]- What does `fn.length` return, and why can it be surprising?
> The number of parameters before the first default value, excluding rest parameters. `(a, b = 1, ...c) => {}` has length 1.

### Pure functions & immutability

> [!question]- What is a pure function?
> A function that returns the same output for the same input and has no side effects (doesn't modify arguments, outer state, DOM, network, storage, etc.).

> [!question]- What is a side effect? Give examples.
> Any observable change outside a function's return value: mutating an argument or outer variable, DOM updates, `fetch`, `localStorage`, logging, reading the current time or a random number.

> [!question]- Why are pure functions preferred?
> They're predictable, easy to test, safe to memoize, safe to run in any order or in parallel, and fit React's rendering model (components, reducers, selectors).

> [!question]- Is `Math.random()` or `Date.now()` pure? What about a function that uses them?
> They're impure (non-deterministic). A function that calls them is impure unless the value is passed in as an argument.

> [!question]- How do you avoid mutating data?
> Copy before changing: spread (`[...arr]`, `{...obj}`), `map`/`filter` for updates and removals, `toSorted()`/`toReversed()`, and `structuredClone` for deep copies. Libraries like Immer help with nested updates.

> [!question]- Does `Object.freeze` make an object immutable?
> Only shallowly: top-level properties can't be added/changed/removed, but nested objects stay mutable. In sloppy mode violations fail silently; in strict mode they throw `TypeError`.

> [!question]- Is `console.log` a side effect? Is a function that logs still "pure"?
> Technically it is a side effect (I/O). In practice it's tolerated for debugging, but a strictly pure function wouldn't log.

> [!question]- Why must Redux/`useReducer` reducers be pure?
> The framework compares old and new state by reference and may call reducers multiple times (time travel, StrictMode). Mutation or randomness causes missed re-renders and inconsistent state.

### Currying & partial application

> [!question]- What is currying?
> Converting a function of several arguments into a sequence of single-argument functions: `f(a, b, c)` → `f(a)(b)(c)`.

> [!question]- Currying vs partial application?
> Currying always produces unary functions in a chain. Partial application fixes some arguments and returns a function expecting the rest (any number). `bind` and closures do partial application.

> [!question]- Why is currying useful?
> It lets you build specialized functions from general ones (`const warn = log("WARN")`), makes composition easier since each step takes one argument, and improves reuse.

> [!question]- How do you implement a generic `curry`?
> Return a function that collects arguments; once the collected count is at least `fn.length`, call `fn` with them, otherwise return another function that accumulates more. (See coding practice.)

> [!question]- Implement `sum(1)(2)(3)()`.
> `const sum = a => b => b === undefined ? a : sum(a + b);`

> [!question]- What are the limits of `fn.length`-based currying?
> It can't handle default or rest parameters (they aren't counted). Pass the arity manually for variadic functions.

> [!question]- How does `bind` relate to partial application?
> `fn.bind(thisArg, a)` returns a new function with `this` fixed and the leading argument `a` preset.

### Recursion

> [!question]- What is recursion? What are its two essential parts?
> A function calling itself on a smaller input. It needs a base case (stops the recursion) and a recursive case (moves toward the base case).

> [!question]- What is a stack overflow? How do you avoid it?
> Too many nested calls exhaust the call stack and throw `RangeError: Maximum call stack size exceeded`. Avoid with a correct base case, iteration or an explicit stack, memoization, or a trampoline.

> [!question]- Does JavaScript support tail-call optimization?
> The ES2015 spec defines it, but only Safari/JavaScriptCore implements it. Chrome, Firefox and Node don't, so deep tail recursion still overflows.

> [!question]- Why is naive recursive Fibonacci slow, and how do you fix it?
> It recomputes the same subproblems, giving O(2ⁿ) time. Memoize results (O(n)) or compute iteratively.

> [!question]- When would you choose recursion over a loop?
> For nested/tree-shaped data of unknown depth (comments, menus, file trees, JSON, DOM) and divide-and-conquer or backtracking problems. For flat iteration a loop is simpler and safer.

> [!question]- How would you flatten a deeply nested array?
> Recursion with `reduce`/`concat`, or `arr.flat(Infinity)`. Mention the iterative alternative with a stack.

> [!question]- How do you recurse over an object that may contain circular references?
> Track visited nodes in a `Set`/`WeakMap` and skip (or reuse) those already seen. Otherwise it recurses infinitely.

### IIFE

> [!question]- What is an IIFE and why was it used?
> A function expression invoked immediately, e.g. `(function () {})()`. It created a private scope to avoid global variables, hold private state (module pattern), run one-time setup, and capture loop variables.

> [!question]- Why do you need parentheses around the function in an IIFE?
> Without them `function () {}()` is parsed as a function *declaration* (which requires a name and can't be invoked directly), giving a SyntaxError. The parentheses force an expression.

> [!question]- Are IIFEs still needed today?
> Rarely. ES modules, block scope (`let`/`const`), classes with `#private`, and top-level `await` cover most uses. They remain common in legacy code and for async setup in plain scripts.

> [!question]- What is an async IIFE for?
> To use `await` where top-level `await` isn't available: `(async () => { ... })();`. Handle errors inside with `try/catch`, since nothing awaits it.

> [!question]- What bug can happen with a missing semicolon before an IIFE?
> The previous line and the IIFE merge into one expression (e.g. `const a = 1\n(function(){})()` → `1(...)`), causing a `TypeError`. End the previous statement with `;` or start the IIFE with `;`.

---

## 7. Predict the output

```js
// 1
const add = a => b => a + b;
console.log(add(1)(2), typeof add(1));

// 2
function f(a, b, c) {}
console.log(f.length, ((a, b = 1) => {}).length, ((...rest) => {}).length);

// 3
const arr = [3, 1, 2];
const sorted = arr.sort();
console.log(arr === sorted, arr);

// 4
try { [].reduce((a, b) => a + b); } catch (e) { console.log(e.name); }
console.log([].reduce((a, b) => a + b, 0));

// 5
function countdown(n) {
  if (n === 0) return;
  console.log(n);
  countdown(n - 1);
  console.log(n);
}
countdown(3);

// 6
const sum = a => b => (b === undefined ? a : sum(a + b));
console.log(sum(1)(2)(3)());

// 7  (sloppy mode)
(function () { var a = b = 5; })();
console.log(typeof a, typeof b);

// 8
const counter = (() => { let c = 0; return () => ++c; })();
counter();
console.log(counter());

// 9
"use strict";
const o = Object.freeze({ list: [1] });
o.list.push(2);
try { o.name = "x"; } catch (e) { console.log(e.name); }
console.log(o.list);

// 10
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const inc = x => x + 1, dbl = x => x * 2;
console.log(compose(inc, dbl)(5), pipe(inc, dbl)(5));

// 11
const fns = [1, 2, 3].map(n => () => n);
console.log(fns.map(fn => fn()));

// 12
const fib = n => (n < 2 ? n : fib(n - 1) + fib(n - 2));
console.log(fib(10));

// 13
function inf() { inf(); }
try { inf(); } catch (e) { console.log(e instanceof RangeError); }

// 14
const addTwo = (a, b) => a + b;
const add5 = addTwo.bind(null, 5);
console.log(add5(3), add5.length);

// 15
const x = 1
;(function () { console.log(x); })()
```

<details>
<summary>Answers</summary>

1. `3 "function"`
2. `3 1 0`
3. `true [1, 2, 3]` (`sort` mutates and returns the same array)
4. `TypeError`, then `0`
5. `3 2 1 1 2 3` (the logs after the recursive call run as the stack unwinds)
6. `6`
7. `"undefined" "number"` (`a` is local; `b` leaked as an implicit global. In strict mode this throws a `ReferenceError`)
8. `2`
9. `TypeError` (strict mode, assigning to a frozen object), then `[1, 2]` (`freeze` is shallow)
10. `11 12` (`compose` runs `dbl` then `inc`; `pipe` runs `inc` then `dbl`)
11. `[1, 2, 3]`
12. `55`
13. `true` (a `RangeError` for the call stack)
14. `8 1` (the bound function's `length` drops by the number of bound arguments)
15. `1` (the leading semicolon prevents the merge with the previous line)

</details>

---

## 8. Coding practice

**1. Polyfills: `map`, `filter`, `reduce`**
```js
Array.prototype.myMap = function (cb, thisArg) {
  const out = new Array(this.length);
  for (let i = 0; i < this.length; i++) {
    if (i in this) out[i] = cb.call(thisArg, this[i], i, this);
  }
  return out;
};

Array.prototype.myFilter = function (cb, thisArg) {
  const out = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && cb.call(thisArg, this[i], i, this)) out.push(this[i]);
  }
  return out;
};

Array.prototype.myReduce = function (cb, ...init) {
  let i = 0, acc;
  if (init.length) acc = init[0];
  else {
    while (i < this.length && !(i in this)) i++;
    if (i >= this.length) throw new TypeError("Reduce of empty array with no initial value");
    acc = this[i++];
  }
  for (; i < this.length; i++) if (i in this) acc = cb(acc, this[i], i, this);
  return acc;
};
```

**2. `compose` and `pipe`**
```js
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe    = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);
```

**3. Generic `curry`**
```js
const curry = (fn, arity = fn.length) =>
  function curried(...args) {
    return args.length >= arity
      ? fn.apply(this, args)
      : (...next) => curried.apply(this, [...args, ...next]);
  };
```

**4. `sum(1)(2)(3)()` and the `valueOf` variant**
```js
const sum = a => b => (b === undefined ? a : sum(a + b));

function add(a) {
  const next = b => add(a + b);
  next.valueOf = () => a;
  return next;
}
```

**5. `partial`**
```js
const partial = (fn, ...preset) => (...later) => fn(...preset, ...later);
```

**6. Wrapper that logs calls and keeps `this`**
```js
function withLogging(fn) {
  return function (...args) {
    console.log(`${fn.name || "anonymous"}(`, ...args, ")");
    return fn.apply(this, args);
  };
}
```

**7. `withRetry` (HOF around an async function)**
```js
const withRetry = (fn, retries = 3, delay = 500) => async (...args) => {
  let lastError;
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await fn(...args);
    } catch (err) {
      lastError = err;
      if (attempt < retries) await new Promise(r => setTimeout(r, delay));
    }
  }
  throw lastError;
};
```

**8. Factorial, Fibonacci (memoized), power**
```js
const factorial = n => (n <= 1 ? 1 : n * factorial(n - 1));

function fib(n, memo = {}) {
  if (n < 2) return n;
  return memo[n] ??= fib(n - 1, memo) + fib(n - 2, memo);
}

const power = (x, n) => n === 0 ? 1 : n % 2 === 0 ? power(x * x, n / 2) : x * power(x, n - 1);
```

**9. Recursive flatten (with optional depth)**
```js
function flatten(arr, depth = Infinity) {
  return depth < 1
    ? arr.slice()
    : arr.reduce((acc, x) =>
        acc.concat(Array.isArray(x) ? flatten(x, depth - 1) : x), []);
}
```

**10. Reverse a string / palindrome recursively**
```js
const reverse = s => (s === "" ? "" : reverse(s.slice(1)) + s[0]);
const isPalindrome = s => s.length < 2 || (s[0] === s.at(-1) && isPalindrome(s.slice(1, -1)));
```

**11. Walk a tree (comments / menu) recursively**
```js
const countNodes = node => 1 + (node.children ?? []).reduce((n, c) => n + countNodes(c), 0);

function findById(node, id) {
  if (node.id === id) return node;
  for (const child of node.children ?? []) {
    const found = findById(child, id);
    if (found) return found;
  }
  return null;
}
```

**12. Memoize (pure functions only)**
```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

**13. Module-pattern IIFE with private state**
```js
const cart = (() => {
  const items = [];
  return {
    add: item => { items.push(item); },
    count: () => items.length,
    total: () => items.reduce((sum, i) => sum + i.price, 0),
  };
})();
```

---

## 9. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Function that takes/returns a function | Higher-order function |
| Transform / filter / combine items | `map` / `filter` / `reduce` (always give `reduce` an initial value) |
| Wrap behavior (log, time, retry, cache) | HOF wrapper using `fn.apply(this, args)` |
| Combine functions | `pipe` (left→right) / `compose` (right→left) |
| Pure function checklist | Same input → same output · no mutation · no I/O · no hidden state |
| Update data without mutating | Spread, `map`, `filter`, `toSorted`, `structuredClone` |
| Keep side effects contained | Pure core, effects at the edges |
| Currying | `f(a)(b)(c)`; generic helper uses `fn.length` |
| Partial application | `fn.bind(null, a)` or `(...later) => fn(a, ...later)` |
| `sum(1)(2)(3)()` | `a => b => b === undefined ? a : sum(a + b)` |
| Recursion checklist | Base case · move toward it · watch stack depth · memoize overlaps |
| Deep or unknown-depth data | Recursion (or an explicit stack) |
| Stack overflow fix | Loop / explicit stack / trampoline (TCO isn't available in most engines) |
| IIFE syntax | `(function () {})()` · `(() => {})()` · `(async () => {})()` |
| IIFE today | Rarely needed: use modules, blocks, classes |
| Statement before an IIFE | End with `;` to avoid merging lines |

---

## Related notes
- [[JS 05 - Core Concepts (Scope, Closures, this)]]
- [[JS 07 - ES6+ Features]]
- [[JS 08 - Async JavaScript]]
- [[JS 18 - Utility Implementations]]
