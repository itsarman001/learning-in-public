# JS 05 — Core Concepts (Scope, Closures, `this`)

> [!info] Scope
> Scope · Hoisting & TDZ · Closures · `this` keyword · `call` / `apply` / `bind` · Pass by value vs reference · Shallow vs deep copy
> Previous: [[JS 04 - DOM & Events Basics]] · Next: [[JS 06 - Functional Concepts]]
> This is the **most-asked interview group** for 1–2 years of experience. Expect "predict the output" questions on every section.

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| Scope | Decided by **where code is written** (lexical). Lookup goes inner → outer → global. |
| Hoisting | Declarations are registered before code runs. `var` → `undefined`, `let`/`const`/`class` → TDZ, function declarations → fully usable. |
| TDZ | The time between entering a scope and the `let`/`const` line. Access throws `ReferenceError`. |
| Closure | A function **plus the variables it was created next to**. It keeps them alive after the outer function returns. |
| Closure in loops | `var` shares one variable (`3 3 3`); `let` creates one per iteration (`0 1 2`). |
| `this` | Decided by **how a function is called**, not where it's defined. Arrow functions use the outer `this`. |
| `this` precedence | `new` > `call/apply/bind` > `obj.method()` > default (`undefined` in strict mode). |
| `bind` | Returns a **new function** with `this` (and optional args) fixed permanently. |
| Pass by value | JS is always pass-by-value. For objects, the value is a **reference** (copied). |
| Copying | Spread / `Object.assign` are **shallow**. `structuredClone` is **deep** (but not for functions/class instances). |

---

## 1. Scope

### Definition
**Scope** is the region of code where a variable is accessible. JavaScript uses **lexical (static) scope**: the scope is determined by the position of the code in the source, not by where a function is called. Each scope has a link to its parent, forming the **scope chain**.

### Summary

| Scope type | Created by | Notes |
|---|---|---|
| Global | Top level of a classic script | `var` and function declarations become `window` properties; `let`/`const` don't |
| Module | A file loaded as `type="module"` / ES module | Top-level variables are private to the module; top-level `this` is `undefined` |
| Function | A function body | Visible to `var`, `let`, `const`, params |
| Block | `{}` of `if`, `for`, `while`, `try`, plain blocks | Only `let`, `const`, `class` (not `var`) |
| Catch | `catch (err) {}` | `err` is scoped to the catch block |

**Lookup rule:** JS searches the current scope, then each parent in turn, up to global. First match wins. If none is found → `ReferenceError`.

```js
const a = "global";

function outer() {
  const b = "outer";
  function inner() {
    const c = "inner";
    console.log(a, b, c);  // all three are reachable via the scope chain
  }
  inner();
}
outer();
// console.log(b);  // ReferenceError: b is not defined (inner scopes are invisible outside)
```

**Lexical scope: location decides, not the call site**
```js
let v = "global";
function show() { console.log(v); }

function run() {
  let v = "local";
  show();            // prints "global", because show() was written in the global scope
}
run();
```

**Shadowing**
```js
let x = 1;
{
  let x = 2;        // shadows the outer x inside the block
  console.log(x);   // 2
}
console.log(x);     // 1
```

### Edge cases
- `var` ignores block scope: a `var` in an `if`/`for` block is visible in the whole function.
- **Illegal shadowing:** `let a = 1; { var a = 2; }` is a `SyntaxError` (the `var` would hoist into the same scope as `let`). The reverse (`var a; { let a; }`) is legal.
- **Implicit globals:** `count = 5` without a declaration creates a global in sloppy mode; throws `ReferenceError` in strict mode and modules. Use `"use strict"` or modules.
- `typeof undeclaredVar` → `"undefined"` (no error), but reading an undeclared variable throws.
- **Named function expression scope:** the name exists only inside the function (`const f = function inner() { inner; }`).
- **Scope ≠ `this`:** scope is about variable lookup, while `this` depends on how the function is called.
- **Each call creates a new scope**, so two calls don't share local variables.
- **Default parameter expressions** have their own scope between the outer scope and the body.
- **Global pollution:** accidental globals (and overriding built-ins like `name` or `status` on `window`) cause hard-to-find bugs.
- Top-level `let`/`const` in a classic script aren't properties of `window`, but are still shared across scripts on the page.
- Functions declared inside blocks behave differently in sloppy vs strict mode. Prefer `const fn = () => {}` inside blocks.

### Use cases
- Keeping variables private to a function or block to avoid name collisions.
- Block-scoped loop variables and temporary values.
- Modules to isolate each file's variables.

---

## 2. Hoisting and the Temporal Dead Zone (TDZ)

### Definition
**Hoisting** is the behavior where declarations are processed **before code execution**. JS runs in two steps: a **creation phase** (declarations are registered in the scope) and an **execution phase** (code runs line by line). Hoisting is not "moving code to the top". It's memory being set up first.

The **Temporal Dead Zone** is the span between entering a scope and the line where a `let`/`const`/`class` is initialized. The binding exists, but accessing it throws `ReferenceError`.

