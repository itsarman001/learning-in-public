# JS 07 — ES6+ Features

> [!info] Scope
> Destructuring · Spread & rest · Optional chaining (`?.`) · Nullish coalescing (`??`) · Modules · `Map` & `Set` · Advanced array methods (`reduce`, `flat`, `flatMap`)
> Previous: [[JS 06 - Functional Concepts]] · Next: [[JS 08 - Async JavaScript]]
> `WeakMap`, `WeakSet`, `Symbol`, generators and iterators are in [[JS 14 - Advanced Language Features]].

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| Destructuring | Unpack arrays by **position**, objects by **key name**. Defaults apply only to `undefined`. |
| Destructuring `null`/`undefined` | Throws `TypeError`. Use `= {}` / `?.` to guard. |
| Spread `...` (expand) | Arrays, function calls, object literals. **Shallow** copy. Later keys override earlier ones. |
| Rest `...` (collect) | Function parameters and destructuring. Must be **last**. |
| `?.` | Short-circuits to `undefined` when the left side is `null`/`undefined`. Doesn't mask other errors. |
| `??` | Fallback only for `null`/`undefined`. `\|\|` falls back for **any falsy** value. |
| Modules | `import`/`export`. Named vs default. Live bindings. Strict mode, evaluated once. |
| `Map` | Any key type, insertion-ordered, `size`. Better than an object for dynamic keys. |
| `Set` | Unique values, O(1) `has`. `[...new Set(arr)]` removes duplicates. |
| `reduce` | Folds an array to one value. Always give an initial value. Always return the accumulator. |
| `flat` / `flatMap` | `flat(depth)` flattens nested arrays. `flatMap` = `map` + `flat(1)` (one level only). |

---

## 1. Destructuring

### Definition
**Destructuring** is syntax for unpacking values from arrays (and any iterable) or properties from objects into separate variables.

### Summary

**Array destructuring: by position**
```js
const [a, b] = [10, 20];              // a = 10, b = 20
const [first, , third] = [1, 2, 3];   // skip with an empty slot
const [x = 5, y = 7] = [1];           // x = 1, y = 7 (default used for missing/undefined)
const [head, ...tail] = [1, 2, 3, 4]; // head = 1, tail = [2, 3, 4]

let p = 1, q = 2;
[p, q] = [q, p];                      // swap without a temp variable
const [ch1, ch2] = "hi";              // works on any iterable (strings, Sets, Maps)
```

**Object destructuring: by key**
```js
const user = { id: 1, name: "Asha", address: { city: "Kolkata", pin: 700001 }, tags: ["a"] };

const { name, id } = user;                       // by property name
const { name: userName } = user;                 // rename → userName
const { role = "guest" } = user;                 // default
const { name: n = "Anon" } = user;               // rename + default
const { address: { city } } = user;              // nested (address itself is NOT a variable)
const { tags: [firstTag] } = user;               // nested array
const { id: _id, ...others } = user;             // rest: everything except id
const key = "name";
const { [key]: dynamic } = user;                 // computed key
```

**In function parameters**
```js
function greet({ name, age = 18 } = {}) {        // = {} lets you call greet() safely
  return `${name} (${age})`;
}
greet({ name: "A" });                            // "A (18)"

const pairs = Object.entries({ a: 1, b: 2 });
for (const [key, value] of pairs) console.log(key, value);

const [state, setState] = useState(0);           // array destructuring in React
```

**Assigning to existing variables (object form needs parentheses)**
```js
let a, b;
({ a, b } = { a: 1, b: 2 });
```

| Feature | Array | Object |
|---|---|---|
| Matches by | Position | Property name |
| Works on | Any iterable | Any object (and primitives with wrapper props) |
| Skip items | `[a, , c]` | Just don't list them |
| Rename | Not needed | `{ old: newName }` |
| Rest | `[a, ...rest]` → array | `{ a, ...rest }` → object |
| Default | `[a = 1]` | `{ a = 1 }` |

