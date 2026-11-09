# Objects in JavaScript

## Topics

1. Ways to Create an Object
2. The Four Pillars of OOP
3. Prototype and `__proto__`

---

## Ways to Create an Object (6)

|#|Method|Key point|
|---|---|---|
|1|Object literal `{}`|Simple, but **not a blueprint** (a change must be repeated in every object)|
|2|Constructor function (`new Person()`)|Pre-ES6. Methods defined inside are **copied per instance**, so put them on `Person.prototype`|
|3|`class`|ES6. **Syntactic sugar** over constructor + prototype. Methods live on the prototype (shared)|
|4|`Object.create(proto)`|Creates an object whose `__proto__` is `proto`. `Object.create(null)` → no prototype|
|5|`new Object()`|Same as literal, so avoid|
|6|Factory function|Returns an object, no `new`. Methods copied per object, `instanceof` doesn't work|

- Naming: `camelCase` for functions, `PascalCase` for constructors/classes.
- Constructor functions are **not deprecated**, but `class` is preferred.

```js
function Person(fname) { this.fname = fname; }
Person.prototype.greet = function () { return `Hi ${this.fname}`; }; // shared

class Person {
  constructor(fname) { this.fname = fname; } // runs on `new`
  greet() { return `Hi ${this.fname}`; }     // on Person.prototype
}
```

**`new` does 4 things:**

1. Creates an empty object
2. Sets its `__proto__` = `Constructor.prototype`
3. Runs the constructor with `this` = new object
4. Returns the object

**Errors:**

- Class without `new` → `TypeError: Class constructor X cannot be invoked without 'new'`
- Class used before declaration → `ReferenceError` (TDZ)
- Constructor function without `new` → `this` is global (`undefined` in strict mode), so it is a silent bug

---

## Four Pillars of OOP

|Pillar|Definition|JS mechanism|
|---|---|---|
|**Encapsulation**|Bundling data with methods and restricting direct access|`#private` fields, closures|
|**Abstraction**|Exposing only essential behavior, hiding implementation|Public methods over private state|
|**Inheritance**|A class acquires properties and methods of another|`extends`, `super`, **prototype chain**|
|**Polymorphism**|Same method name, different behavior|**Method overriding**|

---

## Prototype and `__proto__`

**Definition:** every object has an internal `[[Prototype]]` link (exposed as `__proto__`) to another object or `null`. **Prototypal inheritance** = property lookup through this link (**prototype chain**).

**Why:** copying methods violates **DRY**. Linking via the prototype **shares** them.

**Rules:**

- Lookup: **own property** first → then `__proto__` → repeat until found or `null` (returns `undefined`).
- **Shadowing:** an own property hides a prototype property of the same name.
- **Writes** go on the object itself, never on the prototype.
- Chain always ends: `Object.prototype.__proto__ === null`.

```js
p3.__proto__ = p2;
p2.__proto__ = p1;
// p3 → p2 → p1 → Object.prototype → null
p3.x1; // found on p1 via the chain
```

### `__proto__` vs `prototype` ⭐

||`__proto__`|`prototype`|
|---|---|---|
|On|Every object|Constructor functions / classes only|
|Purpose|**Actual link** used for lookup|**Blueprint** used to build instances' `__proto__`|

```js
const s1 = new Student();
s1.__proto__ === Student.prototype; // true
```

### "Everything is an object"

- Primitives get a temporary **wrapper object** (`String`, `Number`, `Boolean`, `Symbol`, `BigInt`) on property access.
- `"hi".__proto__ === String.prototype` → `Object.prototype` → `null` (same pattern for `Number`, `Boolean`).
- Functions and arrays are objects too, so they also trace back to `Object.prototype`.
- **Exception:** primitives themselves are **not** objects. `null`/`undefined` have no wrapper (`null.x` → `TypeError`).

### `instanceof`

Checks whether `Class.prototype` appears **anywhere in the object's chain**.

```js
s1.__proto__ = null;
s1 instanceof Student; // false (chain broken)
```

### Edge Cases

- **Mutation through the chain:** `p2.__proto__.fname = "x"` changes `p1.fname`. `__proto__` is a **reference**, not a copy.
- **`this` is decided by the call-site:** `s1.__proto__.getName()` → `this` is the prototype, so you get `undefined`. `s1.getName()` works.
- `Object.create(null)` has no `toString`/`hasOwnProperty`.
- Arrow functions and method shorthand have **no `prototype`** property.
- `for...in` also lists **inherited enumerable** properties. Filter with `Object.hasOwn()`.
- **Best practice:** don't modify `__proto__` directly (legacy, causes bugs, deoptimizes). Use `Object.create()`, `Object.getPrototypeOf()`, `Object.setPrototypeOf()`.

### Errors

|Case|Error|
|---|---|
|Circular chain (`a.__proto__ = b; b.__proto__ = a`)|`TypeError: Cyclic __proto__ value`|
|Missing inherited method `obj.foo()`|`TypeError: obj.foo is not a function`|
|`new` on an arrow function|`TypeError: x is not a constructor`|

### Quick Interview Answers

- **Prototype chain?** Linked list of objects through `__proto__`, ending at `null`.
- **`__proto__` vs `prototype`?** Link on the instance vs blueprint on the constructor.
- **Why is everything an object?** Every chain ends at `Object.prototype` (primitives via wrapper objects).
- **What does `new` do?** Create object, link `__proto__` to `Constructor.prototype`, run constructor, return object.
- **Why put methods on the prototype?** One shared copy instead of one per instance.