### Summary

| Declaration | Hoisted? | Value before its line |
|---|---|---|
| `var x` | Yes | `undefined` |
| `let x` / `const x` | Yes (binding), but in TDZ | **ReferenceError** |
| `function f() {}` | Yes, fully | The function (callable) |
| `var f = function () {}` / `var f = () => {}` | Only the `var` | `undefined` → `TypeError: f is not a function` when called |
| `let/const f = () => {}` | In TDZ | ReferenceError |
| `class C {}` | In TDZ | ReferenceError |
| `import` | Yes (hoisted to the top of the module) | Available |

```js
console.log(a);     // undefined
var a = 1;

console.log(b);     // ReferenceError: Cannot access 'b' before initialization
let b = 2;

sayHi();            // "hi"  (declaration hoisted with its body)
function sayHi() { console.log("hi"); }

sayBye();           // TypeError: sayBye is not a function (var hoisted as undefined)
var sayBye = function () {};
```

**What the engine "sees" for `var`**
```js
// Written:
function f() {
  console.log(x);
  var x = 5;
}
// Behaves as:
function f() {
  var x;            // declaration hoisted
  console.log(x);   // undefined
  x = 5;            // assignment stays in place
}
```

### Edge cases
- **Function vs `var` with the same name:** the function declaration wins during hoisting, but a later assignment overwrites it.
  ```js
  console.log(typeof foo);  // "function"
  var foo = 1;
  function foo() {}
  console.log(typeof foo);  // "number"
  ```
- **TDZ with `typeof`:** `typeof x` before `let x` throws; `typeof undeclared` doesn't.
- **TDZ is time-based, not position-based:** a function defined above `let x` can use `x` if it's *called* after the line runs.
  ```js
  const show = () => console.log(x);
  let x = 10;
  show();  // 10
  ```
- **TDZ in default params:** `function f(a = b, b = 1) {}` → `f()` throws.
- **Only declarations are hoisted, not initializations:** `var x = 5` hoists `var x`, not `= 5`.
- **Hoisting happens per scope:** `var` hoists to the top of its function; `let/const` to the top of their block.
- **Hoisted functions inside `if` blocks** are inconsistent across modes. Avoid.
- `var` hoisting inside loops or conditionals still yields one variable per function.
- **Why TDZ exists:** to catch use-before-define bugs that `var` silently hid.

### Use cases
- Understanding "why is this `undefined`?" bugs and ReferenceErrors.
- Function declarations let you put the main logic first and helpers below.
- Interviews: tracing output of code with mixed `var`/`let`/functions.

---

## 3. Closures

### Definition
A **closure** is a function bundled with its **lexical environment**, meaning references to the variables that were in scope where the function was created. A closure lets a function **remember and access** those variables even after the outer function has finished.

> [!note] One-liner for interviews
> "A closure is a function that retains access to its outer scope's variables even after the outer function has returned."

### Summary

**Basic example**
```js
function makeCounter() {
  let count = 0;                 // private to each makeCounter() call
  return function () {
    count++;
    return count;
  };
}

const a = makeCounter();
const b = makeCounter();
a(); a();                // 1, 2
console.log(a(), b());   // 3, 1   (separate environments)
```

**Key points**
- Closures capture **variables (by reference)**, not copies of their values.
- Every call to the outer function creates a **new environment**.
- Variables live as long as some closure references them, so they can be garbage-collected only after.

**Classic loop problem**
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3 3 3  (one shared i, which is 3 when the callbacks run)

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// 0 1 2  (a new j per iteration)

// Old fix with an IIFE:
for (var k = 0; k < 3; k++) {
  (function (k) { setTimeout(() => console.log(k), 0); })(k);
}
```

**Common patterns**

| Pattern | Example |
|---|---|
| Data privacy / encapsulation | `function Bank() { let balance = 0; return { deposit(n) { balance += n; }, get() { return balance; } }; }` |
| Function factory | `const multiplier = m => n => n * m;` |
| Memoization (cache) | `function memo(fn) { const cache = new Map(); return n => cache.has(n) ? cache.get(n) : (cache.set(n, fn(n)), cache.get(n)); }` |
| `once` | `function once(fn) { let done = false, r; return (...a) => done ? r : (done = true, r = fn(...a)); }` |
| Debounce / throttle (timer variable) | See [[JS 11 - Browser & DOM Advanced]] |
| Module pattern | IIFE returning public methods, hiding private variables |
| Event handlers with state | Handlers created in a function remember its variables |
| Partial application / currying | `const add = a => b => a + b;` |

**Frontend example: stale closure in React**
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count);       // always logs 0, captured from the first render
      // setCount(count + 1);   // always sets 1
      setCount(c => c + 1);     // fix: functional update doesn't depend on the captured value
    }, 1000);
    return () => clearInterval(id);
  }, []);                        // empty deps → closure from the first render only
}
```

### Edge cases
- **Closures capture variables, not values:**
  ```js
  let x = 1;
  const get = () => x;
  x = 2;
  get();   // 2
  ```
