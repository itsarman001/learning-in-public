# 02 — Functions

> [!info] Scope Declarations vs expressions · Arrow functions · Parameters (default / rest) · Return values · Scope basics Previous: [[JS 01 - Language Basics]] · Next: [[JS 03 - Arrays & Objects]]

---

## 0. One-page summary

| Topic          | The one thing to remember                                                                                                  |
| -------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Declaration    | `function f() {}` is fully hoisted. You can call it before it appears in the code.                                         |
| Expression     | `const f = function () {}` is **not** callable before its line (`var` → `TypeError`, `let/const` → `ReferenceError`).      |
| Arrow function | Short syntax, **no own `this`, `arguments`, or `prototype`**, can't be used with `new`.                                    |
| Default params | Used only when the argument is `undefined` (not `null`). Evaluated at call time.                                           |
| Rest params    | `...args` collects the remaining arguments into a **real array**. Must be last.                                            |
| Return         | No `return` → `undefined`. A line break right after `return` returns `undefined`.                                          |
| Arrow body     | `=> { ... }` needs an explicit `return`. `=> expr` returns implicitly. `=> ({ ... })` returns an object.                   |
| Arguments      | Primitives are copied. For objects, the reference is copied (mutation is visible, reassignment is not).                    |
| Scope          | Inner functions can read outer variables; outer code can't read inner ones. Scope is decided by **where code is written**. |

---

## 1. Function declarations vs expressions

### Definition

A **function** is a reusable block of code that takes input (parameters), runs statements, and optionally returns a value. In JavaScript, functions are **first-class**: they are values you can store in variables, pass as arguments, and return from other functions.

### Summary

| |Declaration|Expression|
|---|---|---|
|Syntax|`function add(a, b) { return a + b; }`|`const add = function (a, b) { return a + b; };`|
|Hoisting|Fully hoisted (name + body)|Only the variable is hoisted, not the function|
|Callable before definition|**Yes**|**No**|
|Name required|Yes|Optional (anonymous allowed)|
|Is a statement or a value|Statement|Value (can be passed around)|

```js
console.log(add(2, 3));   // 5  (declaration is hoisted)
function add(a, b) { return a + b; }

console.log(mul(2, 3));   // TypeError: mul is not a function  (var is hoisted as undefined)
var mul = function (a, b) { return a * b; };

console.log(div(6, 2));   // ReferenceError  (let/const in the TDZ)
const div = function (a, b) { return a / b; };
```

**Named function expression**

```js
const fact = function inner(n) { return n <= 1 ? 1 : n * inner(n - 1); };
fact(5);      // 120
typeof inner; // "undefined" — the name exists only inside the function
```

**Other common forms**

```js
// Method shorthand
const obj = { greet() { return "hi"; } };

// Callback (function passed as an argument)
[1, 2, 3].map(function (n) { return n * 2; });

// IIFE (Immediately Invoked Function Expression)
(function () { console.log("runs once"); })();
```

### Edge cases

- **Declaration vs `var` with the same name:** function declarations are hoisted first, then the `var` assignment runs.
    
    ```js
    var a = 1;function a() {}typeof a;  // "number"
    ```
    
- **Functions are objects:** they have properties like `name` and `length`, and you can attach your own properties.
    
    ```js
    function greet(a, b) {}greet.name;    // "greet"greet.length;  // 2 (declared parameters)
    ```
    
- **Function declarations inside blocks** (`if`, `for`) behave differently in strict mode vs sloppy mode. Avoid them; use a `const` function expression instead.
- **Anonymous functions** show up as `anonymous` in stack traces. Give functions names (or assign them to a named `const`) for easier debugging.
- Assigning an anonymous function to a variable gives it that name: `const f = function () {}; f.name` → `"f"`.
- A declaration can be **overwritten**: declaring two functions with the same name keeps the last one.

### Use cases

- **Declaration:** utilities and top-level helpers where hoisting lets you place the main logic first and helpers at the bottom.
- **Expression:** callbacks, event handlers, values assigned conditionally, functions stored in objects or arrays.
- **IIFE:** one-time setup that shouldn't leak variables (older module pattern, initialization code).

---

## 2. Arrow functions

### Definition

**Arrow functions** (ES6) are a shorter function-expression syntax that does **not** create its own `this`, `arguments`, `super`, or `new.target`. They capture those from the surrounding scope (lexical binding).

### Summary

