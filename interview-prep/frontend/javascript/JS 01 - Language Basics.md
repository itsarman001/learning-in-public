# 01 — Language Basics Javascript

> [!info] Scope Variables · Data types · Operators · Type coercion · Conditionals · Loops · Strings · Numbers · Dates Next note: [[JS 02 - Functions]]

---

## 0. One-page summary

|Topic|The one thing to remember|
|---|---|
|Variables|`const` by default, `let` when reassigning, never `var`. `const` blocks reassignment, not mutation.|
|Data types|7 primitives + object. `typeof null === "object"` is a bug. Use `Array.isArray` for arrays.|
|Operators|Use `===`. `??` only falls back on `null`/`undefined`; `\|` falls back on any falsy value.|
|Coercion|`+` concatenates if either side is a string; other math operators convert to numbers.|
|Conditionals|`switch` uses `===`. `[]` and `{}` are truthy. Use guard clauses.|
|Loops|`for...of` for values, `for...in` for object keys. `forEach` can't `break` or `await`.|
|Strings|Immutable. Methods return new strings. `length` counts UTF-16 units, not characters.|
|Numbers|64-bit floats. `0.1 + 0.2 !== 0.3`. `toFixed` returns a string.|
|Dates|Months are 0-indexed. Date-only ISO strings parse as UTC. Dates are mutable.|

---

## 1. Variables

### Definition

A **variable** is a named binding to a value in memory. JavaScript has three declaration keywords: `var` (legacy), `let` and `const` (ES6, 2015).

### Summary

| |`var`|`let`|`const`|
|---|---|---|---|
|Scope|Function|Block|Block|
|Reassign|Yes|Yes|No|
|Redeclare in same scope|Yes|No (SyntaxError)|No (SyntaxError)|
|Must initialize at declaration|No|No|**Yes**|
|Hoisted|Yes, initialized to `undefined`|Yes, but in the TDZ|Yes, but in the TDZ|
|Becomes property of `window` (classic scripts)|Yes|No|No|

**Hoisting:** declarations are registered before code runs. `var` is usable (as `undefined`) before its line. `let`/`const` exist but throw `ReferenceError` if accessed before their line. That gap is the **Temporal Dead Zone (TDZ)**.

### Examples

```js
console.log(a); // undefined  (var hoisted)
var a = 1;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 2;

if (true) {
  var leaks = "visible outside";
  let stays = "block only";
}
console.log(leaks); // "visible outside"
console.log(stays); // ReferenceError
```

```js
const user = { name: "A" };
user.name = "B";      // allowed: mutating the object
user = {};            // TypeError: Assignment to constant variable
```

### Edge cases

- **`const` ≠ immutable.** It freezes the binding, not the value. `Object.freeze(obj)` blocks property changes but is **shallow** (nested objects stay mutable).
- **Loop closure bug**
    
    ```js
    for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));
    // 3 3 3for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
    ```
    
    `var` shares one binding; `let` creates a fresh binding per iteration.
- **Implicit globals:** `x = 5` (no keyword) creates a global in sloppy mode, throws `ReferenceError` in strict mode and ES modules.
- **Chained assignment trap:** `let a = b = 5;` declares only `a`; `b` becomes an implicit global.
- **`typeof` and the TDZ:** `typeof undeclared` gives `"undefined"` (no error), but `typeof x` before `let x` **throws**.
- **`const` in a classic `for` loop** fails on `i++` (TypeError). `for (const x of arr)` is fine because each iteration has a new binding.
- **`let`/`const` inside `switch` cases** share one block scope. Wrap each case in `{ }` to avoid redeclaration errors.
- **Shadowing:** an inner `let x` hides an outer `x`. Legal, but a common source of bugs.

### Use cases

- `const` for imports, config, functions, DOM references, objects and arrays you only mutate.
- `let` for counters, accumulators, values reassigned in loops or branches.
- `var` only when reading old code.

---

## 2. Data types

### Definition

JavaScript is **dynamically typed**: variables have no fixed type; values do. There are **7 primitive types** and **1 non-primitive type** (object).

### Summary