- **Memory leaks:** a closure holding a large object (or DOM node) keeps it alive. Common causes: forgotten event listeners, timers (`setInterval`) never cleared, caches that only grow.
- **Closures in loops with `var`** are the #1 interview trap (see above).
- **Stale closures** (React hooks, timers) use values from when the closure was created. Fix with dependency arrays, functional updates, or refs.
- **`this` isn't captured by closures:** regular inner functions get their own `this`. Arrow functions capture `this` lexically.
- **Multiple closures share one environment:**
  ```js
  function pair() {
    let n = 0;
    return { inc: () => ++n, get: () => n };  // both see the same n
  }
  ```
- **Closures can't be inspected from outside.** That's the point (privacy), but it makes testing harder. DevTools shows them in the *Closure* section of the Scope pane.
- Performance: closures have small memory overhead. Don't create thousands of unnecessary closures in hot paths.
- A function declared inside a loop creates a new closure per iteration.

### Use cases
- Private state without classes (counters, caches, flags).
- Callbacks and event handlers that need context (`onClick` with an id).
- Factories and configuration (`createLogger("auth")`).
- Memoization, `once`, debounce/throttle, currying.

---

## 4. The `this` keyword

### Definition
`this` is a special reference whose value is determined by **how a function is called** (its call-site), not where it's defined. **Arrow functions** are the exception. They take `this` from the surrounding scope (lexical `this`).

### Summary

**Binding rules (highest priority first)**

| # | Rule | Example | `this` is |
|---|---|---|---|
| 1 | **`new` binding** | `new Person()` | The newly created object |
| 2 | **Explicit binding** | `fn.call(obj)`, `fn.apply(obj)`, `fn.bind(obj)()` | `obj` |
| 3 | **Implicit binding** | `obj.method()` | `obj` (the object before the dot) |
| 4 | **Default binding** | `fn()` | `undefined` (strict mode / modules / classes) or `globalThis` (sloppy mode) |
| – | **Arrow function** | `() => this` | `this` of the enclosing scope (can't be changed) |

**Where `this` points in common places**

| Context | `this` |
|---|---|
| Global scope (browser script) | `window` |
| Global scope (ES module) | `undefined` |
| Regular function, strict mode | `undefined` |
| Regular function, sloppy mode | `globalThis` (`window`) |
| Object method | The object |
| Class method / constructor | The instance (when called as `instance.method()`) |
| DOM event handler (regular function) | `event.currentTarget` (the element) |
| DOM event handler (arrow function) | Outer `this` |
| `setTimeout(fn)` regular function | `window` (sloppy) / `undefined` (strict) |

```js
const user = {
  name: "Asha",
  greet() { return `Hi ${this.name}`; },        // implicit binding
  greetArrow: () => typeof this,                 // NOT the object: outer this
  later() {
    setTimeout(() => console.log(this.name), 0); // arrow keeps `this` = user → "Asha"
    setTimeout(function () { console.log(this?.name); }, 0); // regular: not user → undefined
  },
};
user.greet();   // "Hi Asha"
```

**The "lost `this`" problem**
```js
const greet = user.greet;
greet();                         // `this` is lost: undefined (strict) or window (sloppy)

button.addEventListener("click", user.greet);   // `this` is the button, not user

class Timer {
  constructor() { this.seconds = 0; }
  tick() { this.seconds++; }
}
const t = new Timer();
const tick = t.tick;
tick();    // TypeError: Cannot read properties of undefined (class bodies are strict)
```

**Fixes**
```js
const g1 = user.greet.bind(user);          // 1. bind
button.addEventListener("click", () => user.greet());   // 2. wrap in an arrow
class Timer2 {
  seconds = 0;
  tick = () => { this.seconds++; };        // 3. class field arrow (per-instance method)
}
```

### Edge cases
- **Arrow functions ignore `call`/`apply`/`bind` for `this`.** The first argument is ignored (extra args still pass).
- **Arrow functions as object methods** get the outer `this` (often `window`/`undefined`), not the object.
- **Nested regular functions lose `this`:**
  ```js
  const o = { n: 1, m() { return [1].map(function () { return this?.n; }); } };
  o.m();   // [undefined]  (the callback's `this` isn't `o`)
  ```
  Use an arrow function, `.map(fn, thisArg)`, or `bind`.
- **Implicit binding is lost when you detach the method,** even with grouping tricks:
  ```js
  obj.method();            // this = obj
  (obj.method)();          // this = obj (parentheses keep the reference)
  (0, obj.method)();       // this lost (comma expression returns the function value)
  (obj.method = obj.method)();  // this lost
  ```
- **Optional call keeps `this`:** `obj.method?.()` still binds `obj`.
- **`this` in a callback passed to array methods** is `undefined`/`window` unless you pass the second `thisArg` argument.
- **Primitives as `this`:** in sloppy mode `this` is boxed (`typeof this === "object"`), in strict mode it stays primitive.
- **Constructors returning objects:** if a function called with `new` returns an object, that object replaces `this`. Returning a primitive is ignored.
- **`bind` + `new`:** `new` overrides the bound `this`.
- **Event handlers:** `this` inside a regular-function handler is the element; with React class components you had to `bind` in the constructor for the same reason.
- **`this` at the top of a Node CommonJS module** is `module.exports`.
- **`globalThis`** is the portable name for the global object (window / self / global).

### Use cases
- Object methods reading their own data, constructors/classes setting instance fields.
- Borrowing methods with `call`/`apply`.
- Arrow functions for callbacks, timers, and handlers inside methods.

---

## 5. `call`, `apply`, and `bind`

### Definition
Three methods on every function (`Function.prototype`) that let you **explicitly set `this`**:
- `fn.call(thisArg, a, b)`: **invokes now**, arguments listed.
- `fn.apply(thisArg, [a, b])`: **invokes now**, arguments as an array (or array-like).
- `fn.bind(thisArg, a)`: **returns a new function** with `this` (and optional leading args) fixed. Doesn't invoke.

### Summary

| | `call` | `apply` | `bind` |
|---|---|---|---|
| Invokes immediately | Yes | Yes | **No** (returns a function) |
| Arguments | Individually | One array | Individually (partial application) |
| Typical use | Borrow a method, set `this` once | Spread an array into args (pre-ES6) | Callbacks/handlers, partial functions |

```js
function intro(greeting, punct) {
  return `${greeting}, I'm ${this.name}${punct}`;
}
const p = { name: "Asha" };

