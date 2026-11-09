# JS 03 — Arrays & Objects

> [!info] Scope
> Array methods (`map`, `filter`, `find`, `some`, `every`, `forEach`) · Sorting · Object basics · `Object.keys / values / entries` · JSON
> Previous: [[JS 02 - Functions]] · Next: [[JS 04 - DOM & Events Basics]]
> Deeper array work (`reduce`, `flat`, `flatMap`, immutable updates) is in [[JS 07 - ES6+ Features]].

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| `map` | Returns a **new array of the same length**. Always `return` inside a block body. |
| `filter` | Returns a new array of items where the callback is truthy (maybe empty, never `undefined`). |
| `find` | Returns the **first matching item** or `undefined`. `findIndex` returns `-1` when missing. |
| `some` / `every` | `some([])` is `false`, `every([])` is `true`. Both short-circuit. |
| `forEach` | Returns `undefined`. Can't `break`, doesn't wait for `await`. |
| `sort` | **Mutates**, and sorts as **strings** by default. Always pass a comparator for numbers. |
| Objects | Keys are strings or symbols. Integer-like keys come first in order. `{} === {}` is `false`. |
| `Object.keys/values/entries` | Own, enumerable properties only. `Object.fromEntries` reverses `entries`. |
| Copying | Spread / `Object.assign` are **shallow**. `structuredClone` is deep. |
| JSON | Drops `undefined`, functions and symbols. Dates become strings. Circular refs throw. |

---

## 1. Array basics (needed for the rest)

### Definition
An **array** is an ordered, zero-indexed, resizable list of values. Arrays are objects (`typeof [] === "object"`), so use `Array.isArray(x)` to check them.

### Summary

```js
const nums = [10, 20, 30];
nums[0];            // 10
nums.at(-1);        // 30 (negative index)
nums.length;        // 3
Array.isArray(nums); // true

Array.from({ length: 3 }, (_, i) => i);  // [0, 1, 2]
Array(3).fill(0);                         // [0, 0, 0]
```

**Mutating vs non-mutating methods**

| Mutates the original | Returns a new array / value |
|---|---|
| `push`, `pop`, `shift`, `unshift` | `map`, `filter`, `slice`, `concat` |
| `splice`, `sort`, `reverse`, `fill` | `toSorted`, `toReversed`, `toSpliced`, `with` (ES2023) |
| `arr.length = 0` (clears the array) | `find`, `some`, `every`, `includes`, `indexOf` |

```js
const a = [1, 2, 3];
a.push(4);                // [1, 2, 3, 4]  (returns new length: 4)
a.slice(1, 3);            // [2, 3]  (original untouched)
a.splice(1, 2);           // removes 2 items at index 1, returns [2, 3], mutates a
```