| Type      | Example                             | `typeof`                    |
| --------- | ----------------------------------- | --------------------------- |
| string    | `"hi"`                              | `"string"`                  |
| number    | `42`, `3.14`, `NaN`, `Infinity`     | `"number"`                  |
| boolean   | `true`                              | `"boolean"`                 |
| undefined | `let x;`                            | `"undefined"`               |
| null      | `null`                              | `"object"` (historical bug) |
| bigint    | `10n`                               | `"bigint"`                  |
| symbol    | `Symbol("id")`                      | `"symbol"`                  |
| object    | `{}`, `[]`, `new Date()`, functions | `"object"` / `"function"`   |

| |`null`|`undefined`|
|---|---|---|
|Meaning|Intentional "no value"|Not assigned / missing|
|Set by|Developer|JavaScript engine|
|`typeof`|`"object"`|`"undefined"`|
|`null == undefined`|true|true|
|`null === undefined`|false|false|

**Primitives** are immutable and copied **by value**. **Objects** are copied **by reference** (the reference is copied, not the object).

```js
let x = 1, y = x; y = 2;           // x is still 1
const o1 = { n: 1 }, o2 = o1; o2.n = 2;  // o1.n is 2
{} === {}                           // false (different references)
```

### Edge cases

- `typeof null` → `"object"`. Check null with `value === null`.
- `typeof NaN` → `"number"`. `NaN !== NaN`; test with `Number.isNaN(x)` or `Object.is(x, NaN)`.
- `typeof []` → `"object"`. Use `Array.isArray(x)`.
- `typeof function(){}` → `"function"` (a special case of object).
- `-0 === 0` is `true`, but `Object.is(-0, 0)` is `false`.
- **Wrapper objects:** `typeof new String("a")` is `"object"`, and `new String("a") == "a"` is true but `===` is false. Never use `new String/Number/Boolean`.
- **Autoboxing:** `"abc".length` works because JS temporarily wraps the primitive. You can't add properties to primitives.
- **BigInt** can't mix with Number: `1n + 1` throws `TypeError`. Convert explicitly.
- **Reliable type check:** `Object.prototype.toString.call(x)` → `"[object Array]"`, `"[object Date]"`, `"[object Null]"`.
- `Symbol("a") !== Symbol("a")` — every symbol is unique.

### Use cases

- `null` for an intentionally empty value (API field, cleared selection). `undefined` for "not provided" (missing argument, missing key).
- `BigInt` for IDs or counters beyond `2^53 − 1`.
- `Symbol` for unique object keys or library-internal constants.
- Reference vs value is the root of most "why did my state change?" bugs, especially in React state updates.

---

## 3. Operators

### Definition

Operators perform operations on operands (values). Categories: arithmetic, comparison, logical, assignment, unary, ternary.

### Summary

| Category           | Operators                                                     |
| ------------------ | ------------------------------------------------------------- |
| Arithmetic         | `+ - * / % **` `++ --`                                        |
| Comparison         | `== != === !== < > <= >=`                                     |
| Logical            | `&& \| !`                                                     |
| Nullish / optional | `??` `?.`                                                     |
| Assignment         | `= += -= *= /= \|= &&= ??=`                                   |
| Other              | `?:` (ternary), `typeof`, `in`, `instanceof`, `delete`, `...` |

**Equality**

- `===` strict: no type conversion. Default choice.
- `==` loose: converts types first. Only common intentional use: `x == null` (true for both `null` and `undefined`).

**Logical operators return an operand, not a boolean.**

- `a || b` → first truthy operand (or the last)
- `a && b` → first falsy operand (or the last)
- `a ?? b` → `b` only if `a` is `null` or `undefined`

```js
0 || 10        // 10
0 ?? 10        // 0
"" || "n/a"    // "n/a"
"" ?? "n/a"    // ""
user && user.name
user?.address?.city   // undefined instead of TypeError
fn?.()                // call only if fn exists
arr?.[0]
```

### Edge cases

- `%` is **remainder**, not modulo: `-5 % 3` → `-2`.
- `**` is right-associative: `2 ** 3 ** 2` → `512`.
- `/ 0` → `Infinity`, `0 / 0` → `NaN` (no exception).
- Prefix vs postfix: `let i = 1; i++` returns `1`, `++i` returns `2`.
- String comparison is lexicographic: `"10" < "9"` is `true`. `"10" < 9` is `false` (number comparison).
- Chained comparisons don't work as in math: `1 < 2 < 3` is `true`, but `3 > 2 > 1` is `false` (`true > 1` → `1 > 1`).
- `null >= 0` is `true` but `null == 0` is `false` (relational operators convert `null` to `0`; `==` has its own rules).
- `NaN` is never equal to anything, including itself.
- **Mixing `??` with `||` or `&&` without parentheses is a SyntaxError.**
- `&&` binds tighter than `||`: `a || b && c` is `a || (b && c)`.
- `!!x` converts to boolean (same as `Boolean(x)`).