intro.call(p, "Hi", "!");          // "Hi, I'm Asha!"
intro.apply(p, ["Hi", "!"]);       // "Hi, I'm Asha!"
const bound = intro.bind(p, "Hi"); // this fixed, first arg fixed
bound("?");                        // "Hi, I'm Asha?"
```

**Method borrowing**
```js
Math.max.apply(null, [1, 5, 3]);               // 5 (modern: Math.max(...arr))
Array.prototype.slice.call(arguments);         // array-like → array (modern: Array.from)
const arrayLike = { 0: "a", 1: "b", length: 2 };
Array.prototype.map.call(arrayLike, s => s.toUpperCase());  // ["A", "B"]
```

**Partial application with `bind`**
```js
const add = (a, b) => a + b;
const add10 = add.bind(null, 10);
add10(5);   // 15
```

**Polyfills (very common interview task)**
```js
Function.prototype.myCall = function (ctx, ...args) {
  ctx = ctx == null ? globalThis : Object(ctx);   // box primitives, default to global
  const key = Symbol("fn");
  ctx[key] = this;                                // `this` here is the function being called
  const result = ctx[key](...args);               // call as a method → implicit binding
  delete ctx[key];
  return result;
};

Function.prototype.myApply = function (ctx, args = []) {
  return this.myCall(ctx, ...args);
};

Function.prototype.myBind = function (ctx, ...bound) {
  const fn = this;
  if (typeof fn !== "function") throw new TypeError("Bind must be called on a function");
  function bf(...args) {
    // `new bf()` → use the new instance as `this`; otherwise use ctx
    return fn.apply(this instanceof bf ? this : ctx, [...bound, ...args]);
  }
  if (fn.prototype) bf.prototype = Object.create(fn.prototype);
  return bf;
};
```

### Edge cases
- **`call(null)` / `call(undefined)`:** `this` becomes `globalThis` in sloppy mode, stays `null`/`undefined` in strict mode.
- **Bound functions can't be re-bound:** `fn.bind(a).bind(b)()` still uses `a`. `call`/`apply` on a bound function also can't change `this`.
- **`bind` + `new`:** the bound `this` is ignored; `new` creates a fresh object.
- **Arrow functions:** `call`/`apply`/`bind` can't change `this` (arguments still pass).
- **`bind` returns a new function** each time, so you can't remove an event listener bound inline: `el.removeEventListener("click", fn.bind(x))` doesn't work. Store the bound function.
- **Bound function name:** `fn.bind(x).name` → `"bound fn"`.
- **`apply` with huge arrays** can exceed the call stack/argument limit.
- **`apply` requires an array-like** (`null`/`undefined` allowed); other values throw `TypeError`.
- **Modern alternatives:** spread (`fn(...args)`), `Array.from`, arrow functions, class fields. You still need to know `call/apply/bind` for interviews and legacy code.

### Use cases
- Passing methods as callbacks while keeping `this` (`setTimeout(obj.method.bind(obj))`).
- Borrowing array methods for array-likes (`arguments`, `NodeList`).
- Partial application / presetting arguments.
- Writing polyfills and decorators.

---

## 6. Pass by value vs reference

### Definition
JavaScript is **always pass-by-value**. What gets copied depends on the type:
- **Primitives** (`string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`): the **value** is copied. Changing the copy never affects the original.
- **Objects** (objects, arrays, functions, dates, maps...): the **reference (address)** is copied. Both variables point to the same object. Some call this *call by sharing*.

### Summary

```js
// Primitives → independent copies
let a = 10;
let b = a;
b = 20;
console.log(a);   // 10