```js
const add = (a, b) => a + b;          // implicit return
const square = n => n * n;            // one param: parentheses optional
const hello = () => "hi";             // no params: parentheses required
const sum = (a, b) => { return a + b; }; // block body: explicit return
const makeUser = name => ({ name });  // return an object literal: wrap in ( )
```

| |Regular function|Arrow function|
|---|---|---|
|Own `this`|Yes (depends on how it's called)|**No** (uses the surrounding `this`)|
|Own `arguments`|Yes|**No** (use rest params)|
|Can be used with `new`|Yes|**No** (TypeError)|
|Has `prototype`|Yes|No|
|Can be a generator (`function*`)|Yes|No|
|Hoisted|Declarations only|No (always an expression)|
|Best for|Methods, constructors|Callbacks, short helpers|

**Why `this` matters (preview, covered in depth in [[JS 05 - Core Concepts (Scope, Closures, this)]])**

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => { this.seconds++; }, 1000); // arrow keeps `this` = timer
  },
};
```

### Edge cases

- **Braces without `return`** give `undefined`:
    
    ```js
    const f = x => { x * 2 };   // returns undefined[1, 2, 3].map(x => { x * 2 });  // [undefined, undefined, undefined]
    ```
    
- **Returning an object literal** needs parentheses, otherwise `{}` is parsed as a block:
    
    ```js
    const bad  = () => { a: 1 };   // block with a label → returns undefinedconst good = () => ({ a: 1 });
    ```
    
- **Not suitable as object methods** if you need `this` to be the object:
    
    ```js
    const user = { name: "A", say: () => this.name }; // `this` is NOT user
    ```
    
- **No `arguments`:** `(() => arguments)()` refers to the outer function's `arguments` (or throws if none). Use `(...args) => args`.
- **Can't use `new`:** `new (() => {})` throws `TypeError: ... is not a constructor`.
- **`call`, `apply`, `bind` can't change an arrow's `this`.**
- **Line break before `=>` is a SyntaxError.**
- Arrow functions can still be **recursive** if assigned to a `const` and called by name.

### Use cases

- Array callbacks: `items.filter(x => x.active).map(x => x.id)`.
- Event handlers and timers inside classes/React components where you need the outer `this`.
- Small inline helpers and function-returning functions (`const add = a => b => a + b`).
- Avoid for: object methods relying on `this`, constructors, prototypes, and functions needing `arguments`.

---

## 3. Parameters: default and rest

### Definition

- **Parameter**: a variable named in the function definition.
- **Argument**: the actual value passed when calling.
- **Default parameter**: a fallback value used when an argument is `undefined`.
- **Rest parameter**: `...name`, which collects extra arguments into an array.

### Summary

```js
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}
greet();                  // "Hello, Guest!"
greet("Arman");           // "Hello, Arman!"
greet(undefined, "Hi");   // "Hi, Guest!"
greet(null);              // "Hello, null!"   (null does NOT trigger the default)
```

```js
function sum(...nums) {        // nums is a real array
  return nums.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3);       // 6
sum();              // 0

function log(level, ...messages) {  // rest must be last
  console.log(level, messages);
}
```

**Spread in calls (the opposite of rest)**

```js
const nums = [4, 9, 2];
Math.max(...nums);   // 9
```

**Destructured parameters (named-options pattern)**

```js
function createUser({ name, role = "user", active = true } = {}) {
  return { name, role, active };
}
createUser({ name: "A" });  // { name: "A", role: "user", active: true }
createUser();               // { name: undefined, role: "user", active: true }
```

|Concept|Rule|
|---|---|
|Missing arguments|Become `undefined` (no error)|
|Extra arguments|Ignored (unless collected with rest or `arguments`)|
|Default trigger|Only `undefined`|
|Defaults can use earlier params|`function f(a, b = a * 2) {}`|
|Rest position|Last, only one|

### Edge cases

- **`null` doesn't trigger a default**, only `undefined`. Use `??` inside the function if `null` should also fall back.
- **Defaults are evaluated each call** (unlike some other languages):
    
    ```js
    function add(x, list = []) { list.push(x); return list; }add(1);  // [1]add(2);  // [2]   (a new array every call)
    ```
    
- **Defaults can reference earlier parameters, but not later ones:**
    
    ```js
    function f(a = b, b = 1) {}f();  // ReferenceError: Cannot access 'b' before initialization (TDZ)
    ```
    
- **`function.length`** counts parameters before the first default and excludes rest: `((a, b = 1, ...c) => {}).length` → `1`.
- **`arguments` object** (regular functions only) is array-like, not a real array. Prefer rest params. Convert with `Array.from(arguments)`.
- **Parameters are passed by value (of the reference):**
    
    ```js
    function change(o) { o.x = 2; o = { x: 3 }; }const obj = { x: 1 };change(obj);obj.x;   // 2  (mutation visible; reassignment of the local `o` is not)
    ```
    
- **Too many positional parameters** is a design smell. Use an options object after 3–4 params.
- Destructuring without a default object (`function f({ a }) {}`) throws if called with no argument. Add `= {}`.

### Use cases

- Optional configuration (`fetchData(url, { retries = 3, timeout = 5000 } = {})`).
- Variadic helpers: `sum(...nums)`, `logger(level, ...messages)`, `compose(...fns)`.
- Forwarding arguments to another function: `(...args) => target(...args)`.
- Named-parameter style for readable call sites in React helpers and API utilities.

---

## 4. Return values

### Definition

`return` ends function execution and hands a value back to the caller. A function with no `return` (or a bare `return;`) gives back `undefined`.

### Summary

```js
function max(a, b) {
  if (a > b) return a;   // early return (guard clause)
  return b;
}