### Use cases

- `??` for defaults when `0`, `""` or `false` are valid values (counts, input values).
- `?.` for safely reading API responses where nested fields may be missing.
- `&&` for conditional execution/rendering, `||=` / `??=` for lazy defaults (`config.retries ??= 3`).

---

## 4. Type coercion

### Definition

**Type coercion** is the automatic (implicit) or manual (explicit) conversion of a value from one type to another. Three target conversions exist: **to string**, **to number**, **to boolean**.

### Summary

**To boolean: the 8 falsy values** `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy, including `"0"`, `"false"`, `[]`, `{}`, and `function(){}`.

**To number (`Number(x)`)**

|Input|Result|
|---|---|
|`"42"`, `" 42 "`|`42`|
|`""`, `" "`|`0`|
|`"42px"`, `"abc"`|`NaN`|
|`true` / `false`|`1` / `0`|
|`null`|`0`|
|`undefined`|`NaN`|
|`[]`|`0`|
|`[5]`|`5`|
|`[1, 2]`, `{}`|`NaN`|

**Operator rules**

- `+`: if either operand is a string (after converting objects to primitives) → concatenation. Otherwise → numeric addition.
- `- * / % **`: always convert both sides to numbers.
- `==`: `null == undefined` is true and equals nothing else. string/number/boolean → compared as numbers. Object vs primitive → object converted to primitive first.

**Explicit conversion**

|Goal|Tools|
|---|---|
|To number|`Number(x)`, unary `+x`, `parseInt(x, 10)`, `parseFloat(x)`|
|To string|`String(x)`, `` `${x}` ``, `x.toString()`|
|To boolean|`Boolean(x)`, `!!x`|

`Number("42px")` → `NaN` (strict). `parseInt("42px")` → `42` (reads as many digits as it can).

### Examples

```js
"5" + 2         // "52"
"5" - 2         // 3
"5" * "2"       // 10
true + 1        // 2
null + 1        // 1
undefined + 1   // NaN
[] + []         // ""
[] + {}         // "[object Object]"
[1, 2] + [3]    // "1,23"
```

### Edge cases

- `["1", "2", "3"].map(parseInt)` → `[1, NaN, NaN]` because `map` passes `(value, index)` and `parseInt`'s second argument is the radix. Use `.map(Number)` or `.map(s => parseInt(s, 10))`.
- `parseInt(0.0000005)` → `5` (the number becomes the string `"5e-7"` first).
- `parseInt(null)` → `NaN` (converts to `"null"`). `Number(null)` → `0`.
- `[] == false` → `true`, but `if ([])` is truthy. Loose equality and truthiness use different rules.
- `[] == ![]` → `true` (`![]` is `false`, then `"" == 0` path).
- `"b" + "a" + +"a" + "a"` → `"baNaNa"`.
- `isNaN("abc")` → `true` (coerces first). `Number.isNaN("abc")` → `false`. Prefer `Number.isNaN`.
- `Number("")` is `0`, so "empty input = 0" bugs are common in forms.
- **React gotcha:** `{count && <List />}` renders `0` when `count` is `0`. Use `{count > 0 && <List />}` or a ternary.
- Objects convert via `valueOf()` then `toString()` (or `Symbol.toPrimitive` if defined).

### Use cases

- Form inputs are always strings → convert with `Number()` and validate with `Number.isNaN`.
- Truthiness checks for presence (`if (name)`) — know which values count as empty.
- Query params, JSON values, `localStorage` values are strings and need explicit conversion.

---

## 5. Conditionals

### Definition

Conditionals run code depending on whether an expression evaluates to truthy or falsy: `if / else if / else`, `switch`, ternary `?:`, and short-circuit expressions.

### Summary

```js
if (score >= 90) grade = "A";
else if (score >= 75) grade = "B";
else grade = "C";

const label = isAdmin ? "Admin" : "User";