// Objects → shared reference
const o1 = { n: 1 };
const o2 = o1;
o2.n = 99;
console.log(o1.n);   // 99 (same object)

// Function arguments
function change(obj, num) {
  obj.x = 2;           // mutates the shared object → visible to the caller
  obj = { x: 3 };      // reassigns the local variable only → caller unaffected
  num = 100;           // primitive copy → caller unaffected
}
const original = { x: 1 };
let n = 5;
change(original, n);
console.log(original.x, n);   // 2 5
```

| Action | Primitive | Object |
|---|---|---|
| `b = a` | Copies the value | Copies the reference |
| Change `b` | `a` unaffected | `a` affected (if mutated) |
| `a === b` | Compares values | Compares references |
| Pass to a function and reassign | Caller unaffected | Caller unaffected |
| Pass to a function and mutate | n/a | **Caller sees the change** |

### Edge cases
- **Mutation vs reassignment:** `obj.x = 2` changes the shared object; `obj = {...}` just repoints the local variable.
- **Equality:** `{} === {}` and `[] === []` are `false`; two variables are `===` only if they reference the *same* object.
- **`const` doesn't prevent mutation,** only reassignment.
- **Arrays/objects in default params** are created fresh per call, so no shared-mutable-default bug (unlike some languages).
- **Strings are primitives and immutable:** `s[0] = "x"` does nothing; methods return new strings.
- **`Array(3).fill([])`** puts the same array reference into every slot.
- **State management:** mutating objects in place (`state.items.push(x)`) can skip re-renders in React because the reference doesn't change. Create new objects/arrays instead.
- **Object.freeze** blocks changes to the top-level only (shallow).
- **Sorting/reversing** mutate in place, so callers see the change. Copy first (`[...arr].sort()`).
- Comparing objects by content needs a custom deep-equal.

### Use cases
- Writing pure functions (don't mutate arguments).
- Debugging "why did my other variable change?"
- Immutable updates in React/Redux.

---

## 7. Shallow copy vs deep copy

### Definition
- **Shallow copy:** copies the top-level properties. Nested objects/arrays are still **shared** by reference.
- **Deep copy:** recursively copies everything, producing a fully independent structure.

### Summary

| Technique | Depth | What works / breaks |
|---|---|---|
| `{ ...obj }`, `[...arr]` | Shallow | Fast, idiomatic |
| `Object.assign({}, obj)` | Shallow | Mutates its first argument |
| `arr.slice()`, `Array.from(arr)`, `arr.concat()` | Shallow | |
| `JSON.parse(JSON.stringify(x))` | Deep (lossy) | Loses `undefined`, functions, symbols; Dates → strings; `Map`/`Set` → `{}`; `NaN`/`Infinity` → `null`; throws on circular refs and BigInt |
| `structuredClone(x)` | **Deep** | Handles Date, RegExp, Map, Set, typed arrays, circular refs. **Throws** on functions and DOM nodes; class instances become plain objects; symbol keys/getters/prototypes not preserved |
| Custom recursive `deepClone` | Deep | Full control, handle special types yourself |
| `_.cloneDeep` (Lodash) | Deep | Handles most cases, adds a dependency |

```js
const original = { name: "A", address: { city: "Kolkata" }, tags: ["x"] };

const shallow = { ...original };
shallow.address.city = "Delhi";
original.address.city;        // "Delhi"  (shared nested object)