### Edge cases
- **Arrays are copied by reference:** `const b = a; b.push(1)` also changes `a`. Copy with `[...a]` or `a.slice()`.
- **`const` arrays can still be mutated.** `const` blocks reassignment only.
- **`[] === []` is `false`** (different references). Compare contents manually.
- **Holes:** `Array(3)` creates 3 empty slots, not `undefined` values. `map`/`forEach` skip holes. Use `Array(3).fill(0)` or `Array.from`.
- **`Array(3).fill({})` shares one object** across all slots (same reference).
- **`delete arr[1]`** leaves a hole and doesn't change `length`. Use `splice`.
- **`length` is writable:** `arr.length = 2` truncates the array.
- `indexOf` uses `===` (can't find `NaN`). `includes` uses SameValueZero (can find `NaN`).
- `splice` vs `slice`: `splice` mutates and removes/inserts, `slice` copies a portion.

### Use cases
- Lists of items from an API (products, users, messages).
- Stack (`push`/`pop`) and queue (`push`/`shift`) behavior.
- Holding UI state as a list (todos, cart items), usually updated **immutably** in React.

---

## 2. Array iteration methods

### Definition
Higher-order array methods that take a **callback** `(item, index, array) => ...` and apply it to each element, without writing manual loops.

### Summary

| Method | Returns | Use it to | Mutates? |
|---|---|---|---|
| `forEach(fn)` | `undefined` | Run side effects per item | No |
| `map(fn)` | New array (same length) | Transform every item | No |
| `filter(fn)` | New array (0..n items) | Keep items that pass a test | No |
| `find(fn)` | First matching item or `undefined` | Look up one item | No |
| `findIndex(fn)` | Index or `-1` | Position of first match | No |
| `findLast(fn)` / `findLastIndex(fn)` | Last match / its index | Search from the end | No |
| `some(fn)` | `true` / `false` | Does at least one item pass? | No |
| `every(fn)` | `true` / `false` | Do all items pass? | No |
| `includes(x)` | `true` / `false` | Is a primitive in the array? | No |

```js
const users = [
  { id: 1, name: "Asha", age: 22, active: true },
  { id: 2, name: "Ravi", age: 17, active: false },
  { id: 3, name: "Meera", age: 30, active: true },
];

users.map(u => u.name);                 // ["Asha", "Ravi", "Meera"]
users.filter(u => u.active);            // [Asha, Meera]
users.find(u => u.id === 2);            // { id: 2, ... }
users.find(u => u.id === 99);           // undefined
users.findIndex(u => u.age > 25);       // 2
users.some(u => u.age < 18);            // true
users.every(u => u.active);             // false
users.forEach(u => console.log(u.name));// returns undefined
```

**Chaining**
```js
const activeNames = users
  .filter(u => u.active)
  .map(u => u.name.toUpperCase());      // ["ASHA", "MEERA"]
```

**Choosing the right one**

| I want to... | Use |
|---|---|
| Convert each item to something else | `map` |
| Remove some items | `filter` |
| Get one item by condition | `find` |
| Check a condition across items | `some` / `every` |
| Just run code (log, DOM update) | `forEach` or `for...of` |
| Stop early or use `await` in order | `for...of` |

### Edge cases
- **`map` with a block body needs `return`:** `arr.map(x => { x * 2 })` gives `[undefined, ...]`.
- **`map` for side effects only is a misuse.** Use `forEach` (or `for...of`) when you don't need the result.
- **`forEach` can't be stopped** with `break` or `return`. Use `for...of`, `some`, or `every`.
- **`forEach` ignores promises:** `arr.forEach(async x => await save(x))` doesn't wait. Use `for...of` with `await`, or `Promise.all(arr.map(...))`.
- **Empty arrays:** `[].some(fn)` → `false`, `[].every(fn)` → `true` (vacuous truth), `[].find(fn)` → `undefined`.
- **`find` returns the item, not a boolean.** A found value like `0` or `""` is falsy, so check `!== undefined` or use `findIndex`/`some`.
- **`filter(Boolean)`** removes all falsy values (`0`, `""`, `null`, `undefined`, `NaN`, `false`), including legitimate `0`s.
- **Callback arguments:** `["1","2","3"].map(parseInt)` → `[1, NaN, NaN]` (index passed as radix). Use `.map(Number)`.
- **Objects inside arrays are shared by reference:** `map(u => u)` or `filter` returns the *same* objects. Mutating one changes the original.
- **Mutating while iterating** (push/splice inside `forEach`) causes skipped or repeated items.
- **Sparse arrays:** `forEach`, `map`, `filter` skip holes.
- **Methods short-circuit:** `some` stops at the first `true`, `every` at the first `false`, `find` at the first match. `map`, `filter`, `forEach` always visit every item.

### Use cases
- `map`: turn API data into UI props or dropdown options; render lists in React.
- `filter`: search, tabs, visible items, removing an item by id (`items.filter(i => i.id !== id)`).
- `find`: look up the selected record by id.
- `some` / `every`: validation (`fields.every(f => f.valid)`), permission checks, "any item selected?".

---

## 3. Sorting

### Definition
`Array.prototype.sort()` sorts the array **in place** and returns the same array. Without a comparator it converts items to strings and compares UTF-16 code unit order.

### Summary

```js
[10, 9, 1].sort();                // [1, 10, 9]   (string order!)
[10, 9, 1].sort((a, b) => a - b); // [1, 9, 10]   ascending
[10, 9, 1].sort((a, b) => b - a); // [10, 9, 1]   descending
```

**Comparator rule:** `compare(a, b)` returns
- negative → `a` goes before `b`
- positive → `a` goes after `b`
- `0` → keep order (sort is **stable** since ES2019)

| Sort what | Comparator |
|---|---|
| Numbers ascending | `(a, b) => a - b` |
| Numbers descending | `(a, b) => b - a` |
| Strings (human order) | `(a, b) => a.localeCompare(b)` |
| Objects by number field | `(a, b) => a.age - b.age` |
| Objects by string field | `(a, b) => a.name.localeCompare(b.name)` |
| Multiple fields | `(a, b) => a.role.localeCompare(b.role) \|\| a.age - b.age` |
| Dates | `(a, b) => new Date(a.date) - new Date(b.date)` |

**Non-mutating sorts**
```js
const sorted = [...arr].sort((a, b) => a - b);  // copy, then sort
const sorted2 = arr.toSorted((a, b) => a - b);  // ES2023: returns a new array
```

### Edge cases
- **Default sort is lexicographic:** `[1, 2, 10].sort()` → `[1, 10, 2]`. `["b", "a", "C"].sort()` → `["C", "a", "b"]` (uppercase first).
- **`sort` mutates the original.** In React state, copy first or you'll mutate state directly.
- **Comparator must return a number.** `(a, b) => a > b` (boolean) is unreliable. It returns `true`/`false` (1/0), never negative.
- `undefined` values are always moved to the end; holes are removed from the sort order.
- **`reverse()` also mutates.** Use `toReversed()` or `[...arr].reverse()`.
- **`a - b` can misbehave** with `NaN`, `Infinity` pairs, or non-numeric values.
- **Accents/case:** use `localeCompare` (optionally with `"en", { sensitivity: "base" }`) for user-visible text.
- Sorting dates as strings works only for ISO format (`YYYY-MM-DD`).

### Use cases
- Leaderboards, price low→high, newest first, alphabetical lists, multi-column table sorting.

---

## 4. Object basics

### Definition
An **object** is a collection of **key–value pairs** (properties). Keys are strings or symbols (numbers are converted to strings); values can be anything, including functions (methods) and other objects.

### Summary

```js
const user = {
  name: "Arman",
  "favorite-color": "blue",   // quoted key (not a valid identifier)
  skills: ["JS", "React"],
  address: { city: "Kolkata" },
  greet() { return `Hi, ${this.name}`; },  // method shorthand
};

user.name;                 // dot notation
user["favorite-color"];    // bracket notation (needed for special chars / dynamic keys)
const key = "name";
user[key];                 // dynamic key
user.address?.city;        // optional chaining
user.age = 22;             // add
user.name = "A";           // update
delete user.age;           // remove
```

**Shorthand & computed keys**
```js
const name = "Asha", age = 22;
const person = { name, age };             // { name: "Asha", age: 22 }
const field = "email";
const form = { [field]: "a@b.com" };      // { email: "a@b.com" }
```

**Checking for a property**

| Check | Result for own prop `a: undefined` | Looks at prototype chain? |
|---|---|---|
| `obj.a !== undefined` | `false` (wrong: property exists) | Yes |
| `"a" in obj` | `true` | Yes |
| `obj.hasOwnProperty("a")` | `true` | No |
| `Object.hasOwn(obj, "a")` | `true` (preferred, ES2022) | No |

**Copying**

| Method | Depth | Notes |
|---|---|---|
| `{ ...obj }` | Shallow | Later keys override earlier ones |
| `Object.assign({}, obj)` | Shallow | Mutates the first argument |
| `structuredClone(obj)` | Deep | Handles Dates, Maps, Sets, circular refs; **not** functions |
| `JSON.parse(JSON.stringify(obj))` | Deep (lossy) | See JSON section |

```js
const a = { x: 1, nested: { y: 2 } };
const b = { ...a };
b.nested.y = 99;
a.nested.y;          // 99 (shallow copy shares nested objects)

const c = structuredClone(a);   // fully independent
```

### Edge cases
- **Key order:** integer-like keys come first (ascending), then string keys in insertion order.
  ```js
  Object.keys({ b: 1, 2: "x", a: 2, 1: "y" });  // ["1", "2", "b", "a"]
  ```
- **Keys are strings:** `obj[1]` and `obj["1"]` are the same property. Using an object as a key gives `"[object Object]"`. Use a `Map` for non-string keys.
- **`{} === {}` is `false`.** Objects compare by reference. Deep comparison needs custom code or a library.
- **Missing property** returns `undefined` (no error), but reading a property of `undefined`/`null` throws `TypeError`. Use `?.`.
- **`const` objects are mutable.** `Object.freeze(obj)` prevents changes (shallow). `Object.isFrozen(obj)` checks.
- **Dynamic keys from user input:** a key like `__proto__` can cause prototype pollution. Use `Map` or `Object.create(null)` for user-keyed data.
- **`this` in arrow-function methods is not the object** (see [[JS 02 - Functions]]).
- **Spread order matters:** `{ ...defaults, ...overrides }` — later wins.
- **Inherited properties:** `for...in` and `in` include prototype properties; `Object.keys` doesn't.
- **Getters/setters** (`get fullName() {}`) look like properties but run code.
- **Deleting** a property doesn't free its value if other references exist.

### Use cases
- Representing entities (user, product, config, form state).
- Lookup tables / dictionaries (`const statusLabel = { 1: "Open", 2: "Closed" }`).
- Options objects for function arguments.
- Counting/grouping by key (`counts[word] = (counts[word] ?? 0) + 1`).

---

## 5. `Object.keys`, `Object.values`, `Object.entries`

### Definition
Static methods that convert an object's **own, enumerable, string-keyed** properties into arrays so you can use array methods and loops on them.

### Summary

```js
const prices = { apple: 40, banana: 10, mango: 80 };

Object.keys(prices);     // ["apple", "banana", "mango"]
Object.values(prices);   // [40, 10, 80]
Object.entries(prices);  // [["apple", 40], ["banana", 10], ["mango", 80]]
```

| Method | Returns | Typical use |
|---|---|---|
| `Object.keys(obj)` | Array of keys | Count keys, check emptiness, iterate keys |
| `Object.values(obj)` | Array of values | Sum / max of values |
| `Object.entries(obj)` | Array of `[key, value]` | Loop with destructuring, transform objects |
| `Object.fromEntries(entries)` | Object | Inverse of `entries`; rebuild an object |

**Common patterns**
```js
Object.keys(obj).length === 0;                       // is the object empty?

for (const [key, value] of Object.entries(prices)) {
  console.log(key, value);
}

// Transform values (double all prices)
Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);  // { apple: 80, banana: 20, mango: 160 }

// Filter an object
Object.fromEntries(
  Object.entries(prices).filter(([, v]) => v > 20)
);  // { apple: 40, mango: 80 }

// Sum of values
Object.values(prices).reduce((a, b) => a + b, 0);    // 130

// Invert keys and values
Object.fromEntries(Object.entries({ a: 1, b: 2 }).map(([k, v]) => [v, k]));  // { 1: "a", 2: "b" }
```

### Edge cases
- Only **own enumerable** properties: inherited ones and non-enumerable ones are skipped. Symbol keys are skipped (use `Object.getOwnPropertySymbols`).
- **Order** follows the key-order rule (integer keys first, then insertion order).
- On **arrays** they work on indexes: `Object.keys(["a", "b"])` → `["0", "1"]`. On strings: `Object.keys("hi")` → `["0", "1"]`.
- `Object.keys(null)` / `Object.keys(undefined)` throws `TypeError`.
- **Keys are always strings**, even if the original key looked like a number.
- `Object.entries` returns **copies of the pairs**, so changing the returned array doesn't change the object.
- `Object.assign(target, source)` **mutates** `target`. Use `{ ...a, ...b }` for a new object.
- `for...in` vs `Object.keys`: `for...in` also walks inherited enumerable properties.

### Use cases
- Rendering an object as a list (`Object.entries(errors).map(...)` for form errors).
- Counting items per category, building lookup maps, reshaping API responses.
- Checking empty state (`Object.keys(filters).length`).

---

## 6. JSON

### Definition
**JSON (JavaScript Object Notation)** is a text format for exchanging data. `JSON.stringify` converts a JS value to a JSON string; `JSON.parse` converts a JSON string back to a value.

### Summary

```js
const user = { name: "Arman", skills: ["JS", "React"], active: true };

const text = JSON.stringify(user);
// '{"name":"Arman","skills":["JS","React"],"active":true}'

JSON.parse(text);                      // back to an object
JSON.stringify(user, null, 2);         // pretty-printed with 2-space indent
JSON.stringify(user, ["name"]);        // only listed keys: '{"name":"Arman"}'
JSON.stringify(user, (k, v) => (k === "active" ? undefined : v));  // replacer function
JSON.parse(text, (k, v) => (k === "createdAt" ? new Date(v) : v)); // reviver
```

**JSON rules (stricter than JS objects)**
- Keys must be in **double quotes**. Strings use **double quotes**.
- No trailing commas, no comments, no `undefined`, no functions.
- Allowed values: object, array, string, number, `true`, `false`, `null`.

**What `JSON.stringify` does to special values**

| Input | Output |
|---|---|
| `undefined`, function, symbol as an **object property** | Property is **omitted** |
| `undefined`, function, symbol **inside an array** | `null` |
| `NaN`, `Infinity`, `-Infinity` | `null` |
| `Date` | ISO string (via `toJSON`) |
| `Map`, `Set` | `{}` (data lost) |
| `BigInt` | **Throws** `TypeError` |
| Circular reference | **Throws** `TypeError` |
| Top-level `undefined` | Returns `undefined` (not a string) |

```js
JSON.stringify({ a: undefined, b: () => {}, c: NaN, d: new Date(0), e: [undefined] });
// '{"c":null,"d":"1970-01-01T00:00:00.000Z","e":[null]}'
```

**Safe parse helper**
```js
function safeParse(text, fallback = null) {
  try { return JSON.parse(text); }
  catch { return fallback; }
}
```

### Edge cases
- **`JSON.parse` throws `SyntaxError`** on invalid input (single quotes, trailing comma, `undefined`, empty string). Always wrap untrusted input in `try/catch`.
- **`JSON.parse(undefined)`** throws because it converts to the string `"undefined"`. `JSON.parse(null)` returns `null`.
- **Dates don't round-trip:** `parse(stringify(date))` gives a string. Revive manually.
- **`JSON.parse(JSON.stringify(obj))` as a deep clone** loses `undefined`, functions, `Map`/`Set`, turns Dates into strings, and fails on circular refs and BigInt. Prefer `structuredClone`.
- **Key order and duplicate keys:** later duplicate keys win in `parse`.
- **`toJSON()` method** on an object customizes its serialization.
- **Large numbers:** integers beyond `2^53` lose precision when parsed.
- `JSON.stringify("a")` → `'"a"'` (a string with quotes), `JSON.stringify(5)` → `"5"`.
- **Never `eval` JSON.** Use `JSON.parse`.
- `localStorage` stores only strings, so objects need `stringify` on save and `parse` on read.

### Use cases
- API requests/responses (`fetch` → `res.json()`; body: `JSON.stringify(data)`).
- Saving state to `localStorage` / `sessionStorage`.
- Config files (`package.json`, `tsconfig.json`), logging, copying plain data.
- Quick pretty-print for debugging: `console.log(JSON.stringify(obj, null, 2))`.

---

## 7. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer.

### Basic

> [!question]- What is the difference between `map` and `forEach`?
> `map` returns a **new array** with each callback result and is used to transform data. `forEach` returns `undefined` and is used for side effects. Neither mutates the original array (unless your callback does).

> [!question]- What is the difference between `map` and `filter`?
> `map` transforms every element (same length output). `filter` keeps only elements for which the callback returns truthy (output length 0..n).

> [!question]- Difference between `find` and `filter`?
> `find` returns the **first matching element** (or `undefined`) and stops. `filter` returns an **array** of all matches (empty if none).

> [!question]- What do `some` and `every` do?
> `some` returns `true` if at least one element passes; `every` returns `true` only if all pass. Both short-circuit. On an empty array, `some` is `false` and `every` is `true`.

> [!question]- Which array methods mutate the original?
> `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`, `fill`, `copyWithin`. Non-mutating: `map`, `filter`, `slice`, `concat`, plus the ES2023 `toSorted`, `toReversed`, `toSpliced`, `with`.

> [!question]- How do you check whether a value is an array?
> `Array.isArray(value)`. `typeof` returns `"object"`.

> [!question]- How do you check if an object is empty?
> `Object.keys(obj).length === 0` (also check `obj` isn't `null`).

> [!question]- What is the difference between `Object.keys`, `Object.values` and `Object.entries`?
> They return arrays of an object's own enumerable keys, values, or `[key, value]` pairs respectively.

> [!question]- What does `JSON.stringify` / `JSON.parse` do?
> `stringify` converts a JS value to a JSON string; `parse` converts a JSON string back to a JS value.

> [!question]- Difference between dot notation and bracket notation?
> Dot notation needs a valid identifier and a fixed key (`obj.name`). Bracket notation accepts any string, including dynamic keys and keys with spaces/hyphens (`obj[key]`).

### Intermediate

> [!question]- Why does `[10, 9, 1].sort()` give `[1, 10, 9]`?
> Default `sort` converts items to strings and compares them lexicographically, so `"10"` comes before `"9"`. Pass a comparator: `(a, b) => a - b`.

> [!question]- Does `sort` mutate the array? How do you avoid that?
> Yes, it sorts in place and returns the same array. Copy first (`[...arr].sort(...)`) or use `toSorted()`.

> [!question]- How does the sort comparator work?
> It returns a negative number if `a` should come first, positive if `b` should come first, and `0` if equal. The sort is stable (since ES2019), so equal items keep their order.

> [!question]- Shallow copy vs deep copy?
> A shallow copy (`{...obj}`, `[...arr]`, `Object.assign`) copies top-level properties, but nested objects stay shared. A deep copy duplicates everything; use `structuredClone`.

> [!question]- How do you copy an object or array without mutating the original?
> Spread: `{ ...obj }` and `[...arr]`. For nested data use `structuredClone`, or update each level immutably.

> [!question]- `in` vs `hasOwnProperty` vs `Object.hasOwn`?
> `in` checks the object and its prototype chain. `hasOwnProperty` and `Object.hasOwn` check only own properties; `Object.hasOwn` is the safer modern form.

> [!question]- How do you remove an item from an array immutably?
> `arr.filter(item => item.id !== id)`. To remove by index: `arr.filter((_, i) => i !== index)`.

> [!question]- How do you update an item in an array immutably?
> `arr.map(item => item.id === id ? { ...item, done: true } : item)`.

> [!question]- Why doesn't `forEach` work with `async/await` the way you'd expect?
> `forEach` ignores returned promises, so it doesn't wait. Use `for...of` with `await` for sequential execution, or `await Promise.all(arr.map(async ...))` for parallel.

> [!question]- What are the limitations of `JSON.parse(JSON.stringify(obj))` for cloning?
> It drops `undefined`, functions and symbols, converts Dates to strings, turns `NaN`/`Infinity` into `null`, loses `Map`/`Set`, and throws on circular references and BigInt.

> [!question]- Can an object key be a number or an object?
> Numbers are converted to strings (`obj[1]` is `obj["1"]`). Objects become `"[object Object]"`. Use a `Map` when you need non-string keys.

> [!question]- How are objects compared?
> By reference. `{} === {}` is `false`; two variables are equal only if they point to the same object. Deep equality needs a custom function or library.

### Tricky / follow-up

> [!question]- Why does `[1,2,3].map(x => { x * 2 })` return `[undefined, undefined, undefined]`?
> The block body has no `return`. Use `x => x * 2` or add `return`.

> [!question]- Why does `["1","2","3"].map(parseInt)` return `[1, NaN, NaN]`?
> `map` passes `(value, index)`, and `parseInt`'s second argument is the radix. Use `.map(Number)`.

> [!question]- What's the output of `[].every(x => false)` and `[].some(x => true)`?
> `true` and `false`. `every` on an empty array is vacuously true; `some` has nothing to match.

> [!question]- Why is `Array(3).fill([])` a bug-prone pattern?
> All three slots point to the **same array**. Pushing to one changes all. Use `Array.from({ length: 3 }, () => [])`.

> [!question]- What happens to key order when an object has numeric-like keys?
> Integer-like keys come first in ascending order, then string keys in insertion order.

> [!question]- What does `JSON.stringify({ a: undefined, b: [undefined] })` return?
> `'{"b":[null]}'`. `undefined` properties are omitted, but inside arrays they become `null`.

> [!question]- `Object.freeze` — is it deep?
> No. It prevents adding, removing and changing top-level properties only. Nested objects stay mutable unless frozen recursively.

> [!question]- `includes` vs `indexOf` for `NaN`?
> `[NaN].includes(NaN)` is `true` (SameValueZero). `[NaN].indexOf(NaN)` is `-1` (`===`).

---

## 8. Predict the output

```js
// 1
console.log([10, 9, 1].sort());
// 2
console.log([1, 2, 3].map(x => { x * 2 }));
// 3
console.log([1, 2, 3, 4].filter(x => x % 2));
// 4
console.log([].every(x => false), [].some(x => true));
// 5
console.log([NaN].includes(NaN), [NaN].indexOf(NaN));
// 6
const a = [1, 2, 3]; const b = a; b.push(4);
console.log(a.length);
// 7
console.log(JSON.stringify({ a: undefined, b: () => {}, c: NaN, d: new Date(0) }));
// 8
console.log(Object.keys({ b: 1, 2: "x", a: 2, 1: "y" }));
// 9
console.log([1, 2, 3].forEach(x => x));
// 10
const arr = Array(3).fill([]); arr[0].push(1);
console.log(arr);
// 11
console.log(typeof JSON.stringify(undefined));
// 12
console.log({} === {}, [] == [], [] + []);
// 13
const x = { n: { v: 1 } }; const y = { ...x }; y.n.v = 2;
console.log(x.n.v);
// 14
console.log(["b", "a", "C"].sort());
```

<details>
<summary>Answers</summary>

1. `[1, 10, 9]`
2. `[undefined, undefined, undefined]`
3. `[1, 3]`
4. `true false`
5. `true -1`
6. `4` (same array)
7. `{"c":null,"d":"1970-01-01T00:00:00.000Z"}`
8. `["1", "2", "b", "a"]`
9. `undefined`
10. `[[1], [1], [1]]` (same array in every slot)
11. `"undefined"`
12. `false false ""`
13. `2` (shallow copy shares the nested object)
14. `["C", "a", "b"]` (uppercase sorts before lowercase)

</details>

---

## 9. Coding practice

**1. Remove duplicates**
```js
const unique = (arr) => [...new Set(arr)];

// By key (objects)
const uniqueBy = (arr, key) => [...new Map(arr.map(o => [o[key], o])).values()];
```

**2. Count frequency**
```js
function frequency(arr) {
  const counts = {};
  for (const item of arr) counts[item] = (counts[item] ?? 0) + 1;
  return counts;
}
frequency(["a", "b", "a"]);  // { a: 2, b: 1 }
```

**3. Group by a key**
```js
function groupBy(arr, keyFn) {
  const out = {};
  for (const item of arr) {
    const k = keyFn(item);
    (out[k] ??= []).push(item);
  }
  return out;
}
groupBy(users, u => u.active);  // { true: [...], false: [...] }
// Newer built-in: Object.groupBy(users, u => u.active) (check browser support)
```

**4. Sort objects by multiple fields**
```js
people.sort((a, b) => a.city.localeCompare(b.city) || a.age - b.age);
```

**5. Two Sum (return indexes)**
```js
function twoSum(nums, target) {
  const seen = new Map();
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return null;
}
```

**6. Chunk an array**
```js
const chunk = (arr, size) =>
  Array.from({ length: Math.ceil(arr.length / size) }, (_, i) =>
    arr.slice(i * size, i * size + size));
chunk([1, 2, 3, 4, 5], 2);  // [[1,2],[3,4],[5]]
```

**7. Intersection of two arrays**
```js
const intersect = (a, b) => { const s = new Set(b); return a.filter(x => s.has(x)); };
```

**8. Pick / omit keys**
```js
const pick = (obj, keys) =>
  Object.fromEntries(keys.filter(k => k in obj).map(k => [k, obj[k]]));
const omit = (obj, keys) =>
  Object.fromEntries(Object.entries(obj).filter(([k]) => !keys.includes(k)));
```

**9. Immutable update of an item in a list**
```js
const toggleTodo = (todos, id) =>
  todos.map(t => (t.id === id ? { ...t, done: !t.done } : t));
```

**10. Safe JSON parse with a fallback**
```js
function safeParse(text, fallback = null) {
  try { return JSON.parse(text); } catch { return fallback; }
}
```

**11. Transform object values**
```js
const mapValues = (obj, fn) =>
  Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, fn(v, k)]));
mapValues({ a: 1, b: 2 }, v => v * 10);  // { a: 10, b: 20 }
```

---

## 10. Quick-recall cheat sheet

| Need | Use |
|---|---|
| Transform every item | `arr.map(fn)` |
| Keep matching items | `arr.filter(fn)` |
| First match / its index | `arr.find(fn)` / `arr.findIndex(fn)` |
| Any / all match | `arr.some(fn)` / `arr.every(fn)` |
| Loop with `break` or `await` | `for...of` |
| Sort numbers | `arr.sort((a, b) => a - b)` |
| Sort without mutating | `[...arr].sort(...)` / `arr.toSorted(...)` |
| Sort text properly | `a.localeCompare(b)` |
| Copy array / object | `[...arr]` / `{ ...obj }` |
| Deep copy | `structuredClone(x)` |
| Remove by id (immutable) | `arr.filter(i => i.id !== id)` |
| Update by id (immutable) | `arr.map(i => i.id === id ? {...i, ...changes} : i)` |
| Own-property check | `Object.hasOwn(obj, key)` |
| Empty object check | `Object.keys(obj).length === 0` |
| Object → pairs → object | `Object.entries` → `Object.fromEntries` |
| Object to string / back | `JSON.stringify` / `JSON.parse` (in `try/catch`) |
| Pretty-print | `JSON.stringify(obj, null, 2)` |

---

## Related notes
- [[JS 02 - Functions]]
- [[JS 04 - DOM & Events Basics]]
- [[JS 07 - ES6+ Features]]
- [[JS 05 - Core Concepts (Scope, Closures, this)]]