switch (status) {
  case "pending":
  case "processing":
    msg = "In progress"; break;       // break prevents fall-through
  case "done":
    msg = "Complete"; break;
  default:
    msg = "Unknown";
}
```

|Tool|Best for|
|---|---|
|`if / else`|General branching, complex conditions|
|Ternary|Simple two-way value selection (avoid nesting)|
|`switch`|Many discrete values of one variable|
|Lookup object|Mapping a key to a value/function|
|Guard clause|Early return to reduce nesting|

**Guard clause**

```js
function getDiscount(user) {
  if (!user) return 0;
  if (!user.isMember) return 0;
  return user.years > 5 ? 0.2 : 0.1;
}
```

**Lookup object instead of switch**

```js
const colors = { success: "green", error: "red", warning: "orange" };
const color = colors[status] ?? "gray";
```

### Edge cases

- `switch` compares with `===`: `switch ("1") { case 1: }` never matches.
- Missing `break` causes **fall-through** into the next case.
- `switch (true) { case x > 5: ... }` pattern works but is easy to misread.
- `if ([])` and `if ({})` are **true**. Check emptiness with `arr.length` or `Object.keys(obj).length`.
- `if ("0")` and `if ("false")` are true (non-empty strings).
- `if (x = 5)` assigns instead of comparing. It's always truthy and a classic typo.
- Nested ternaries hurt readability. Use `if` or a lookup instead.
- Lexical declarations in `case` need braces: `case 1: { let a = 1; break; }`.

### Use cases

- Form validation, permission checks, feature flags.
- Mapping API status codes or roles to UI labels/colors (lookup object).
- Conditional rendering in UI (ternary, `&&`).

---

## 6. Loops

### Definition

Loops repeat a block of code while a condition holds or for each item of a collection.

### Summary

|Loop|Iterates|Notes|
|---|---|---|
|`for (init; cond; step)`|Counter-based|Most control, works with `break`/`continue`|
|`while (cond)`|Until condition false|Unknown iteration count|
|`do { } while (cond)`|Same, runs at least once||
|`for...of`|**Values** of iterables (Array, String, Map, Set, NodeList)|Supports `break`, `continue`, `await`|
|`for...in`|**Enumerable keys** (strings) of an object|Includes inherited keys; avoid for arrays|
|`forEach`|Array elements|No `break`/`continue`; not awaited|

```js
for (const v of [10, 20]) {}                  // 10, 20
for (const [i, v] of ["a", "b"].entries()) {} // index + value
for (const k in { a: 1, b: 2 }) {}            // "a", "b"
for (const [k, v] of Object.entries(obj)) {}  // key + value pairs
```

### Edge cases

- `for...in` on arrays gives string indexes (`"0"`, `"1"`) and any extra/inherited properties. Use `for...of` instead.
- `for...in` order: integer-like keys first (ascending), then string keys in insertion order.
- `forEach` ignores `return` for flow control and does **not** wait for `async` callbacks:
    
    ```js
    items.forEach(async (x) => { await save(x); }); // fires all, doesn't waitfor (const x of items) { await save(x); }       // sequential, waits
    ```
    
- Use `Promise.all(items.map(...))` for parallel async work.
- Mutating an array while iterating (adding/removing) can skip or repeat items.
- Forgetting to update the loop condition causes an infinite loop (and a frozen tab).
- `break` exits the loop; `continue` skips to the next iteration. Labeled `break outer;` can exit nested loops.
- `for (const i = 0; i < 3; i++)` throws on the first `i++` (const).

### Use cases

- Rendering lists, summing values, searching for the first match (`for...of` + `break`).
- Retry logic with `while` and a counter.
- Sequential API calls where order matters (`for...of` + `await`).
- Iterating `NodeList` from `querySelectorAll`.

---

## 7. Strings

### Definition

A string is an immutable sequence of UTF-16 code units, written with `'single'`, `"double"` or `` `template` `` quotes.

### Summary

| Method                             | Purpose                                | Returns |
| ---------------------------------- | -------------------------------------- | ------- |
| `length`                           | Number of UTF-16 units                 | number  |
| `slice(start, end)`                | Substring (supports negatives)         | string  |
| `substring(start, end)`            | Substring (negatives become 0)         | string  |
| `at(-1)`                           | Character at index (negatives allowed) | string  |
| `indexOf / lastIndexOf`            | Position or `-1`                       | number  |
| `includes / startsWith / endsWith` | Existence checks                       | boolean |
| `split(sep)`                       | Break into array                       | array   |
| `trim / trimStart / trimEnd`       | Remove whitespace                      | string  |
| `toUpperCase / toLowerCase`        | Case change                            | string  |
| `replace / replaceAll`             | Substitute text                        | string  |
| `padStart / padEnd`                | Pad to length                          | string  |
| `repeat(n)`                        | Repeat                                 | string  |
| `localeCompare`                    | Locale-aware compare/sort              | number  |

```js
const s = "  Hello World  ";
s.trim()                  // "Hello World"
s.trim().slice(0, 5)      // "Hello"
s.includes("World")       // true
"a-b-c".split("-")        // ["a", "b", "c"]
"7".padStart(3, "0")      // "007"
`Hi ${name}, total: ${price * qty}`   // template literal (also multi-line)
```

### Edge cases

- **Immutable:** `s[0] = "x"` is silently ignored (throws `TypeError` in strict mode). Always use the returned value: `s = s.toUpperCase()`.
- **`length` counts UTF-16 units:** `"😀".length` → `2`. `[..."😀"].length` → `1`. `split("")` also breaks emoji apart.
- `replace("a", "b")` changes only the **first** match. Use `replaceAll` or a regex with `/g`.
- `slice` vs `substring` vs `substr`: prefer `slice`. `substr` is deprecated.
- `"abc".indexOf("")` → `0`. `"abc".split("")` → `["a","b","c"]`; `"".split("")` → `[]`; `"".split(",")` → `[""]`.
- `"10" < "9"` is `true` (character-by-character). Sort numbers with a numeric comparator.
- `"a" < "B"` is `false` (uppercase letters sort before lowercase). Use `localeCompare` for human sorting.
- `str.length` of a `null`/`undefined` value throws. Guard first.
- `+` with a number silently concatenates: `"Total: " + 5 + 5` → `"Total: 55"`.

### Use cases

- Slugs, truncation, capitalization, search/filter (`includes` + `toLowerCase`).
- Building URLs and messages with template literals (use `encodeURIComponent` for query values).
- Parsing CSV-like text, masking values (`"****" + card.slice(-4)`).

### Example snippets

```js
const capitalize = (s) => s ? s[0].toUpperCase() + s.slice(1).toLowerCase() : "";