function noReturn() { /* ... */ }
noReturn();  // undefined
```

**Returning multiple values** — return an array or an object:

```js
const minMax = (arr) => [Math.min(...arr), Math.max(...arr)];
const [min, max] = minMax([3, 1, 9]);

const getUser = () => ({ id: 1, name: "A" });
const { id, name } = getUser();
```

**Returning a function**

```js
const makeMultiplier = (m) => (n) => n * m;
const double = makeMultiplier(2);
double(5);  // 10
```

|Situation|Result|
|---|---|
|No `return` statement|`undefined`|
|`return;`|`undefined`|
|Arrow with expression body|That expression's value|
|Arrow with `{ }` body, no `return`|`undefined`|
|Constructor called with `new`, returns an object|That object|
|Constructor called with `new`, returns a primitive|Ignored; the new instance is returned|

### Edge cases

- **Automatic Semicolon Insertion (ASI) trap:** a line break right after `return` ends the statement.
    
    ```js
    function bad() {  return  { ok: true };}bad();   // undefined
    ```
    
    Keep the value on the same line as `return` (or open the bracket there).
- Code after `return` is unreachable and never runs.
- `return` inside `forEach` only exits that callback, not the outer function.
- `finally` can override a return from `try`:
    
    ```js
    function f() { try { return 1; } finally { return 2; } }f();  // 2
    ```
    
- Functions can return `undefined` implicitly, so always check before using the result (`fn()?.value`).
- Returning large objects/arrays from a function returns a **reference**, so callers can mutate shared data.

### Use cases

- Pure computations (`calculateTotal`, `formatPrice`).
- Early returns for validation (reduces nesting).
- Returning objects for multiple results; returning functions for configuration (factories, closures).
- Returning promises from async work (see the async notes).

---

## 5. Scope basics

### Definition

**Scope** decides where a variable can be accessed. JavaScript uses **lexical (static) scope**: access is determined by where the code is _written_, not where it's called.

### Summary

|Scope|Created by|Variables visible|
|---|---|---|
|Global|Top level of a script|Everywhere|
|Function|A function body|Inside that function (all declaration types)|
|Block|`{ }` of `if`, `for`, `while`, bare blocks|`let` / `const` only (`var` ignores block scope)|

**Scope chain:** when a variable is referenced, JS looks in the current scope, then the parent scope, then outward to the global scope. The first match wins.

```js
const city = "Kolkata";           // global