### Edge cases
- **Defaults apply only to `undefined`**, not `null`: `const { a = 5 } = { a: null }` → `a` is `null`.
- **Destructuring `null`/`undefined` throws:** `const { a } = null` → `TypeError: Cannot destructure property 'a' of 'null'`. Guard with `= {}` or `?? {}`.
- **Nested destructuring of a missing parent throws:** `const { a: { b } } = {}` → `TypeError` (`a` is `undefined`). Use defaults at each level: `{ a: { b } = {} } = {}`.
- **Destructuring is shallow:** you get references to the same nested objects, not copies.
- **Array destructuring requires an iterable:** `const [a] = {}` throws (`{}` isn't iterable). `const [a] = null` throws.
- **Missing items are `undefined`:** `const [a, b] = [1]` → `b` is `undefined`.
- **Rest must be last** and can't have a trailing comma.
- **Statement starting with `{`** is parsed as a block: `{ a, b } = obj;` is a SyntaxError. Wrap in parentheses.
- **ASI trap with array swap:** a line starting with `[` can merge with the previous line. End the previous statement with `;`.
- **Primitives:** `const { length } = "hello"` → `5`. Works through autoboxing.
- **Defaults are lazy:** evaluated only when needed. `const { a = expensive() } = obj` doesn't call `expensive` if `a` exists.
- **Order of object keys doesn't matter**, only names.
- **Renaming reads as "from: to":** `{ name: userName }` means take `name`, store in `userName`. Easy to reverse in interviews.

### Use cases
- Pulling fields from API responses, props, and options objects.
- Function parameters with defaults (named-argument style).
- Returning multiple values (`const [min, max] = range(arr)`), swapping, React hooks.
- Removing a key immutably: `const { password, ...safeUser } = user`.

---

## 2. Spread and rest (`...`)

### Definition
The same `...` syntax does two opposite jobs depending on position:
- **Spread** *expands* an iterable/object into individual elements/properties.
- **Rest** *collects* the remaining elements/properties into an array/object.

### Summary

| | Spread (expand) | Rest (collect) |
|---|---|---|
| Where | Array literal, call arguments, object literal | Function parameters, destructuring patterns |
| Direction | One thing → many | Many → one array/object |
| Position | Anywhere | **Last** only |

```js
// SPREAD
const a = [1, 2], b = [3, 4];
const merged = [...a, ...b, 5];            // [1, 2, 3, 4, 5]
const copy = [...a];                       // shallow copy
Math.max(...[3, 9, 2]);                    // 9  (array → arguments)
[..."abc"];                                // ["a", "b", "c"]
[...new Set([1, 1, 2])];                   // [1, 2]

const defaults = { theme: "light", size: 12 };
const settings = { ...defaults, size: 14 };     // { theme: "light", size: 14 }
const withId = { ...user, id: 99 };             // later key wins

// REST
function sum(...nums) { return nums.reduce((t, n) => t + n, 0); }
sum(1, 2, 3);                                    // 6

const [first, ...others] = [1, 2, 3];            // others = [2, 3]
const { a: x, ...remaining } = { a: 1, b: 2, c: 3 };  // remaining = { b: 2, c: 3 }
```

**Order matters for objects**
```js
({ ...{ a: 1, b: 2 }, a: 9 });   // { a: 9, b: 2 }   (override after the spread)
({ a: 9, ...{ a: 1, b: 2 } });   // { a: 1, b: 2 }   (spread overrides the earlier a)
```

**Immutable update patterns**
```js
const add    = [...list, item];
const remove = list.filter(i => i.id !== id);
const update = list.map(i => (i.id === id ? { ...i, done: true } : i));
const nested = { ...state, user: { ...state.user, name: "New" } };
```

### Edge cases
- **Shallow only:** spread copies top-level properties; nested objects/arrays are still shared references. Use `structuredClone` for a deep copy.
- **Spread in arrays/calls requires an iterable:** `[...{ a: 1 }]` throws `TypeError`. But **object spread** accepts any object.
- **`{ ...null }` and `{ ...undefined }`** give `{}` (no error). `[...null]` throws.
- **Spreading arrays/strings into objects** gives index keys: `{ ...["a", "b"] }` → `{ 0: "a", 1: "b" }`; `{ ..."hi" }` → `{ 0: "h", 1: "i" }`.
- **Object spread copies own enumerable properties only** (including symbols). It doesn't copy the prototype, so a class instance becomes a plain object (methods on the prototype are lost). Getters are **evaluated** and copied as plain values.
- **Rest in objects collects own enumerable properties** not already destructured.
- **Rest parameters vs `arguments`:** rest is a real array, works in arrow functions, and only captures the *remaining* arguments.
- **Rest must be last:** `function f(...a, b) {}` and `[...a, b] = arr` are SyntaxErrors.
- **Huge arrays in calls:** `Math.max(...hugeArray)` can throw `RangeError` (argument limit, roughly 100k+). Use `reduce` or a loop.
- **Performance trap:** spreading inside a `reduce` loop (`{ ...acc, [k]: v }`) is O(n²). Mutate a local accumulator instead.
- **`[...arr]` vs `arr.slice()`**: equivalent shallow copies. Spread also works on any iterable.
- **Spread doesn't dedupe or merge deeply:** `{ ...a, ...b }` replaces nested objects entirely (`{ ...a.n, ...b.n }` needed to merge one level deeper).
- **Duplicate keys:** later wins silently. Be careful with the order of defaults and overrides.

### Use cases
- Copying/merging arrays and objects without mutation (React/Redux state).
- Passing array items as arguments, forwarding arguments (`(...args) => fn(...args)`).
- Variadic functions, omitting properties, default configs with overrides.
- Converting iterables (`Set`, `Map`, `NodeList`, strings) into arrays.

---

## 3. Optional chaining (`?.`)

### Definition
**Optional chaining** safely accesses a property, index, or method of something that might be `null` or `undefined`. If the value *before* `?.` is `null`/`undefined`, the whole chain **short-circuits** and evaluates to `undefined` instead of throwing.

### Summary

```js
user?.address?.city          // property access
user?.["first-name"]         // bracket access
list?.[0]                    // array index
user.getName?.()             // call only if the method exists
callback?.(data)             // call only if it's a function (not null/undefined)

const city = user?.address?.city ?? "Unknown";   // with a default
```

**What it replaces**
```js
// Before
const city = user && user.address && user.address.city;

// After
const city = user?.address?.city;
```

| Value before `?.` | Result |
|---|---|
| `null` / `undefined` | `undefined` (rest of the chain skipped) |
| `0`, `""`, `false`, `NaN` | Continues normally (only `null`/`undefined` stop it) |
| Object / function | Continues normally |

### Edge cases
- **Only checks `null`/`undefined`, not falsiness:** `(0)?.toFixed()` still runs. Unlike `&&`.
- **Short-circuits the whole chain:** `a?.b.c.d` → if `a` is nullish, the entire expression is `undefined` (no error on `.c`/`.d`).
- **Parentheses break short-circuiting:** `(a?.b).c` → if `a` is nullish, `(undefined).c` **throws**.
- **Missing property in the middle still throws:** `a?.b.c` with `a = {}` → `a.b` is `undefined`, then `.c` throws. Write `a?.b?.c`.
- **Only guards the *left* side:** `obj.method?.()` skips the call if `method` is nullish, but if `method` is a non-function (e.g. `5`), it throws `TypeError`.
- **Can't be used for assignment:** `a?.b = 1` is a SyntaxError.
- **Not allowed with `new` or template tags:** `new a?.()` is a SyntaxError.
- **Undeclared variables still throw:** `notDeclared?.x` → `ReferenceError`. `?.` only protects values, not missing declarations.
- **`delete a?.b` is allowed** (returns `true` if `a` is nullish).
- **Result is `undefined`, never `null`:** even when the left side was `null`.
- **Overuse hides bugs:** `?.` everywhere can mask a missing required value. Use it where absence is expected; validate where it isn't.
- **Not a try/catch:** doesn't catch errors thrown *inside* a method call, only a nullish base.

### Use cases
- Reading nested API data where fields may be missing (`data?.user?.profile?.avatar`).
- Optional callbacks and props (`onChange?.(value)`).
- DOM lookups (`document.querySelector(".x")?.classList.add("on")`).
- Optional array elements (`results?.[0]?.id`).

---

## 4. Nullish coalescing (`??`) and logical assignment

### Definition
`a ?? b` returns `b` **only if `a` is `null` or `undefined`**; otherwise it returns `a`. It's a safer default operator than `||`, which treats every falsy value as missing.

### Summary

| Expression | `a \|\| "x"` | `a ?? "x"` |
|---|---|---|
| `a = undefined` | `"x"` | `"x"` |
| `a = null` | `"x"` | `"x"` |
| `a = 0` | `"x"` | **`0`** |
| `a = ""` | `"x"` | **`""`** |
| `a = false` | `"x"` | **`false`** |
| `a = NaN` | `"x"` | **`NaN`** |
| `a = "hi"` | `"hi"` | `"hi"` |

```js
const count = input.count ?? 0;             // keeps a real 0
const name  = user?.name ?? "Guest";        // pairs with ?.
const port  = config.port ?? 3000;

// Logical assignment operators (ES2021)
config.retries ??= 3;       // assign only if null/undefined
settings.theme ||= "light"; // assign if falsy
user.admin &&= checkRole(); // assign only if truthy
```

**Choosing the default operator**

| Need | Use |
|---|---|
| Missing means `null`/`undefined` only | `??` |
| Any falsy value should fall back (empty string = missing) | `\|\|` |
| Default parameter (only `undefined`) | `function f(a = 1)` |
| Check presence of a key | `"key" in obj` / `Object.hasOwn(obj, "key")` |

### Edge cases
- **`0`, `""`, `false` are kept** by `??`. Using `||` here is the classic bug (a real `0` replaced by the default).
- **Mixing `??` with `||` or `&&` without parentheses is a SyntaxError:** `a || b ?? c` fails. Write `(a || b) ?? c`.
- **Default parameters vs `??`:** defaults trigger only for `undefined`; `null` passes through. `??` handles both.
- **`??=` short-circuits:** the assignment (and any setter) doesn't run when the value isn't nullish.
- **`null ?? undefined`** → `undefined`; `undefined ?? null` → `null` (it returns the right-hand operand as is).
- **Right side is lazy:** `a ?? expensive()` only calls `expensive()` when needed.
- **Combine carefully with `?.`:** `a?.b ?? "d"` is fine. If `a.b` is `0`, you get `0`, not `"d"`.
- **Doesn't convert types:** `"" ?? "x"` is `""`. If you want a non-empty string, use `||` or an explicit check.
- **Form inputs are strings:** `input.value ?? 0` never falls back (it's `""`, not nullish). Convert and validate explicitly.

### Use cases
- Defaults for config, counts, flags, and API values where `0`/`false`/`""` are valid.
- Lazy initialization with `??=` (caches: `cache[key] ??= compute(key)`).
- Safe reads: `data?.items?.length ?? 0`.

---

## 5. Modules (ES Modules)

### Definition
An **ES module** is a file with its own scope that explicitly **exports** values and **imports** values from other modules. Variables inside a module are private unless exported.

### Summary

**Named vs default exports**
```js
// math.js
export const PI = 3.14159;                   // named
export function add(a, b) { return a + b; }  // named
const secret = 42;                           // private (not exported)
export default function multiply(a, b) { return a * b; }  // default (one per module)

// app.js
import multiply, { PI, add } from "./math.js";       // default + named
import { add as sum } from "./math.js";              // rename
import * as math from "./math.js";                   // namespace object
import "./polyfill.js";                              // side-effect only
```

| | Named export | Default export |
|---|---|---|
| Count per module | Many | One |
| Import syntax | `import { x } from` (exact name, or `as`) | `import anyName from` |
| Refactoring/tooling | Better (consistent names, autocomplete) | Name can differ per file |
| Re-export | `export { x } from "./a.js"` | `export { default } from "./a.js"` |

**Re-exporting (barrel files)**
```js
// components/index.js
export { Button } from "./Button.js";
export { default as Modal } from "./Modal.js";
export * from "./utils.js";          // re-exports all named exports (not default)
```

**Dynamic import (lazy loading)**
```js
button.addEventListener("click", async () => {
  const { openChart } = await import("./chart.js");   // loaded on demand, returns a promise
  openChart();
});
```

**Using modules in the browser**
```html
<script type="module" src="./main.js"></script>
```

**Live bindings**
```js
// counter.js
export let count = 0;
export function inc() { count++; }

// main.js
import { count, inc } from "./counter.js";
console.log(count);   // 0
inc();
console.log(count);   // 1  (imports are live views, not copies)
// count = 5;          // TypeError: Assignment to constant variable (imports are read-only)
```

**ES modules vs CommonJS (Node)**

| | ES Modules | CommonJS |
|---|---|---|
| Syntax | `import` / `export` | `require()` / `module.exports` |
| Loading | Static (analyzed before running), async-capable | Dynamic, synchronous |
| Values | **Live bindings** | Copies of values at `require` time |
| Strict mode | Always | Only if declared |
| Top-level `await` | Yes | No |
| Tree shaking | Yes (static structure) | Hard |
| `this` at top level | `undefined` | `module.exports` |
| `__dirname` / `__filename` | Not available (use `import.meta.url`) | Available |
| Enable in Node | `"type": "module"` or `.mjs` | Default / `.cjs` |

### Edge cases
- **`export default` is not a named export:** `import { default as x }` works; `import { something }` for a default does not.
- **Named imports must match exactly** (case-sensitive) or be renamed with `as`. A wrong name is a `SyntaxError` at load time.
- **Imports are hoisted** and executed before the module body. They must be at the **top level** (use `import()` for conditional loading).
- **Evaluated once:** a module runs a single time and is cached (singleton). Importing from many files shares the same instance and state.
- **Imported bindings are read-only,** but you can mutate imported *objects*: `import { config } from "./c.js"; config.debug = true;` works.
- **Circular dependencies** can leave a binding in the TDZ (`ReferenceError`) or `undefined` when accessed too early. Restructure or move shared code to a third module.
- **Browsers require file extensions and valid paths:** `import "./utils"` fails; use `"./utils.js"`. Bundlers (Vite/Webpack) often relax this.
- **`file://` doesn't work for modules** (CORS). Serve over HTTP (`npx serve`, Vite).
- **Modules are deferred and strict** by default, with their own scope. Top-level variables don't become globals, and `this` is `undefined`.
- **`import()` returns a promise** of the module namespace; default export is at `.default`.
- **Barrel files** (`index.js` re-exports) are convenient but can hurt tree shaking and slow builds. Import directly from the source file in large apps.
- **Default import of CommonJS** gives `module.exports`; named imports from CJS may not work without interop.
- **`export * ` doesn't re-export the default** export.
- **Top-level `await`** makes the importing module wait, which can delay startup.

### Use cases
- Splitting code into files with clear public APIs, avoiding globals.
- Lazy loading routes/components/heavy libraries (`import()`, `React.lazy`).
- Shared constants, utilities, and API clients across a project; tree shaking to ship less code.

---

## 6. `Map` and `Set`

### Definition
- **`Map`** is a collection of **key–value pairs** where keys can be **any type** and iteration follows **insertion order**.
- **`Set`** is a collection of **unique values** (no duplicates), also in insertion order.

Both use **SameValueZero** equality: `NaN` equals `NaN`, and `+0` equals `-0`; objects are compared by **reference**.

### Summary

**Map API**
```js
const m = new Map();
m.set("a", 1).set(2, "two").set({ id: 1 }, "obj");   // set is chainable
m.get("a");          // 1
m.has(2);            // true
m.delete("a");       // true
m.size;              // 2  (a property, not a method)
m.clear();

const m2 = new Map([["x", 1], ["y", 2]]);            // from entries
for (const [key, value] of m2) console.log(key, value);
[...m2.keys()]; [...m2.values()]; [...m2.entries()];
m2.forEach((value, key) => {});                      // note: (value, key)

Object.fromEntries(m2);                              // Map → object  { x: 1, y: 2 }
new Map(Object.entries({ a: 1 }));                   // object → Map
```

**Set API**
```js
const s = new Set([1, 2, 2, 3]);    // {1, 2, 3}
s.add(4).add(1);                    // adding a duplicate is ignored
s.has(2);                           // true  (O(1) on average)
s.delete(3);
s.size;                             // 3
[...s];                             // to array

const unique = [...new Set(arr)];                   // remove duplicates
```

**Set operations**
```js
const a = new Set([1, 2, 3]), b = new Set([2, 3, 4]);
const union        = new Set([...a, ...b]);                 // {1, 2, 3, 4}
const intersection = new Set([...a].filter(x => b.has(x))); // {2, 3}
const difference   = new Set([...a].filter(x => !b.has(x)));// {1}
// Newer built-ins: a.union(b), a.intersection(b), a.difference(b), a.isSubsetOf(b) (check browser support)
```

**`Map` vs plain object**

| | `Map` | Object |
|---|---|---|
| Key types | Any (objects, functions, numbers...) | Strings and symbols (numbers become strings) |
| Order | Insertion order (always) | Integer-like keys first, then insertion order |
| Size | `map.size` | `Object.keys(obj).length` |
| Default keys | None | Inherits from prototype (`toString`, `__proto__`) |
| Iteration | Directly iterable | `Object.keys/values/entries` first |
| Frequent add/remove | Optimized | Not designed for it |
| JSON | Not serialized directly (`{}`) | Yes |
| Best for | Dynamic keys, caches, counters, non-string keys | Fixed-shape records, JSON data |

**`Set` vs array**

| | `Set` | Array |
|---|---|---|
| Duplicates | Not allowed | Allowed |
| Membership test | `has` O(1) average | `includes` O(n) |
| Index access | No | Yes |
| `map`/`filter`/`reduce` | No (convert first) | Yes |
| Order | Insertion | Index |

**`WeakMap` / `WeakSet` (preview):** keys must be objects, hold them weakly (garbage-collectable), can't be iterated, and have no `size`. See [[JS 14 - Advanced Language Features]].

### Edge cases
- **Object keys are by reference:** `m.set({}, 1); m.get({})` → `undefined` (different object). Keep a reference to the key.
- **`NaN` works as a key/value:** `new Set([NaN, NaN]).size` → `1`. `[NaN].includes(NaN)` is `true`, but `indexOf(NaN)` is `-1`.
- **`+0` and `-0` are the same key.**
- **Objects in a Set are compared by reference:** `new Set([{}, {}]).size` → `2`. Deduping objects needs a key (`Map` by id).
- **`size` is a property:** `set.size`, not `set.size()`. (Arrays use `length`.)
- **Setting an existing key keeps its original position** in a Map (value is updated). Deleting and re-adding moves it to the end.
- **`Map` doesn't serialize with `JSON.stringify`** (gives `{}`). Convert first: `JSON.stringify([...map])` or `Object.fromEntries(map)`.
- **`forEach` argument order differs:** `map.forEach((value, key) => ...)`; `set.forEach((value, value2) => ...)`.
- **Adding items during iteration** means they will be visited; deleting items not yet visited skips them.
- **`new Set("hello")`** → `{ "h", "e", "l", "o" }` (strings are iterable).
- **`map.get(missing)`** returns `undefined`; use `map.get(k) ?? default` or `map.has(k)` when `undefined` is a valid value.
- **Sets have no `map`/`filter`:** spread to an array first. Newer Set methods (`union`, `intersection`...) exist but check browser support.
- **Memory:** a `Map` keeps strong references to keys, so entries are never garbage-collected until removed (use `WeakMap` for object-keyed caches that shouldn't leak).
- **Sorting a Map:** convert to an array of entries, sort, then rebuild: `new Map([...m].sort((a, b) => a[1] - b[1]))`.

### Use cases
- **Map:** frequency counters, caches/memoization, LRU cache, grouping by key, lookups keyed by DOM nodes or objects, graph adjacency lists.
- **Set:** removing duplicates, fast membership checks, tracking visited items, tags/selected IDs, set math (union/intersection).

---

## 7. Advanced array methods: `reduce`, `flat`, `flatMap`

### Definition
- **`reduce`** folds an array into a **single value** by applying a function that carries an **accumulator**.
- **`flat(depth)`** returns a new array with nested arrays flattened up to `depth` levels.
- **`flatMap(fn)`** maps each element with `fn`, then flattens the result by **one level**.

### Summary

**`reduce` signature**
```js
array.reduce((accumulator, current, index, array) => newAccumulator, initialValue);
```

```js
[1, 2, 3, 4].reduce((sum, n) => sum + n, 0);                     // 10
[3, 9, 2].reduce((max, n) => Math.max(max, n), -Infinity);       // 9
[[1, 2], [3]].reduce((flat, arr) => flat.concat(arr), []);       // [1, 2, 3]
```

**`reduce` patterns**

| Goal | Code |
|---|---|
| Sum | `nums.reduce((t, n) => t + n, 0)` |
| Count occurrences | `arr.reduce((c, x) => (c[x] = (c[x] ?? 0) + 1, c), {})` |
| Group by | `users.reduce((g, u) => { (g[u.role] ??= []).push(u); return g; }, {})` |
| Index by id | `users.reduce((m, u) => (m[u.id] = u, m), {})` |
| Unique | `arr.reduce((u, x) => u.includes(x) ? u : [...u, x], [])` |
| Min / max | `nums.reduce((a, b) => Math.min(a, b), Infinity)` |
| Pipeline | `fns.reduce((v, fn) => fn(v), input)` |
| Sum of a field | `cart.reduce((t, i) => t + i.price * i.qty, 0)` |

```js
const users = [
  { id: 1, name: "Asha", role: "admin" },
  { id: 2, name: "Ravi", role: "user" },
  { id: 3, name: "Meera", role: "user" },
];

const byRole = users.reduce((groups, u) => {
  (groups[u.role] ??= []).push(u);
  return groups;
}, {});
// { admin: [Asha], user: [Ravi, Meera] }
```

**`flat`**
```js
[1, [2, [3, [4]]]].flat();          // [1, 2, [3, [4]]]   (depth 1 by default)
[1, [2, [3, [4]]]].flat(2);         // [1, 2, 3, [4]]
[1, [2, [3, [4]]]].flat(Infinity);  // [1, 2, 3, 4]
[1, , 3].flat();                    // [1, 3]  (empty slots are removed)
```

**`flatMap`**
```js
[1, 2, 3].flatMap(n => [n, n * 2]);                   // [1, 2, 2, 4, 3, 6]  (expand)
["a b", "c d"].flatMap(s => s.split(" "));            // ["a", "b", "c", "d"]
[1, 2, 3, 4].flatMap(n => (n % 2 ? [] : [n]));        // [2, 4]  (filter + map in one pass)
orders.flatMap(order => order.items);                 // all items from all orders
[1, 2, 3].flatMap(n => [[n]]);                        // [[1], [2], [3]]  (only ONE level flattened)
```

**Other modern array methods worth knowing**

| Method | What it does |
|---|---|
| `Array.from(iterable, mapFn)` | Array from iterable/array-like; `Array.from({ length: 3 }, (_, i) => i)` → `[0, 1, 2]` |
| `at(-1)` | Last element (negative indexes) |
| `findLast` / `findLastIndex` | Search from the end |
| `includes` | Membership check (works with `NaN`) |
| `toSorted`, `toReversed`, `toSpliced`, `with` (ES2023) | Non-mutating versions |
| `Object.groupBy(arr, fn)` (ES2024) | Group items by key (check browser support) |
| `entries()`, `keys()`, `values()` | Iterators over indexes/values |
| `reduceRight` | `reduce` from the end |

### Edge cases
- **`reduce` on an empty array without an initial value throws** `TypeError: Reduce of empty array with no initial value`. Always pass one.
- **Forgetting to `return` the accumulator** makes it `undefined` on the next iteration. With arrow functions + braces, you need an explicit `return`.
- **Wrong initial value type:** summing with no initial value on objects uses the first object as the accumulator (`[{n:1},{n:2}].reduce((a,b) => a + b.n)` → `"[object Object]2"`). Provide `0`.
- **Without an initial value,** the first element is the accumulator and iteration starts at index 1.
- **Spreading the accumulator each iteration** (`{ ...acc, [k]: v }` or `[...acc, x]`) is O(n²). Mutate a local accumulator and return it (it's your own object, so that's safe).
- **`reduce` isn't always clearer:** a long `reduce` that does several things is harder to read than `map` + `filter` or a `for...of` loop. Use it when you're genuinely folding to one value.
- **`flat` depth defaults to `1`** and only flattens arrays, not array-likes or objects. It removes empty slots (holes), but keeps `undefined` and `null`.
- **`flatMap` flattens exactly one level.** For deeper results, call `flat` afterwards.
- **`flatMap` callback must return an array to expand;** returning a non-array just adds the value; returning `[]` removes the item.
- **Shallow:** `flat`/`flatMap` don't clone nested objects.
- **`reduce` passes `(acc, cur, index, array)`:** don't pass functions with extra parameters directly (the `parseInt` issue shows up with `map` the same way).
- **`flat(Infinity)` on circular nested arrays** recurses forever and throws.
- **`reduce` can't short-circuit.** Use `some`, `every`, `find`, or a loop with `break` when you can stop early.
- **Mutation inside `reduce`:** mutating the *accumulator* is fine; mutating the *input array/items* is not.

### Use cases
- Totals, averages, counts, grouping, indexing API data by id.
- Flattening nested API results (`orders.flatMap(o => o.items)`), tag lists, search results.
- Combining filter + map in one pass; expanding one item into many (e.g. splitting text into words).
- Building objects/maps/pipelines from lists.

---

## 8. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### Destructuring, spread & rest

> [!question]- What is destructuring?
> Syntax that unpacks values from arrays/iterables (by position) or properties from objects (by name) into variables: `const { name, age } = user; const [a, b] = arr;`.

> [!question]- How do you rename and set a default during object destructuring?
> `const { name: userName = "Guest" } = user;` Read it as "take `name`, store in `userName`, default `"Guest"` if `undefined`".

> [!question]- When does a destructuring default apply?
> Only when the value is `undefined`. `null`, `0`, `""` and `false` do not trigger it.

> [!question]- What happens when you destructure `null` or `undefined`?
> A `TypeError` (e.g. "Cannot destructure property 'a' of 'undefined'"). Guard with a default `= {}` or `?? {}`.

> [!question]- How do you swap two variables without a temp?
> `[a, b] = [b, a];` (make sure the previous statement ends with a semicolon).

> [!question]- What is the difference between spread and rest?
> Spread expands an iterable/object into individual elements/properties (`[...a]`, `fn(...args)`, `{...obj}`); rest collects the remaining items into an array/object (`function f(...args)`, `const [x, ...others] = arr`). Rest must be last.

> [!question]- Is spread a deep copy?
> No, it's shallow. Nested objects/arrays are shared references. Use `structuredClone` for deep copies or copy each level you change.

> [!question]- What is the order of precedence in `{ ...a, ...b }`?
> Later properties override earlier ones, so `b`'s values win for duplicate keys.

> [!question]- How do you remove a property from an object immutably?
> `const { password, ...safeUser } = user;` `safeUser` has everything except `password`.

> [!question]- Does object spread copy the prototype or getters?
> No prototype (class instances become plain objects). Getters are evaluated and copied as plain values. Only own enumerable properties are copied.

> [!question]- Can you spread an object into an array?
> No: `[...{a: 1}]` throws because plain objects aren't iterable. You can spread an object into another object, or use `Object.entries`.

### `?.` and `??`

> [!question]- What is optional chaining?
> `?.` short-circuits and returns `undefined` if the value before it is `null`/`undefined`, avoiding a `TypeError` when reading nested properties or calling optional methods (`user?.address?.city`, `fn?.()`).

> [!question]- Does `?.` check for falsy values?
> No, only `null` and `undefined`. `0`, `""`, and `false` continue the chain.

> [!question]- What is the difference between `??` and `||`?
> `||` returns the right side for any falsy left value (`0`, `""`, `false`, `NaN`, `null`, `undefined`). `??` only does so for `null`/`undefined`, so valid values like `0` are preserved.

> [!question]- Can you mix `??` with `||` or `&&`?
> Only with parentheses. `a || b ?? c` is a SyntaxError; write `(a || b) ?? c`.

> [!question]- What do `??=`, `||=`, and `&&=` do?
> Logical assignment: assign only if the left is nullish (`??=`), falsy (`||=`), or truthy (`&&=`). They short-circuit, so the assignment doesn't happen otherwise.

> [!question]- Why can `(a?.b).c` still throw?
> Parentheses end the short-circuit. If `a` is nullish, `(a?.b)` is `undefined`, then `.c` throws.

> [!question]- Can you use `?.` on the left side of an assignment?
> No. `a?.b = 1` is a SyntaxError.

### Modules

> [!question]- Named export vs default export?
> A module can have many named exports (imported by exact name with `{}` or renamed with `as`) and one default export (imported under any name). Named exports are better for tooling and refactoring.

> [!question]- What are live bindings in ES modules?
> Imported names are live, read-only views of the exporter's variables. If the exporting module changes the variable, importers see the new value. CommonJS exports copy values at `require` time.

> [!question]- ES modules vs CommonJS?
> ESM: static `import`/`export`, async-capable, live bindings, strict mode, tree-shakeable, top-level `await`. CommonJS: dynamic `require`/`module.exports`, synchronous, value copies, Node's legacy default.

> [!question]- What is a dynamic `import()`?
> A function-like expression that loads a module on demand and returns a promise for the module namespace. Used for code splitting and lazy loading.

> [!question]- Are modules executed once?
> Yes. A module is evaluated a single time and cached, so all importers share the same instance and state.

> [!question]- Why does `import "./utils"` fail in the browser?
> Browsers need the full path with extension (`./utils.js`) and an HTTP(S) origin. Bundlers resolve extensions for you.

> [!question]- What are barrel files and what is their downside?
> `index.js` files that re-export many modules for shorter imports. In large apps they can hurt tree shaking, build speed and create circular dependencies.

> [!question]- What is tree shaking?
> Removing unused exports from the final bundle. It relies on ES modules' static structure.

### Map & Set

> [!question]- Difference between `Map` and a plain object?
> `Map` accepts any key type, keeps insertion order, has `size`, is directly iterable and has no inherited keys. Objects have string/symbol keys, prototype keys, and no `size`. Use `Map` for dynamic keys/caches; objects for fixed-shape records and JSON.

> [!question]- How do you remove duplicates from an array?
> `[...new Set(arr)]` (primitives). For objects, dedupe by a key using a `Map`/`Set` of ids.

> [!question]- Why does `new Set([{}, {}]).size` equal 2?
> Objects are compared by reference, and each `{}` is a different object.

> [!question]- How does `Set` handle `NaN`?
> It treats `NaN` as equal to itself (SameValueZero), so `new Set([NaN, NaN]).size` is `1`.

> [!question]- Why is `Set.has` faster than `Array.includes`?
> `Set` uses hashing (O(1) average) while `includes` scans the array (O(n)).

> [!question]- Can you `JSON.stringify` a `Map` or `Set`?
> Not directly (both become `{}`). Convert first: `JSON.stringify([...map])` / `[...set]`, or `Object.fromEntries(map)`.

> [!question]- When would you use `WeakMap`?
> To associate data with objects without preventing garbage collection (private data, caches keyed by DOM nodes). Keys must be objects; it's not iterable.

> [!question]- How would you implement an LRU cache?
> A `Map` keeps insertion order: on `get`, delete and re-insert the key to mark it recent; on `set`, if over capacity delete the first key (`map.keys().next().value`). See coding practice.

### Advanced array methods

> [!question]- Explain `reduce`.
> It iterates the array, passing an accumulator and the current item to a callback; the returned value becomes the next accumulator, and the final accumulator is the result. Always provide an initial value.

> [!question]- What happens with `reduce` on an empty array and no initial value?
> `TypeError: Reduce of empty array with no initial value`.

> [!question]- How do you group an array of objects by a property?
> `arr.reduce((groups, x) => { (groups[x.key] ??= []).push(x); return groups; }, {})` (or `Object.groupBy` in newer environments).

> [!question]- What is the difference between `flat` and `flatMap`?
> `flat(depth)` flattens an existing nested array to the given depth. `flatMap(fn)` maps each item and then flattens the result exactly one level, in a single pass.

> [!question]- How can `flatMap` act as a filter?
> Return `[]` to drop an item and `[value]` to keep it: `arr.flatMap(x => cond(x) ? [x] : [])`.

> [!question]- How do you flatten an array of any depth?
> `arr.flat(Infinity)` (or recursion/iteration with an explicit stack).

> [!question]- Why is spreading the accumulator inside `reduce` a performance problem?
> Each iteration copies the whole accumulator, making the loop O(n²). Mutate a local accumulator instead.

> [!question]- When should you avoid `reduce`?
> When a simpler `map`/`filter`/`for...of` is clearer, or when you need early exit (`some`/`every`/`find`/loop with `break`).

---

## 9. Predict the output

```js
// 1
const [a, , b = 5, ...rest] = [1, 2, undefined, 4, 5];
console.log(a, b, rest);

// 2
const { x: y = 10, z = 20 } = { x: undefined, z: null };
console.log(y, z);

// 3
try { const { a: { b } } = { a: undefined }; } catch (e) { console.log(e.name); }

// 4
console.log([..."hi"], { ..."hi" });

// 5
console.log({ ...{ a: 1, b: 2 }, a: 9 }, { a: 9, ...{ a: 1, b: 2 } });

// 6
console.log(0 ?? "d", "" ?? "d", null ?? "d", 0 || "d");

// 7
const user = { profile: null };
console.log(user.profile?.name, user.settings?.theme ?? "light");

// 8
const o = { n: 5 };
try { o.n?.(); } catch (e) { console.log(e.name); }

// 9
const u = undefined;
console.log(u?.b.c.d);
try { (u?.b).c; } catch (e) { console.log(e.name); }

// 10
console.log(new Set([1, "1", 1, NaN, NaN, {}, {}]).size);

// 11
const m = new Map();
m.set(NaN, "a"); m.set({}, "b");
console.log(m.get(NaN), m.get({}));

// 12
console.log([1, [2, [3, [4]]]].flat(), [1, [2, [3, [4]]]].flat(Infinity));

// 13
console.log([1, 2, 3].flatMap(n => [[n]]));

// 14
try { [].reduce((a, b) => a + b); } catch (e) { console.log(e.name); }
console.log([[1, 2], [3]].reduce((acc, arr) => acc.concat(arr), []));

// 15
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q);

// 16
const mm = new Map([["a", 1], ["b", 2]]);
mm.set("a", 9);
console.log([...mm]);

// 17  (counter.js exports `let count = 0` and `inc()` which does count++)
// import { count, inc } from "./counter.js";
console.log(count); inc(); console.log(count);

// 18
console.log([3, 1, 2].reduce((acc, n) => acc + n));   // no initial value
console.log([{ n: 1 }, { n: 2 }].reduce((a, b) => a + b.n));
```

<details>
<summary>Answers</summary>

1. `1 5 [4, 5]` (the `undefined` slot uses the default; the empty slot skips `2`)
2. `10 null` (default only for `undefined`)
3. `TypeError` (cannot destructure `b` from `undefined`)
4. `["h", "i"]` and `{ 0: "h", 1: "i" }`
5. `{ a: 9, b: 2 }` and `{ a: 1, b: 2 }`
6. `0 "" "d" "d"`
7. `undefined "light"`
8. `TypeError` (`5` is not a function; `?.()` only guards nullish)
9. `undefined`, then `TypeError` (parentheses break the short-circuit)
10. `5` (`1`, `"1"`, `NaN`, `{}`, `{}`)
11. `"a" undefined` (`NaN` works as a key; the second `{}` is a different object)
12. `[1, 2, [3, [4]]]` and `[1, 2, 3, 4]`
13. `[[1], [2], [3]]` (only one level flattened)
14. `TypeError`, then `[1, 2, 3]`
15. `2 1`
16. `[["a", 9], ["b", 2]]` (updating a key keeps its position)
17. `0` then `1` (live binding)
18. `6`, then `"[object Object]2"` (the first object becomes the accumulator and `object + 2` coerces to a string)

</details>

---

## 10. Coding practice

**1. Remove duplicates (primitives and by key)**
```js
const unique = arr => [...new Set(arr)];
const uniqueBy = (arr, key) => [...new Map(arr.map(o => [o[key], o])).values()];
```

**2. Set operations**
```js
const union        = (a, b) => new Set([...a, ...b]);
const intersection = (a, b) => new Set([...a].filter(x => b.has(x)));
const difference   = (a, b) => new Set([...a].filter(x => !b.has(x)));
```

**3. Frequency counter with `Map`**
```js
function frequency(items) {
  const counts = new Map();
  for (const item of items) counts.set(item, (counts.get(item) ?? 0) + 1);
  return counts;
}
```

**4. First non-repeating character**
```js
function firstUnique(str) {
  const counts = new Map();
  for (const ch of str) counts.set(ch, (counts.get(ch) ?? 0) + 1);
  for (const [ch, n] of counts) if (n === 1) return ch;   // Map keeps insertion order
  return null;
}
```

**5. Group anagrams**
```js
function groupAnagrams(words) {
  const map = new Map();
  for (const w of words) {
    const key = [...w].sort().join("");
    if (!map.has(key)) map.set(key, []);
    map.get(key).push(w);
  }
  return [...map.values()];
}
groupAnagrams(["eat", "tea", "tan", "ate", "nat"]);  // [["eat","tea","ate"],["tan","nat"]]
```

**6. LRU cache**
```js
class LRUCache {
  constructor(limit) { this.limit = limit; this.map = new Map(); }
  get(key) {
    if (!this.map.has(key)) return undefined;
    const value = this.map.get(key);
    this.map.delete(key);
    this.map.set(key, value);                 // mark as most recent
    return value;
  }
  set(key, value) {
    if (this.map.has(key)) this.map.delete(key);
    else if (this.map.size >= this.limit) this.map.delete(this.map.keys().next().value);  // evict oldest
    this.map.set(key, value);
  }
}
```

**7. Group by / index by / count with `reduce`**
```js
const groupBy = (arr, fn) => arr.reduce((acc, x) => { (acc[fn(x)] ??= []).push(x); return acc; }, {});
const indexBy = (arr, key) => arr.reduce((acc, x) => { acc[x[key]] = x; return acc; }, {});
const countBy = (arr, fn) => arr.reduce((acc, x) => { const k = fn(x); acc[k] = (acc[k] ?? 0) + 1; return acc; }, {});
```

**8. Flatten (built-in and manual)**
```js
const flat1 = arr => arr.flat(Infinity);

function flatten(arr) {
  const out = [];
  const stack = [...arr];
  while (stack.length) {
    const item = stack.pop();
    if (Array.isArray(item)) stack.push(...item);
    else out.push(item);
  }
  return out.reverse();
}
```

**9. Safe nested getter with `?.` and `??`**
```js
const get = (obj, path, fallback) =>
  path.split(".").reduce((o, k) => o?.[k], obj) ?? fallback;

get({ a: { b: [10, 20] } }, "a.b.1", 0);   // 20
get({}, "a.b.c", "none");                  // "none"
```

**10. Omit / pick with destructuring and rest**
```js
const { password, ...safeUser } = user;                 // omit
const pick = (obj, keys) => Object.fromEntries(keys.filter(k => k in obj).map(k => [k, obj[k]]));
```

**11. Merge config with defaults (one level deep)**
```js
const withDefaults = (userConfig = {}) => ({
  ...defaults,
  ...userConfig,
  theme: { ...defaults.theme, ...userConfig.theme },
});
```

**12. Pipeline with `reduce`**
```js
const pipe = (...fns) => input => fns.reduce((value, fn) => fn(value), input);
pipe(s => s.trim(), s => s.toLowerCase(), s => s.replaceAll(" ", "-"))("  Hello World ");  // "hello-world"
```

**13. Expand and filter in one pass with `flatMap`**
```js
const words = lines => lines.flatMap(line => line.split(/\s+/)).filter(Boolean);
const evensDoubled = nums => nums.flatMap(n => (n % 2 === 0 ? [n * 2] : []));
```

**14. Module example: lazy-load a heavy feature**
```js
// utils/format.js
export const formatPrice = n => new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" }).format(n);
export default function slugify(s) { return s.toLowerCase().trim().replace(/\s+/g, "-"); }

// main.js
import slugify, { formatPrice } from "./utils/format.js";

document.querySelector("#report").addEventListener("click", async () => {
  const { buildReport } = await import("./report.js");   // loaded only when needed
  buildReport();
});
```

---

## 11. Quick-recall cheat sheet

| Need | Use |
|---|---|
| Pull fields from an object | `const { a, b: renamed, c = 1 } = obj` |
| Pull items from an array | `const [first, , third, ...rest] = arr` |
| Swap two values | `[a, b] = [b, a]` |
| Destructure safely | Default `= {}`, or `?.` / `?? {}` |
| Copy or merge | `[...a, ...b]` / `{ ...a, ...b }` (shallow; later wins) |
| Variadic function | `function f(...args)` |
| Omit a key immutably | `const { key, ...rest } = obj` |
| Safe nested read | `a?.b?.[0]?.c` |
| Optional method/callback | `obj.method?.()` / `cb?.(x)` |
| Default that keeps `0`/`""`/`false` | `value ?? fallback` |
| Default only if missing key/falsy | `value \|\| fallback` |
| Lazy init | `cache[k] ??= compute(k)` |
| Export / import | `export const x` · `export default fn` · `import fn, { x } from "./f.js"` |
| Lazy load a module | `await import("./heavy.js")` |
| Unique values | `[...new Set(arr)]` |
| Fast membership test | `set.has(x)` |
| Non-string / object keys | `Map` |
| Convert Map ↔ object | `Object.fromEntries(map)` / `new Map(Object.entries(obj))` |
| Fold to one value | `arr.reduce((acc, x) => ..., initial)` |
| Group / count / index | `reduce` with `??=` on the accumulator |
| Flatten nested arrays | `arr.flat(depth)` / `flat(Infinity)` |
| Map then flatten one level / filter+map | `arr.flatMap(fn)` (return `[]` to drop) |

---

## Related notes
- [[JS 03 - Arrays & Objects]]
- [[JS 06 - Functional Concepts]]
- [[JS 08 - Async JavaScript]]
- [[JS 14 - Advanced Language Features]]