const slugify = (s) =>
  s.toLowerCase().trim().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");

const truncate = (s, n) => (s.length > n ? s.slice(0, n - 1) + "…" : s);
```

---

## 8. Numbers

### Definition

JavaScript has one numeric type, `number`, a **64-bit IEEE 754 double** (plus `bigint` for arbitrary-size integers). It has special values `NaN`, `Infinity`, `-Infinity`.

### Summary

|Task|Tool|
|---|---|
|Round|`Math.round`, `Math.floor`, `Math.ceil`, `Math.trunc`|
|Min / max|`Math.min(...arr)`, `Math.max(...arr)`|
|Random integer 0..n-1|`Math.floor(Math.random() * n)`|
|Format decimals|`n.toFixed(2)` (returns a **string**)|
|Check|`Number.isInteger`, `Number.isFinite`, `Number.isNaN`, `Number.isSafeInteger`|
|Parse|`Number()`, `parseInt(s, 10)`, `parseFloat(s)`|
|Locale format|`n.toLocaleString("en-IN")`, `Intl.NumberFormat`|

```js
(1234567.891).toLocaleString("en-IN");   // "12,34,567.891"
new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" }).format(2499.5);
// "₹2,499.50"
```

### Edge cases

- **Floating point:** `0.1 + 0.2` → `0.30000000000000004`. Compare with a tolerance: `Math.abs(a - b) < Number.EPSILON`.
- **Safe integers:** beyond `Number.MAX_SAFE_INTEGER` (`9007199254740991`), precision is lost: `9007199254740992 === 9007199254740993` → `true`. Use `BigInt`.
- `toFixed` returns a string and rounding can surprise: `(1.005).toFixed(2)` → `"1.00"`.
- `Math.round(-2.5)` → `-2` and `Math.round(2.5)` → `3` (rounds half toward +∞).
- `Math.max()` with no arguments → `-Infinity`. `Math.max(...[])` same.
- `isNaN(x)` coerces; `Number.isNaN(x)` doesn't.
- `0.1 * 3`, `1.1 + 2.2` and similar give imprecise results. Money should be stored as integers (paise/cents).
- `parseInt("08")` → `8` (modern engines). Always pass the radix anyway.
- Numeric separators for readability: `1_000_000`.
- `Number.MAX_VALUE` is about `1.79e308`; larger values become `Infinity`.

### Use cases

- Prices/cart totals (integers in smallest unit, format at display).
- Pagination: `totalPages = Math.ceil(total / perPage)`.
- Clamp: `Math.min(Math.max(value, min), max)`.
- Percentages, progress bars, random IDs/selection, currency formatting for Indian locale.

---

## 9. Dates

### Definition

`Date` represents a single moment in time as **milliseconds since 1 Jan 1970 00:00:00 UTC** (the Unix epoch). It displays in local time but stores a UTC timestamp.

### Summary

```js
const now = new Date();           // current time
Date.now();                       // ms timestamp (number)
new Date("2026-10-15");           // ISO date-only → parsed as UTC
new Date("2026-10-15T10:30:00");  // no offset → parsed as local time
new Date(2026, 9, 15);            // year, monthIndex (0-11), day → local time
new Date(1760000000000);          // from timestamp
```

|Getter|Returns|
|---|---|
|`getFullYear()`|4-digit year|
|`getMonth()`|**0–11** (January = 0)|
|`getDate()`|Day of month 1–31|
|`getDay()`|Day of week 0–6 (Sunday = 0)|
|`getHours() / getMinutes() / getSeconds()`|Local time parts|
|`getTime()`|ms since epoch|
|`toISOString()`|UTC string `2026-10-15T04:30:00.000Z`|

**Formatting**

```js
d.toLocaleDateString("en-IN");                     // "15/10/2026"
d.toLocaleString("en-IN", { dateStyle: "medium", timeStyle: "short" });
new Intl.DateTimeFormat("en-IN", { month: "long", year: "numeric" }).format(d);
```

### Edge cases

- **Months are 0-indexed**, days are 1-indexed, weekdays start at Sunday = 0.
- **Parsing differences:** date-only ISO (`"2026-10-15"`) → UTC; date-time without offset → local. This can show the **previous day** in timezones behind UTC. Non-ISO strings (`"10/15/2026"`) are implementation-dependent. Avoid them.
- **Invalid date:** `new Date("abc")` → `Invalid Date`; `getTime()` → `NaN`. Check with `Number.isNaN(d.getTime())`.
- **Overflow rolls over:** `new Date(2026, 0, 32)` → 1 Feb 2026.
- **Dates are mutable:** `setDate`, `setMonth` etc. change the original. Copy first: `new Date(d)`.
- **Subtraction gives ms:** `d2 - d1` is a number. `d + 1` becomes a string.
- `d1 === d2` compares references. Compare `d1.getTime() === d2.getTime()`.
- Daylight saving time makes "add 24 hours" ≠ "add 1 day" in some zones.
- Two-digit years: `new Date(99, 0, 1)` → year 1999.
- For heavy date work use `date-fns` or `dayjs`. The `Temporal` API is the proposed modern replacement; check browser support before relying on it.

### Use cases

- Store timestamps as UTC ISO strings or epoch ms; display in the user's locale.
- "2 hours ago" labels, countdown timers, expiry checks (`Date.now() > expiresAt`).
- Calculating age, date ranges, deadlines; formatting for tables and invoices.

### Example snippets

```js
const addDays = (date, n) => { const d = new Date(date); d.setDate(d.getDate() + n); return d; };