function outer() {
  const greeting = "Hello";       // outer's scope
  function inner() {
    const name = "Arman";         // inner's scope
    return `${greeting}, ${name} from ${city}`;  // can read all three
  }
  return inner();
}
outer();     // "Hello, Arman from Kolkata"
// greeting;  // ReferenceError — not visible outside outer()
```

**Shadowing**

```js
let x = "global";
function outer() {
  let x = "outer";
  function inner() { return x; }   // finds "outer" first
  return inner();
}
outer();  // "outer"
```

**Lexical scope in action**

```js
const msg = "global";
function show() { return msg; }
function run() {
  const msg = "local";
  return show();     // "global" — show() uses the scope where it was written
}
run();
```

### Edge cases

- `var` is **function-scoped**, so it leaks out of `if`/`for` blocks. `let`/`const` stay inside blocks.
- **Implicit global:** assigning to an undeclared name (`count = 5`) creates a global in sloppy mode. Use `"use strict"` or modules to turn this into an error.
- **Shadowing a parameter:** `function f(a) { var a; }` doesn't reset `a`. `function f(a) { let a; }` is a SyntaxError.
- **Closures:** an inner function keeps access to outer variables even after the outer function returned. Deep dive in [[JS 05 - Core Concepts (Scope, Closures, this)]].
    
    ```js
    function counter() {  let c = 0;  return () => ++c;}const next = counter();next();  // 1next();  // 2
    ```
    
- Each call to a function creates a **new** scope. Two counters made with `counter()` don't share `c`.
- Global variables are shared everywhere, which makes them easy to overwrite by accident. Keep them to a minimum.
- Default parameter expressions live in their own scope, separate from the function body.

### Use cases

- Keep variables private to a function or block to avoid name collisions.
- Closures for data privacy, counters, memoization caches, and event-handler state.
- Block scope with `let`/`const` for loop variables and temporary values.

---

## 6. Interview questions & answers

> [!tip] How to use Cover the answer and say it out loud first. Aim for a 20–40 second answer.

### Basic

> [!question]- What is the difference between a function declaration and a function expression? A declaration (`function f() {}`) is fully hoisted, so it can be called before its line. An expression (`const f = function () {}`) is only available after the assignment runs; before that, `var` gives `undefined` (calling it throws `TypeError`) and `let/const` throw `ReferenceError` (TDZ).

> [!question]- What is an arrow function and how is it different from a regular function? A shorter syntax that doesn't have its own `this`, `arguments`, or `prototype`, and can't be used with `new`. It uses the `this` of the surrounding scope, which makes it ideal for callbacks, and unsuitable for object methods that depend on `this`.

> [!question]- What are default parameters? Fallback values used when an argument is `undefined`: `function f(a = 1) {}`. They are evaluated at call time and can reference earlier parameters.

> [!question]- What is a rest parameter? How is it different from the `arguments` object? `...args` collects remaining arguments into a **real array** and works in arrow functions. `arguments` is array-like (no `map`/`filter`), exists only in regular functions, and includes all arguments.

> [!question]- What does a function return if there's no `return`? `undefined`.

> [!question]- What are first-class functions? Functions are values: they can be assigned to variables, passed as arguments (callbacks), returned from functions, and stored in objects/arrays.

> [!question]- What's the difference between parameters and arguments? Parameters are the names in the function definition. Arguments are the values passed when calling it.

> [!question]- What is an IIFE and why use it? A function that is defined and invoked immediately: `(function () {})()`. It creates a private scope so variables don't leak into the global scope (mostly replaced by modules and block scope today).

### Intermediate

> [!question]- Why can't you use an arrow function as a constructor? Arrow functions have no `prototype` and no `[[Construct]]` behavior, so `new` throws `TypeError: ... is not a constructor`.

> [!question]- When should you NOT use an arrow function? For object methods that use `this`, constructors, prototype methods, functions that need `arguments`, and when you need `call`/`apply`/`bind` to set `this`.

> [!question]- Does `null` trigger a default parameter? No. Only `undefined` does. `f(null)` passes `null` through.

> [!question]- Are objects passed by reference or by value? JavaScript is always pass-by-value, but for objects the value is a _reference_. Mutating the object inside the function affects the caller's object; reassigning the parameter doesn't.

> [!question]- What is lexical scope? Scope is determined by where functions and variables are written in the source, not by where functions are called. Inner functions can access variables of enclosing scopes.

> [!question]- What is the scope chain? The lookup path: current scope → enclosing scope → ... → global. The first matching name is used; if none, `ReferenceError`.

> [!question]- What is a callback function? A function passed to another function to be called later (e.g., `arr.map(fn)`, `setTimeout(fn, 100)`, event handlers).

> [!question]- How do you return multiple values from a function? Return an array or object and destructure it: `const { id, name } = getUser();`.

> [!question]- What is a pure function? A function whose output depends only on its inputs and which has no side effects (no mutation of outside state, no I/O). Same input → same output.

### Tricky / follow-up

> [!question]- Why does `const f = () => { a: 1 }` return `undefined`? The `{}` is parsed as a function body, and `a: 1` is a labeled statement. Wrap the object in parentheses: `() => ({ a: 1 })`.

> [!question]- What does this return? `function f() { return \n { ok: true } }` `undefined`. Automatic semicolon insertion adds a semicolon after `return`. Put the value on the same line.

> [!question]- What is `f.length` for `(a, b = 1, ...c) => {}`? `1`. It counts parameters before the first default value, and excludes rest parameters.

> [!question]- What happens if a function declaration and a `var` have the same name? The function is hoisted first and the `var` doesn't override it at hoisting time, but a later assignment (`var a = 1`) overwrites it at runtime.

> [!question]- Why does `[1,2,3].map(x => { x * 2 })` return `[undefined, undefined, undefined]`? The block body has no `return`. Use `x => x * 2` or `x => { return x * 2; }`.

> [!question]- Can a function with default parameters see variables from its body? No. Defaults are evaluated in a separate parameter scope, so they can't see variables declared inside the function body.

> [!question]- What is the difference between `function` and `=>` regarding `this` inside `setTimeout`? A regular function callback gets its own `this` (not the object). An arrow callback keeps the `this` from where it was defined, so `this.value++` inside an arrow in a method updates the object.

---

## 7. Predict the output

```js
// 1
console.log(sum(2, 3));
function sum(a, b) { return a + b; }