const deep = structuredClone(original);
deep.address.city = "Mumbai";
original.address.city;        // still "Delhi"
```

**Manual deep clone with circular-reference support**
```js
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== "object") return value;   // primitives & functions
  if (seen.has(value)) return seen.get(value);                     // circular reference

  if (value instanceof Date) return new Date(value);
  if (value instanceof RegExp) return new RegExp(value.source, value.flags);

  if (value instanceof Map) {
    const m = new Map(); seen.set(value, m);
    value.forEach((v, k) => m.set(deepClone(k, seen), deepClone(v, seen)));
    return m;
  }
  if (value instanceof Set) {
    const s = new Set(); seen.set(value, s);
    value.forEach(v => s.add(deepClone(v, seen)));
    return s;
  }

  const out = Array.isArray(value) ? [] : Object.create(Object.getPrototypeOf(value));
  seen.set(value, out);
  for (const key of Reflect.ownKeys(value)) out[key] = deepClone(value[key], seen);
  return out;
}
```

**Immutable updates (a middle path used in React)**
```js
const next = {
  ...state,
  user: { ...state.user, name: "New" },        // copy only the path you change
  items: state.items.map(i => i.id === 2 ? { ...i, done: true } : i),
};
```

### Edge cases
- **Spread on nested data** only copies one level. A very common source of "mutated state" bugs.
- **`JSON` clone** silently drops data (no error), so Dates, `undefined`, functions, Maps, and Sets are lost or changed.
- **`structuredClone`** throws `DataCloneError` for functions and loses the prototype (class methods disappear). It's available in modern browsers and Node 17+.
- **Circular references:** `JSON.stringify` throws; `structuredClone` handles them; a naive recursive clone overflows the stack (use a `WeakMap`).
- **Property descriptors:** getters/setters are evaluated and copied as plain values by spread/`structuredClone`.
- **Deep cloning big structures is expensive.** Prefer targeted immutable updates.
- **Arrays of primitives:** `[...arr]` is effectively a full copy. Arrays of objects are not.
- **`Object.assign` and spread skip** non-enumerable and inherited properties; spread copies own enumerable symbol keys too.
- **Frozen objects:** copies aren't frozen. `Object.freeze` doesn't carry over.
- **Equality after copy:** `copy !== original` even when the contents are equal.

### Use cases
- Duplicating form state, config, or API data before editing.
- Immutable state updates (React/Redux), undo/redo history.
- Avoiding accidental shared mutation between components or tests.

---

## 8. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, plus a tiny code example if asked.

### Scope & hoisting

> [!question]- What is scope? What kinds are there?
> Scope determines where a variable is accessible. Types: global, module, function, and block (plus catch scope). `var` is function-scoped; `let`/`const`/`class` are block-scoped.

> [!question]- What is lexical scope?
> Scope is fixed by where functions and variables are written in the source code, not by where functions are called. Inner functions can access outer variables through the scope chain.

> [!question]- What is the scope chain?
> The lookup path: current scope → parent → ... → global. JS uses the first match and throws `ReferenceError` if none is found.

> [!question]- What is hoisting?
> Declarations are registered in their scope before the code executes. `var` is initialized as `undefined`; function declarations are fully available; `let`/`const`/`class` are hoisted but uninitialized (TDZ).

> [!question]- What is the Temporal Dead Zone?
> The time between entering a scope and the line where a `let`/`const`/`class` is initialized. Accessing the variable in that window throws `ReferenceError`.

> [!question]- Are `let` and `const` hoisted?
> Yes, the binding is hoisted, but it isn't initialized until its declaration runs. That's why accessing it early throws instead of returning `undefined`.

> [!question]- Why does calling a function expression before it's defined fail?
> With `var f = function () {}`, only `var f` is hoisted (as `undefined`), so `f()` throws `TypeError: f is not a function`. With `let`/`const` it throws `ReferenceError` (TDZ).

> [!question]- What is variable shadowing?
> Declaring a variable in an inner scope with the same name as one in an outer scope. The inner one hides the outer one within that scope.

> [!question]- What is an implicit global and how do you avoid it?
> Assigning to an undeclared identifier creates a global in sloppy mode. Avoid with `"use strict"`, ES modules, and always declaring variables.

### Closures

> [!question]- What is a closure?
> A function together with its lexical environment. It can access variables from the scope where it was created even after that outer function has returned.

> [!question]- Give practical uses of closures.
> Private state (counters, bank balance), function factories, memoization, `once`, debounce/throttle, currying/partial application, event handlers with context, and the module pattern.

> [!question]- Why does a `var` loop with `setTimeout` print the same number? How do you fix it?
> All callbacks close over the same `var i`, which equals the final value when they run. Use `let` (new binding per iteration), or wrap in an IIFE passing `i`.

> [!question]- Do closures capture values or variables?
> Variables (by reference). If the variable changes later, the closure sees the new value.

> [!question]- Can closures cause memory leaks?
> Yes. A closure keeps its outer variables alive. Leaks come from uncleared timers/listeners, large captured data, and caches that grow. Clean up listeners/intervals and avoid capturing big objects unnecessarily.

> [!question]- What is a stale closure? How do you fix it in React?
> A closure that holds an outdated value from an earlier render (e.g. inside `setInterval` in `useEffect` with `[]`). Fix with correct dependency arrays, functional updates (`setCount(c => c + 1)`), or refs.

> [!question]- How would you create a private variable in JavaScript?
> With a closure (variable inside a function, exposed only via returned methods), ES2022 private fields (`#x`), or a `WeakMap`. Naming conventions like `_x` aren't truly private.

> [!question]- Implement `once` and a simple `memoize`.
> See coding practice (section 9). Both use a closure to store `called`/`result` or a cache `Map`.

### `this`, `call`, `apply`, `bind`

> [!question]- How is `this` determined?
> By the call-site: `new` creates a new object; `call/apply/bind` set it explicitly; `obj.method()` uses `obj`; a plain call uses `undefined` (strict) or `globalThis` (sloppy). Arrow functions use the enclosing scope's `this`.

> [!question]- What is the order of precedence for `this` binding?
> `new` > explicit (`call/apply/bind`) > implicit (`obj.method()`) > default.

> [!question]- Why does `this` become `undefined` when I pass `obj.method` as a callback?
> Detaching a method from its object loses implicit binding; the callback is invoked as a plain function. Fix with `bind`, an arrow wrapper, or define the method as an arrow class field.