const daysBetween = (a, b) =>
  Math.round(Math.abs(Date.UTC(b.getFullYear(), b.getMonth(), b.getDate()) -
                      Date.UTC(a.getFullYear(), a.getMonth(), a.getDate())) / 86400000);

const isValidDate = (d) => d instanceof Date && !Number.isNaN(d.getTime());
```

---

## 10. Interview questions & answers

> [!tip] How to use Cover the answer and say it out loud first. Aim for a 20–40 second answer.

### Basic

> [!question]- What is the difference between `var`, `let` and `const`? `var` is function-scoped, hoisted as `undefined`, and can be redeclared. `let` and `const` are block-scoped and sit in the TDZ until declared. `const` can't be reassigned (but objects it points to can be mutated). Use `const` by default, `let` when reassignment is needed.

> [!question]- What is hoisting? Declarations are processed before code executes, so they're known throughout their scope. `var` is initialized as `undefined`, function declarations are fully hoisted, `let`/`const` are hoisted but not initialized (TDZ).

> [!question]- What is the Temporal Dead Zone? The period between entering a scope and the line where a `let`/`const`/`class` is initialized. Accessing the variable there throws `ReferenceError`.

> [!question]- What are the primitive data types? `string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`.

> [!question]- Difference between `null` and `undefined`? `undefined` means a value hasn't been assigned (set by JS). `null` is an intentional empty value (set by the developer). `typeof null` is `"object"`, `typeof undefined` is `"undefined"`. `null == undefined` but `null !== undefined`.

> [!question]- `==` vs `===`? `===` compares value and type without conversion. `==` converts types first, which causes surprises (`0 == ""`, `"1" == 1`). Use `===`; the one common exception is `x == null`.

> [!question]- What are the falsy values? `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`.

> [!question]- How do you check if a value is an array? `Array.isArray(value)`. `typeof` returns `"object"` for arrays.

> [!question]- Are strings mutable? No. Methods return new strings; the original never changes.

> [!question]- `for...in` vs `for...of`? `for...in` iterates enumerable property **keys** (including inherited) and is meant for objects. `for...of` iterates **values** of iterables (arrays, strings, Maps, Sets).

### Intermediate

> [!question]- Why does `typeof null` return `"object"`? A bug from the first JavaScript implementation (null's type tag was 0, the same as objects). It was never fixed for backward compatibility.

> [!question]- What is type coercion? Give examples. Automatic conversion between types. `"5" + 2` → `"52"` (string concatenation), `"5" - 2` → `3` (numeric), `true + 1` → `2`. Explicit conversion uses `Number()`, `String()`, `Boolean()`.

> [!question]- What is the difference between `||` and `??`? `||` returns the right side when the left is **any falsy** value. `??` returns the right side only when the left is `null` or `undefined`. Use `??` when `0`, `""` or `false` are valid values.

> [!question]- Why is `0.1 + 0.2 !== 0.3`? Numbers are binary floating point; 0.1 and 0.2 can't be represented exactly, so the sum is `0.30000000000000004`. Compare with an epsilon tolerance or use integers for money.

> [!question]- What does `const` actually make constant? The variable binding (it can't be reassigned). The value can still change if it's an object or array. `Object.freeze` prevents property changes (shallowly).

> [!question]- How do you convert a string to a number? Differences between the options? `Number(s)` is strict (`"42px"` → `NaN`). `parseInt(s, 10)` reads leading digits (`"42px"` → `42`). `parseFloat` keeps decimals. Unary `+s` behaves like `Number`.

> [!question]- Why does `forEach` not work with `await`? `forEach` ignores returned promises, so async callbacks all start and the loop finishes without waiting. Use `for...of` with `await` for sequential work or `Promise.all(map(...))` for parallel work.

> [!question]- How does `switch` compare values? With strict equality (`===`). Without `break`, execution falls through to the next case.

> [!question]- Why can `{count && <List />}` render `0` in React? `&&` returns the first falsy operand, and React renders the number `0`. Use `count > 0 && ...` or a ternary.

> [!question]- Why do dates sometimes show the previous day? `new Date("2026-10-15")` is parsed as UTC midnight. In timezones behind UTC, local display is the previous evening. Use a full local date-time string or the `(y, m, d)` constructor, or format in UTC.

> [!question]- What is the difference between `slice`, `substring` and `substr`? `slice(start, end)` supports negative indexes. `substring` treats negatives as 0 and swaps arguments if start > end. `substr(start, length)` is deprecated.

### Tricky / follow-up

> [!question]- Why does `["1","2","3"].map(parseInt)` return `[1, NaN, NaN]`? `map` calls `parseInt(value, index, array)`, so the index is used as the radix: `parseInt("2", 1)` and `parseInt("3", 2)` are `NaN`. Use `.map(Number)`.

> [!question]- Why is `"😀".length` equal to 2? Strings are UTF-16; characters outside the Basic Multilingual Plane use two code units (a surrogate pair). `[..."😀"].length` is 1 because iteration is by code point.

> [!question]- What does `Object.is` do differently from `===`? `Object.is(NaN, NaN)` is `true` and `Object.is(-0, 0)` is `false`; `===` gives the opposite for both.

> [!question]- How would you safely access a deeply nested property? Optional chaining with a fallback: `data?.user?.address?.city ?? "Unknown"`.

> [!question]- How do you avoid floating-point issues with money? Store amounts as integers in the smallest unit (paise/cents), do arithmetic on integers, and format only for display with `Intl.NumberFormat`.

---

## 11. Predict the output

```js
// 1
console.log(typeof null, typeof undefined, typeof NaN);
// 2
console.log("3" + 1 + 2, 1 + 2 + "3");
// 3
console.log(0 || "a", 0 ?? "a", "" && "b");
// 4
console.log([] == false, null == 0, null >= 0);
// 5
var x = 1; { var x = 2; } console.log(x);
let y = 1; { let y = 2; } console.log(y);
// 6
console.log(a); var a = 5;
console.log(b); let b = 5;
// 7
console.log(["1", "2", "3"].map(parseInt));
// 8
console.log([] + [], [] + {}, [1, 2] + [3]);
// 9
console.log(typeof typeof 1);
// 10
console.log(1 < 2 < 3, 3 > 2 > 1);
// 11
console.log("b" + "a" + +"a" + "a");
// 12
console.log(new Date("2026-10-15").getMonth());
```

<details> <summary>Answers</summary>

1. `"object" "undefined" "number"`
2. `"312"` and `"33"`
3. `"a"`, `0`, `""`
4. `true`, `false`, `true`
5. `2`, then `1`
6. `undefined`, then `ReferenceError` on the `b` line
7. `[1, NaN, NaN]`
8. `""`, `"[object Object]"`, `"1,23"`
9. `"string"`
10. `true`, `false`
11. `"baNaNa"`
12. `9` (months are 0-indexed)

</details>

---

## 12. Coding practice

**1. Safe string → number (return `null` if invalid)**

```js
function toNumber(v) {
  if (v == null) return null;
  if (typeof v === "string" && v.trim() === "") return null;
  const n = Number(v);
  return Number.isNaN(n) ? null : n;
}
```

**2. FizzBuzz**

```js
for (let i = 1; i <= 100; i++) {
  let out = "";
  if (i % 3 === 0) out += "Fizz";
  if (i % 5 === 0) out += "Buzz";
  console.log(out || i);
}
```

**3. Reverse a string / check palindrome (Unicode-safe)**

```js
const reverse = (s) => [...s].reverse().join("");
const isPalindrome = (s) => {
  const t = s.toLowerCase().replace(/[^a-z0-9]/g, "");
  return t === reverse(t);
};
```

**4. Most frequent character**

```js
function mostFrequent(str) {
  const counts = new Map();
  for (const ch of str) counts.set(ch, (counts.get(ch) ?? 0) + 1);
  let best = null, max = 0;
  for (const [ch, n] of counts) if (n > max) { best = ch; max = n; }
  return best;
}
```

**5. Capitalize each word**

```js
const titleCase = (s) =>
  s.toLowerCase().split(" ").filter(Boolean)
   .map((w) => w[0].toUpperCase() + w.slice(1)).join(" ");