// 2
console.log(mul(2, 3));
var mul = function (a, b) { return a * b; };

// 3
const f = () => { a: 1 };
console.log(f());

// 4
function g(a, b = 2) { return [a, b]; }
console.log(g(1, undefined), g(1, null));

// 5
console.log(((a, b = 1, ...c) => {}).length);

// 6
function h() {
  return
  { ok: true };
}
console.log(h());

// 7
var a = 1;
function a() {}
console.log(typeof a);

// 8
function change(o) { o.x = 2; o = { x: 3 }; }
const obj = { x: 1 };
change(obj);
console.log(obj.x);

// 9
function add(x, list = []) { list.push(x); return list; }
add(1);
console.log(add(2));

// 10
function k(a = b, b = 1) { return a; }
console.log(k());

// 11
let x = "global";
function outer() {
  let x = "outer";
  function inner() { return x; }
  return inner();
}
console.log(outer());

// 12
const counter = () => { let c = 0; return ++c; };
counter();
console.log(counter());
```

<details> <summary>Answers</summary>

1. `5` (declaration hoisted)
2. `TypeError: mul is not a function`
3. `undefined` (block body with a label)
4. `[1, 2]` and `[1, null]`
5. `1`
6. `undefined` (ASI after `return`)
7. `"number"`
8. `2`
9. `[2]` (new default array each call)
10. `ReferenceError: Cannot access 'b' before initialization`
11. `"outer"`
12. `1` (each call creates a fresh `c`)

</details>

---

## 8. Coding practice

**1. Sum any number of arguments (rest)**

```js
const sum = (...nums) => nums.reduce((a, b) => a + b, 0);
```

**2. Average with validation**

```js
function average(...nums) {
  if (nums.length === 0) return 0;
  return nums.reduce((a, b) => a + b, 0) / nums.length;
}
```

**3. Options object with defaults**

```js
function createUser({ name, role = "user", active = true } = {}) {
  if (!name) throw new Error("name is required");
  return { name, role, active };
}
```

**4. Call a function `n` times**

```js
function repeat(n, fn) {
  for (let i = 0; i < n; i++) fn(i);
}
repeat(3, (i) => console.log("run", i));
```

**5. Function that returns a function (multiplier)**

```js
const makeMultiplier = (m) => (n) => n * m;
const triple = makeMultiplier(3);
triple(4);  // 12
```

**6. Swap two values using a returned array**

```js
const swap = (a, b) => [b, a];
let [x, y] = [1, 2];
[x, y] = swap(x, y);
```

**7. Convert to arrow and explain the difference**

```js
// regular
function isEven(n) { return n % 2 === 0; }
// arrow
const isEven2 = (n) => n % 2 === 0;
```

**8. Make a function run only once**

```js
function once(fn) {
  let called = false, result;
  return (...args) => {
    if (!called) { called = true; result = fn(...args); }
    return result;
  };
}
```

---

## 9. Quick-recall cheat sheet

|Need|Use|
|---|---|
|Callable before definition|Function declaration|
|Short callback|Arrow function|
|Object/array result from arrow|`=> ({ ... })`|
|Method that uses `this`|Method shorthand / regular function|
|Optional argument|Default param (`a = 1`)|
|Unlimited arguments|Rest `...args`|
|Spread array into arguments|`fn(...arr)`|
|Many options|Destructured object param with `= {}`|
|Multiple return values|Return array / object|
|Private state|Closure (function returning a function)|
|Avoid globals|Block scope + `const` / `let`, modules|

---

## Related notes

- [[JS 01 - Language Basics]]
- [[JS 03 - Arrays & Objects]]
- [[JS 05 - Core Concepts (Scope, Closures, this)]]