> [!question]- How do arrow functions handle `this`?
> They don't have their own `this`; they capture it lexically from the surrounding scope at definition time. It can't be changed with `call/apply/bind`.

> [!question]- When should you avoid arrow functions?
> Object methods that use `this`, constructors/prototype methods, functions needing `arguments`, and when you need dynamic `this` (e.g. DOM handlers that rely on `this === element`).

> [!question]- Difference between `call`, `apply` and `bind`?
> `call` invokes immediately with arguments listed; `apply` invokes immediately with an array of arguments; `bind` returns a new function with `this` (and optionally arguments) fixed, without invoking.

> [!question]- What happens if you `bind` a function twice?
> The first `bind` wins. A bound function's `this` can't be overridden by later `bind`, `call`, or `apply` (but `new` can still create a new object).

> [!question]- What's `this` in an event handler?
> For a regular function, `event.currentTarget` (the element). For an arrow function, the outer `this`.

> [!question]- What is `this` inside a class method that is passed as a callback?
> `undefined`, because class bodies run in strict mode and the method is called without a receiver. Bind it in the constructor or use an arrow class field.

> [!question]- Write a polyfill for `bind`.
> See section 5: store the function, return a new function that calls `fn.apply(ctx, [...boundArgs, ...newArgs])`, and handle `new` via `this instanceof bound`.

### Values, references, copying

> [!question]- Is JavaScript pass by value or pass by reference?
> Always pass by value. For objects the value is a reference, so mutations are visible to the caller but reassigning the parameter isn't.

> [!question]- What's the difference between shallow and deep copy?
> Shallow copies only the top level (nested objects are shared); deep copies recursively so nothing is shared.

> [!question]- How do you deep clone an object?
> `structuredClone(obj)` in modern environments; otherwise a recursive function (with a `WeakMap` for cycles) or Lodash `cloneDeep`. `JSON.parse(JSON.stringify(obj))` works only for plain JSON-safe data.

> [!question]- What are the problems with `JSON.parse(JSON.stringify(obj))`?
> Drops `undefined`, functions, and symbols; converts Dates to strings; `NaN`/`Infinity` become `null`; loses `Map`/`Set`; throws on circular references and BigInt.

> [!question]- What does `structuredClone` not support?
> Functions and DOM nodes (throws `DataCloneError`); class instances lose their prototype/methods (become plain objects).

> [!question]- Why is mutating state directly a problem in React?
> React detects changes by comparing references. Mutating in place keeps the same reference, so React may not re-render. Create new objects/arrays instead.

> [!question]- Why is `const arr = [1,2]; arr.push(3)` allowed?
> `const` prevents reassigning the binding, not mutating the object it points to.

---

## 9. Predict the output

```js
// 1
console.log(a);
var a = 1;
console.log(typeof b);
let b = 2;

// 2
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j), 0);

// 3
function make() { let c = 0; return () => ++c; }
const a1 = make(), b1 = make();
a1(); a1();
console.log(a1(), b1());

// 4
class C { constructor() { this.n = 1; } get() { return this.n; } }
const c = new C();
const g = c.get;
try { g(); } catch (e) { console.log(e.constructor.name); }

// 5
const o = {
  n: 1,
  viaFunction() { return [1].map(function () { return this?.n; }); },
  viaArrow() { return [1].map(() => this.n); },
};
console.log(o.viaFunction(), o.viaArrow());

// 6
function who() { return this.name; }
const bA = who.bind({ name: "A" });
const bB = bA.bind({ name: "B" });
console.log(bB(), who.call({ name: "C" }), bA.call({ name: "D" }));

// 7
function change(obj) { obj.x = 2; obj = { x: 3 }; return obj; }
const orig = { x: 1 };
const res = change(orig);
console.log(orig.x, res.x);

// 8
const s1 = { n: { v: 1 } };
const s2 = { ...s1 };
s2.n.v = 2;
console.log(s1.n.v);

// 9
console.log(JSON.parse(JSON.stringify({ d: new Date(0), u: undefined, f() {} })));

// 10
try { structuredClone({ f() {} }); } catch (e) { console.log(e.name); }

// 11
console.log(typeof foo);
var foo = 1;
function foo() {}
console.log(typeof foo);

// 12
const fns = [];
for (let k = 0; k < 3; k++) fns.push(() => k);
console.log(fns.map(f => f()));

// 13
var x = 1;
(function () { console.log(x); var x = 2; })();

// 14
let v = "global";
function show() { console.log(v); }
function run() { let v = "local"; show(); }
run();

// 15
const t = {
  n: 5,
  start() {
    setTimeout(() => console.log(this.n), 0);
    setTimeout(function () { console.log(this?.n); }, 0);
  },
};
t.start();

// 16  (strict mode)
"use strict";
const obj = { name: "O", m() { return this?.name; } };
console.log((obj.m)(), (0, obj.m)());
```

<details>
<summary>Answers</summary>