```

**6. Check a valid finite number**

```js
const isNumber = (v) => typeof v === "number" && Number.isFinite(v);
```

**7. Clamp and paginate**

```js
const clamp = (n, min, max) => Math.min(Math.max(n, min), max);
const totalPages = (total, perPage) => Math.ceil(total / perPage);
```

---

## 13. Quick-recall cheat sheet

|Need|Use|
|---|---|
|Declare a variable|`const` → `let`|
|Strict equality|`===`, `Object.is` for `NaN`/`-0`|
|Default value|`??` (keeps `0`, `""`, `false`)|
|Safe nested read|`a?.b?.c`|
|Array check|`Array.isArray(x)`|
|NaN check|`Number.isNaN(x)`|
|String → number|`Number(s)` / `parseInt(s, 10)`|
|Loop values / keys|`for...of` / `for...in` (objects)|
|Async in loop|`for...of` + `await`, or `Promise.all(map())`|
|Replace all|`replaceAll` or `/g` regex|
|Compare floats|`Math.abs(a - b) < Number.EPSILON`|
|Format money (India)|`Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" })`|
|Compare dates|`a.getTime() === b.getTime()`|

---

## Related notes

- [[JS 02 - Functions]]
- [[JS 03 - Arrays & Objects]]
- [[JS 05 - Core Concepts (Scope, Closures, this)]]