1. `undefined`, then `ReferenceError` on `typeof b` (TDZ)
2. `3 3 3` then `0 1 2` (the `var` callbacks queue first, so the order printed is `3 3 3 0 1 2`)
3. `3 1`
4. `TypeError` (class code is strict, so `this` is `undefined`)
5. `[undefined] [1]`
6. `"A" "C" "A"` (bound functions can't be re-bound)
7. `2 3`
8. `2` (shallow copy shares the nested object)
9. `{ d: "1970-01-01T00:00:00.000Z" }` (`u` and `f` are dropped; the Date becomes a string)
10. `DataCloneError`
11. `"function"` then `"number"`
12. `[0, 1, 2]` (with `var` it would be `[3, 3, 3]`)
13. `undefined` (the inner `var x` is hoisted and shadows the outer `x`)
14. `"global"` (lexical scope: `show` uses the scope where it was written)
15. `5` then `undefined` (arrow keeps `this = t`; the regular function gets its own `this`)
16. `"O" undefined`. Parentheses alone keep the receiver, but `(0, obj.m)` (a comma expression) returns the bare function, so `this` is lost (`undefined` in strict mode).

</details>

---

## 10. Coding practice

**1. Counter with private state**
```js
function createCounter(start = 0) {
  let count = start;
  return {
    inc: () => ++count,
    dec: () => --count,
    reset: () => (count = start),
    get value() { return count; },
  };
}
```

**2. `once`**
```js
function once(fn) {
  let called = false, result;
  return function (...args) {
    if (!called) { called = true; result = fn.apply(this, args); }
    return result;
  };
}
```

**3. `memoize`**
```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const value = fn.apply(this, args);
    cache.set(key, value);
    return value;
  };
}
const slowSquare = n => n * n;
const fast = memoize(slowSquare);
```

**4. Fix the loop closure (three ways)**
```js
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);                // let

for (var i = 0; i < 3; i++) (function (n) { setTimeout(() => console.log(n), 0); })(i);  // IIFE

for (var i = 0; i < 3; i++) setTimeout(console.log.bind(console, i), 0);        // bind (partial)
```

**5. Fix a lost `this`**
```js
class Counter {
  count = 0;
  inc = () => { this.count++; };      // arrow class field → always bound
}
button.addEventListener("click", new Counter().inc);
```

**6. `bind`, `call`, `apply` polyfills**: see section 5.

**7. Function that remembers how many times it's been called**
```js
function withCallCount(fn) {
  let calls = 0;
  const wrapped = (...args) => { calls++; return fn(...args); };
  wrapped.getCalls = () => calls;
  return wrapped;
}
```

**8. Currying with closures**
```js
const curry = (fn) => function curried(...args) {
  return args.length >= fn.length
    ? fn.apply(this, args)
    : (...next) => curried.apply(this, [...args, ...next]);
};
const add3 = curry((a, b, c) => a + b + c);
add3(1)(2)(3);   // 6
add3(1, 2)(3);   // 6
```

**9. Deep clone**: see section 7 for `deepClone` with a `WeakMap` for circular references.

**10. Immutable update of nested state**
```js
const state = { user: { name: "A", prefs: { theme: "light" } }, items: [{ id: 1, done: false }] };
const next = {
  ...state,
  user: { ...state.user, prefs: { ...state.user.prefs, theme: "dark" } },
  items: state.items.map(i => (i.id === 1 ? { ...i, done: true } : i)),
};
```

**11. Rate limiter using closures (allow N calls per interval)**
```js
function rateLimit(fn, limit, interval) {
  let calls = 0;
  setInterval(() => { calls = 0; }, interval);
  return (...args) => (calls < limit ? (calls++, fn(...args)) : undefined);
}
```

---

## 11. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Where can a variable be seen? | Lexical scope: the place it's written, searched inner → outer |
| Use before declaration | `var` → `undefined`; `let/const/class` → TDZ error; function declaration → works |
| Private state | Closure (or `#private` fields) |
| Closure in a loop | `let`, not `var` |
| Avoid stale closures (React) | Dependency arrays, functional `setState`, refs |
| `this` for a method call | The object before the dot |
| `this` is lost | `bind`, arrow wrapper, or arrow class field |
| `this` in callbacks inside methods | Arrow functions |
| Set `this` once and call now | `fn.call(obj, a, b)` / `fn.apply(obj, [a, b])` |
| Set `this` for later | `fn.bind(obj, a)` |
| `this` precedence | `new` > `call/apply/bind` > `obj.method()` > default |
| Compare objects | By reference; deep-equal needs custom logic |
| Prevent shared mutation | Copy before changing (`{...o}`, `[...a]`) |
| Deep copy | `structuredClone(x)` |
| Deep copy with functions/class instances | Custom clone or library |
| Update nested state immutably | Spread each level on the changed path |

---

## Related notes
- [[JS 04 - DOM & Events Basics]]
- [[JS 06 - Functional Concepts]]
- [[JS 08 - Async JavaScript]]
- [[JS 09 - OOP in JS]]
- [[JS 11 - Browser & DOM Advanced]]
