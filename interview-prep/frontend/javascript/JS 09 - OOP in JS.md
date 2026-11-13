# JS 09 — OOP in JS

> [!info] Scope
> Prototypes & the prototype chain · Constructor functions · ES6 classes · Getters & setters · Private fields
> Previous: [[JS 08 - Async JavaScript]] · Next: [[JS 10 - Error Handling]]
> Prototypes are the real mechanism. **Classes are syntax on top of them.** Interviewers love asking "what happens under the hood".

---

## 0. One-page summary

| Topic | The one thing to remember |
|---|---|
| Prototype | Every object has a hidden link ([[Prototype]]) to another object. Property lookup follows this chain. |
| Prototype chain | `obj` → `obj`'s prototype → ... → `Object.prototype` → `null`. First match wins. |
| `__proto__` vs `prototype` | `__proto__` ([[Prototype]]) is on **every object**. `prototype` is a property on **constructor functions**; it becomes the `__proto__` of instances made with `new`. |
| Assignment | Writing `obj.x = 1` always creates/updates an **own** property (shadowing the prototype's). It never edits the prototype. |
| `new F()` | Creates an object linked to `F.prototype`, runs `F` with `this` = that object, returns it (unless `F` returns an object). |
| Methods on prototype | Shared by all instances (memory-efficient). Methods created in the constructor are copied per instance. |
| `class` | Syntactic sugar over prototypes, but **not hoisted** (TDZ), always strict, **can't be called without `new`**, methods are non-enumerable. |
| `extends` / `super` | Links both the instance chain and the static chain. In derived constructors call `super()` **before** `this`. |
| Getters/setters | `get x()` / `set x(v)` look like properties but run code. Class accessors live on the prototype. |
| Private fields | `#x` is truly private (enforced by the language). Not accessible outside the class, not inherited by subclasses. |
| `instanceof` | Checks whether `Constructor.prototype` is anywhere in the object's prototype chain. |
| Prefer | Composition over deep inheritance. |

**The four OOP pillars in JavaScript**

| Pillar | How JavaScript does it |
|---|---|
| Encapsulation | Closures, `#private` fields, getters/setters, modules |
| Abstraction | Hide details behind a public API (methods, abstract-style base classes) |
| Inheritance | Prototype chain (`Object.create`, `extends`) |
| Polymorphism | Same method name, different behavior (method overriding, duck typing) |

---

## 1. Prototypes and the prototype chain

### Definition
Every JavaScript object has an internal link called **[[Prototype]]** pointing to another object (or `null`). When you read a property that isn't on the object itself, JavaScript looks at its prototype, then the prototype's prototype, and so on until it finds the property or reaches `null`. This is **prototypal inheritance**.

### Summary

```js
const animal = { eats: true, speak() { return "..."; } };
const dog = Object.create(animal);   // dog's [[Prototype]] is animal
dog.barks = true;

dog.barks;           // true       (own property)
dog.eats;            // true       (found on animal, one step up the chain)
dog.toString();      // "[object Object]"  (found on Object.prototype, two steps up)
dog.fly;             // undefined  (not found anywhere → end of chain)
```

**The chain for common values**
```
[1, 2, 3]  →  Array.prototype  →  Object.prototype  →  null
function f →  Function.prototype → Object.prototype  →  null
new Dog()  →  Dog.prototype     →  Animal.prototype  →  Object.prototype → null
```

**Working with prototypes**

| Tool | Purpose |
|---|---|
| `Object.getPrototypeOf(obj)` | Read an object's prototype (preferred) |
| `Object.setPrototypeOf(obj, proto)` | Change a prototype (slow, avoid) |
| `Object.create(proto)` | New object with the given prototype |
| `Object.create(null)` | Object with **no** prototype (a "pure dictionary") |
| `obj.__proto__` | Legacy accessor for the prototype (works, but avoid in new code) |
| `proto.isPrototypeOf(obj)` | Is `proto` in `obj`'s chain? |
| `obj instanceof Ctor` | Is `Ctor.prototype` in `obj`'s chain? |
| `Object.hasOwn(obj, key)` | Is `key` an **own** property? (replaces `hasOwnProperty`) |
| `key in obj` | Own **or** inherited property? |

**`prototype` vs `__proto__`**

| | `prototype` | `__proto__` / [[Prototype]] |
|---|---|---|
| Exists on | Constructor functions (not arrow functions or methods) | **Every** object |
| It is | The object that will be the prototype of instances created with `new` | The actual link used for property lookup |
| Access | `Dog.prototype` | `Object.getPrototypeOf(obj)` |
| Relationship | `new Dog().__proto__ === Dog.prototype` | |

```js
function Dog() {}
const d = new Dog();
Object.getPrototypeOf(d) === Dog.prototype;            // true
Dog.prototype.constructor === Dog;                     // true (back-reference)
Object.getPrototypeOf(Dog.prototype) === Object.prototype;   // true
Object.getPrototypeOf(Object.prototype);               // null (end of the chain)
```

**Reading vs writing**
```js
const proto = { x: 1 };
const obj = Object.create(proto);

obj.x;              // 1 (inherited)
obj.x = 2;          // creates an OWN property that shadows the inherited one
obj.x;              // 2
proto.x;            // 1 (prototype unchanged)
delete obj.x;
obj.x;              // 1 (inherited value is visible again)
```

**Shared methods vs per-instance methods**
```js
function A() { this.hi = function () {}; }       // a new function per instance
function B() {}
B.prototype.hi = function () {};                 // one function shared by all instances

new A().hi === new A().hi;   // false
new B().hi === new B().hi;   // true
```

### Edge cases
- **Writes never change the prototype** (unless the prototype defines a setter, or the inherited property is read-only). `obj.x = 1` creates an own property.
- **Mutable values on the prototype are shared:** `Ctor.prototype.list = []` means every instance shares the same array. Mutating it (`push`) affects all instances. Put mutable state in the constructor (`this.list = []`).
- **`for...in` includes inherited enumerable properties;** `Object.keys`, `Object.entries`, and `JSON.stringify` don't. Filter with `Object.hasOwn` if needed.
- **`instanceof` checks the chain, not the type:** `[] instanceof Object` is `true`. It can be fooled by changing `prototype` after creating objects, and fails across iframes/realms (use `Array.isArray`).
- **`Object.create(null)` objects have no `toString`, `hasOwnProperty`, etc.,** so `obj.hasOwnProperty(...)` throws. Use `Object.hasOwn(obj, key)`. `instanceof Object` is `false`.
- **Reassigning `Ctor.prototype`** (`Ctor.prototype = {...}`) breaks the `constructor` back-reference and doesn't affect already-created instances (they keep the old prototype).
- **Don't modify built-in prototypes** (`Array.prototype.myMethod = ...`). It can collide with future features and break libraries. Polyfilling a *standard* method that's missing is the one accepted exception.
- **`Object.setPrototypeOf` and `__proto__` assignment are slow** and de-optimize engines. Prefer `Object.create` at creation time.
- **Prototype pollution (security):** merging untrusted objects with keys like `__proto__` can modify `Object.prototype` for the whole app. Validate keys, use `Object.create(null)` or `Map` for user-keyed data.
- **Prototype chains can't be circular:** `Object.setPrototypeOf(a, a)` throws a `TypeError`.
- **Primitives get a temporary wrapper:** `"abc".toUpperCase()` works because strings borrow `String.prototype` (autoboxing).
- **Long chains are slower to look up.** Keep them shallow.
- **Class getters/methods live on the prototype,** so they're not own properties (`Object.hasOwn(instance, "method")` is `false`).

### Use cases
- Sharing methods across many instances without copying them.
- Explaining how arrays, functions, and class instances get their methods.
- Debugging "why does this object have this property?" (inspect the chain in DevTools: `__proto__`).
- Dictionary-style objects with `Object.create(null)`.

---

## 2. Constructor functions

### Definition
A **constructor function** is a regular function (by convention named in `PascalCase`) meant to be called with the **`new`** operator to create and initialize objects.

### Summary

```js
function Person(name, age) {
  this.name = name;           // own properties set in the constructor
  this.age = age;
}
Person.prototype.greet = function () {      // shared method on the prototype
  return `Hi, I'm ${this.name}`;
};

const p = new Person("Asha", 22);
p.greet();                    // "Hi, I'm Asha"
p instanceof Person;          // true
p.constructor === Person;     // true
```

**What `new` does (4 steps)**

| Step | Action |
|---|---|
| 1 | Creates a new empty object |
| 2 | Links its [[Prototype]] to `Constructor.prototype` |
| 3 | Calls the constructor with `this` = the new object |
| 4 | Returns the object (unless the constructor returns another **object**) |

**`new` implemented by hand (interview favorite)**
```js
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);          // steps 1 + 2
  const result = Constructor.apply(obj, args);               // step 3
  return result !== null && (typeof result === "object" || typeof result === "function")
    ? result                                                 // constructor returned an object
    : obj;                                                   // step 4
}
```

**Inheritance with constructor functions (the old way)**
```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} makes a sound`; };

function Dog(name, breed) {
  Animal.call(this, name);                               // 1. inherit instance properties
  this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype);         // 2. link the prototype chain
Dog.prototype.constructor = Dog;                         // 3. restore the constructor reference
Dog.prototype.speak = function () {                      // override
  return `${this.name} barks`;
};

const d = new Dog("Rex", "Lab");
d.speak();                  // "Rex barks"
d instanceof Dog;           // true
d instanceof Animal;        // true
```

**Alternatives to constructors**

| Pattern | Example | Pros / cons |
|---|---|---|
| Constructor function | `new Person()` | Shared prototype methods; needs `new`; `this` pitfalls |
| Factory function | `const createUser = n => ({ name: n, greet() {} })` | No `new`/`this`; real privacy through closures; methods copied per object |
| `Object.create` | `Object.create(proto)` | Direct prototype linking, no constructor |
| Class | `class User {}` | Cleanest syntax for the constructor + prototype pattern |

```js
// Factory function with real privacy via closure
function createCounter() {
  let count = 0;                         // private
  return { inc: () => ++count, get: () => count };
}
```

### Edge cases
- **Forgetting `new`:** `Person("A")` runs as a plain function. In sloppy mode `this` is the global object (it creates `window.name`!); in strict mode `this` is `undefined` → `TypeError`. Guard with `new.target`:
  ```js
  function Person(name) {
    if (!new.target) return new Person(name);
    this.name = name;
  }
  ```
- **Return values:** returning an **object** from a constructor replaces the new instance; returning a **primitive** is ignored.
- **Arrow functions can't be constructors** (no `prototype`, `new` throws `TypeError`). Object methods defined with shorthand (`{ m() {} }`) aren't constructors either.
- **Methods created in the constructor are duplicated per instance** (more memory). Methods on the prototype are shared.
- **`this` is lost** when a method is detached: `const g = p.greet; g()` → `this` is `undefined`/global.
- **Forgetting `Dog.prototype.constructor = Dog`** after replacing the prototype makes `d.constructor` point to `Animal`.
- **`Dog.prototype = new Animal()`** (old pattern) runs the parent constructor at definition time and shares its instance state. Use `Object.create(Animal.prototype)` + `Animal.call(this)`.
- **`instanceof` with a re-assigned prototype:** if you change `Ctor.prototype` later, existing instances are no longer `instanceof Ctor`.
- **Parent constructors that need `new`** (classes, `Map`, `Error` subclasses) can't be called with `Animal.call(this)`. Use `class extends` for those.
- **Constructor functions are hoisted** (function declarations), unlike classes.
- **Properties added after creation** (`p.extra = 1`) work, but hurt engine optimizations. Initialize all properties in the constructor, in the same order.

### Use cases
- Legacy code, older libraries, and interviews that ask you to explain or implement `new`.
- Factory functions for private state without classes.
- Understanding what a `class` compiles down to.

---

## 3. ES6 classes

### Definition
A **class** (ES2015) is a cleaner syntax for creating constructor functions and prototype-based inheritance. Under the hood a class **is a function**, and its methods are placed on `Class.prototype`.

### Summary

```js
class Animal {
  legs = 4;                              // public instance field (own property)
  static count = 0;                      // static property (on the class itself)

  constructor(name) {
    this.name = name;
    Animal.count++;
  }

  speak() {                              // method → Animal.prototype.speak
    return `${this.name} makes a sound`;
  }

  static create(name) {                  // static method → called on the class
    return new this(name);
  }

  toString() { return `Animal(${this.name})`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);                         // must call super() before using `this`
    this.breed = breed;
  }
  speak() {                              // override
    return `${super.speak()} - woof`;    // call the parent version
  }
}

const d = new Dog("Rex", "Lab");
d.speak();                  // "Rex makes a sound - woof"
d instanceof Dog;           // true
d instanceof Animal;        // true
typeof Dog;                 // "function"
Animal.count;               // 1
```

**What `extends` sets up**
```js
Object.getPrototypeOf(Dog.prototype) === Animal.prototype;   // instances inherit from Animal
Object.getPrototypeOf(Dog) === Animal;                        // statics are inherited too (Dog.create works)
```

**Class vs constructor function**

| | Constructor function | Class |
|---|---|---|
| Hoisting | Declaration is hoisted | In the **TDZ** (`ReferenceError` before the definition) |
| Call without `new` | Allowed (bug-prone) | **`TypeError`** |
| Strict mode | Optional | **Always** |
| Prototype methods enumerable? | Yes (when assigned) | **No** |
| Inheritance setup | Manual (`Object.create`, `call`) | `extends` + `super` |
| Static inheritance | Manual | Automatic |
| Private members | Closures only | `#private` fields |
| Fields syntax | Inside the constructor | Class fields |

**Class members at a glance**

| Member | Syntax | Lives on |
|---|---|---|
| Constructor | `constructor() {}` | The class function |
| Instance method | `method() {}` | `Class.prototype` (shared) |
| Instance field | `x = 1;` | Each instance (own property) |
| Arrow-function field | `handle = () => {}` | Each instance (auto-bound `this`) |
| Static method / field | `static m() {}` / `static x = 1` | The class itself |
| Static block | `static { ... }` | Runs once when the class is defined |
| Getter / setter | `get x() {}` / `set x(v) {}` | `Class.prototype` |
| Private | `#x`, `#m()`, `static #x` | Per instance / class, hidden |

**Polymorphism and abstract-style base classes**
```js
class Shape {
  constructor() {
    if (new.target === Shape) throw new TypeError("Shape is abstract");
  }
  area() { throw new Error("area() must be implemented"); }
  describe() { return `${this.constructor.name} with area ${this.area().toFixed(2)}`; }
}
class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return Math.PI * this.r ** 2; }
}
class Square extends Shape {
  constructor(s) { super(); this.s = s; }
  area() { return this.s ** 2; }
}
[new Circle(1), new Square(2)].forEach(s => console.log(s.describe()));   // same call, different behavior
```

**Composition and mixins (alternative to deep inheritance)**
```js
const Serializable = Base => class extends Base {
  serialize() { return JSON.stringify(this); }
};
const Timestamped = Base => class extends Base {
  createdAt = Date.now();
};
class User { constructor(name) { this.name = name; } }
class AppUser extends Serializable(Timestamped(User)) {}
new AppUser("Asha").serialize();
```

**Extending built-ins**
```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}
class Stack extends Array {
  peek() { return this[this.length - 1]; }
}
```

### Edge cases
- **Not hoisted for use:** `new C(); class C {}` → `ReferenceError` (TDZ). Unlike function declarations.
- **Must use `new`:** `Animal()` throws `TypeError: Class constructor Animal cannot be invoked without 'new'`.
- **Derived constructors must call `super()` before touching `this`:** otherwise `ReferenceError: Must call super constructor...`. If you omit a constructor in a subclass, one is generated for you (`constructor(...args) { super(...args); }`).
- **Class fields initialize in a specific order:** base-class fields before its constructor body; derived-class fields **right after `super()` returns**. So an overridden method called from the base constructor sees **uninitialized** derived fields:
  ```js
  class A { constructor() { this.init(); } init() { console.log("A"); } }
  class B extends A { x = 1; init() { console.log(this.x); } }
  new B();   // undefined (B's field x isn't set yet when A's constructor runs)
  ```
  Avoid calling overridable methods from constructors.
- **Methods aren't auto-bound:** `const { speak } = dog; speak()` → `this` is `undefined` (class bodies are strict). Fix with `bind`, arrow wrappers, or arrow-function fields.
- **Arrow-function fields trade-offs:** auto-bound `this`, but each instance gets its own copy (more memory), they're not on the prototype (can't call via `super.method()`), and are harder to override or spy on in tests.
- **Methods are non-enumerable:** `Object.keys(Dog.prototype)` → `[]`, and `for...in` over an instance skips them. Instance **fields** are enumerable own properties.
- **`static` members aren't available on instances** (`dog.create` is `undefined`), but are inherited by subclasses. `this` inside a static method is the class it was called on (so `new this()` works for subclasses).
- **Static fields are shared by reference** across subclasses via the prototype chain until a subclass assigns its own.
- **`typeof` a class is `"function"`.** Class declarations are block-scoped like `let`.
- **Class bodies are always strict mode.**
- **`super.method()` works only inside methods** (and field initializers). It's resolved via the home object, not `this`.
- **Overriding a method completely replaces it,** so call `super.method()` explicitly if you also need the parent behavior.
- **Extending built-ins:** `extends Array/Error/Map` works, but methods like `map`/`filter` return instances of the subclass (via `Symbol.species`). Set `this.name` on custom errors.
- **Deep inheritance hierarchies are brittle** (tight coupling, "gorilla/banana" problem). Prefer composition, small classes, and mixins.
- **`class` is not required:** factory functions + closures are a valid, often simpler approach.
- **`instanceof` and `constructor.name`** can break under minification (names are mangled). Don't rely on class names for logic.

### Use cases
- Modeling entities with behavior (User, Cart, Logger, API client, custom Error types).
- Framework code (React class components, Angular, Web Components `extends HTMLElement`).
- Shared base behavior with specialization (Shape → Circle/Square, Notification → Email/SMS).

---

## 4. Getters and setters

### Definition
**Accessor properties** look like normal properties but run a function when read (**getter**) or assigned (**setter**). They let you compute values, validate input, and hide internals behind a property-style API.

### Summary

```js
class Temperature {
  #celsius = 0;

  get celsius() { return this.#celsius; }
  set celsius(value) {
    if (typeof value !== "number" || Number.isNaN(value)) throw new TypeError("Number required");
    if (value < -273.15) throw new RangeError("Below absolute zero");
    this.#celsius = value;
  }

  get fahrenheit() { return this.#celsius * 9 / 5 + 32; }        // computed, read-only
  set fahrenheit(f) { this.celsius = (f - 32) * 5 / 9; }          // reuses the validating setter
}

const t = new Temperature();
t.celsius = 25;          // runs the setter
t.fahrenheit;            // 77   (runs the getter)
t.fahrenheit = 212;      // sets celsius to 100
```

**In object literals and with `Object.defineProperty`**
```js
const user = {
  first: "Asha",
  last: "Roy",
  get full() { return `${this.first} ${this.last}`; },
  set full(v) { [this.first, this.last] = v.split(" "); },
};
user.full;                 // "Asha Roy"
user.full = "Ravi Sen";    // updates first and last

const o = {};
Object.defineProperty(o, "id", {
  get() { return 42; },
  enumerable: true,
  configurable: false,
});
```

**Data property vs accessor property**

| | Data property | Accessor property |
|---|---|---|
| Holds | A value | A getter and/or setter function |
| Descriptor keys | `value`, `writable` | `get`, `set` |
| Read | Returns the stored value | Runs the getter |
| Write | Stores the value | Runs the setter (or fails if there is none) |

**Which combination does what**

| Defined | Reading | Writing |
|---|---|---|
| getter only | Works | Silently ignored (sloppy) / **`TypeError`** (strict mode and classes) |
| setter only | `undefined` | Works |
| both | Works | Works |

### Edge cases
- **Infinite recursion:** assigning to the same name inside its setter calls the setter again.
  ```js
  class Bad { set name(v) { this.name = v; } }   // RangeError: Maximum call stack size exceeded
  ```
  Store the value under a different name (`_name`) or a private field (`#name`).
- **Getter-only assignment throws in classes** (class bodies are strict): `obj.v = 5` → `TypeError: Cannot set property v of #<T> which has only a getter`.
- **Getters run on every access.** Don't put expensive work in them without caching.
- **Class accessors live on the prototype:** they aren't own properties, so they're **excluded** from `Object.keys`, spread (`{...obj}`), and `JSON.stringify`. Getters defined in an **object literal** are own enumerable properties, so they **are** included (as their current value).
  ```js
  class U { constructor() { this.a = 1; } get b() { return 2; } }
  const lit = { a: 1, get b() { return 2; } };
  JSON.stringify(new U());   // '{"a":1}'
  JSON.stringify(lit);       // '{"a":1,"b":2}'
  ```
- **Spreading copies the getter's value,** not the getter: `{ ...lit }` gives plain data properties.
- **Setters take exactly one parameter,** and getters take none.
- **Validation lives in the setter,** but the constructor must go through it too (`this.celsius = c` in the constructor, not `this.#celsius = c`) or you bypass the checks.
- **Subclasses overriding only a getter** accidentally remove the inherited setter (accessor pairs are defined per property). Override both.
- **Setters + `Object.assign`/spread/`structuredClone`:** `Object.assign(target, src)` triggers *target's* setters; spread doesn't.
- **Side effects in getters** make code surprising. A getter should be cheap and free of side effects.
- **Static accessors** exist: `static get instance() {}`.
- **DevTools** may show a getter value only when you click it (to avoid triggering side effects).
- **Lazy getter pattern:** compute once and replace the getter with a data property via `Object.defineProperty(this, "x", { value })`.

### Use cases
- Validation and normalization on assignment (`email`, `age`, `price`).
- Derived/computed values (`fullName`, `total`, `isEmpty`) without storing duplicates.
- Read-only views of private state; backwards-compatible refactors (a field becomes a computed property without changing callers).
- Reactive systems (Vue 2 used `defineProperty` getters/setters; Proxies in Vue 3).

---

## 5. Private fields (`#`)

### Definition
**Private class members** (ES2022) are declared with a `#` prefix and are accessible **only inside the class body**. The language enforces this: code outside can't read, write, or even detect them. This is real encapsulation, unlike the `_name` naming convention.

### Summary

```js
class BankAccount {
  #balance = 0;                       // private field
  static #count = 0;                  // private static field

  constructor(owner, initial = 0) {
    this.owner = owner;               // public field
    this.#balance = initial;
    BankAccount.#count++;
  }

  deposit(amount) {
    this.#validate(amount);
    this.#balance += amount;
    return this;
  }

  withdraw(amount) {
    this.#validate(amount);
    if (amount > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= amount;
    return this;
  }

  get balance() { return this.#balance; }         // read-only public view

  #validate(amount) {                              // private method
    if (!(amount > 0)) throw new RangeError("Amount must be positive");
  }

  static get accounts() { return BankAccount.#count; }

  static isAccount(obj) { return #balance in obj; }   // brand check (ES2022)
}

const acc = new BankAccount("Asha", 100).deposit(50);
acc.balance;                 // 150
acc.owner;                   // "Asha"
// acc.#balance;             // SyntaxError (outside the class body)
acc["#balance"];             // undefined (it's not a normal property)
Object.keys(acc);            // ["owner"]
BankAccount.isAccount(acc);  // true
```

**Ways to get privacy**

| Technique | Truly private? | Notes |
|---|---|---|
| `#field` / `#method()` | **Yes** (language-enforced) | Modern; per-class; syntax error if misused |
| Closures / factory functions | Yes | Methods copied per instance; no `this` |
| `WeakMap` keyed by instance | Yes (if the map is module-private) | Verbose; works in older environments |
| `Symbol` keys | No (discoverable via `Object.getOwnPropertySymbols`) | "Hard to collide", not private |
| `_name` convention | No | Only a social contract |
| TypeScript `private` | No (compile-time only) | Erased at runtime |

**Visibility comparison**

| | Public field | `_underscore` | `#private` |
|---|---|---|---|
| Accessible outside | Yes | Yes | **No** |
| In `Object.keys` / `JSON.stringify` | Yes | Yes | **No** |
| Inherited by subclass code | Yes | Yes | **No** (subclass code can't access it) |
| Enforced by the language | n/a | No | **Yes** |

### Edge cases
- **Outside access is a `SyntaxError` at parse time** (`obj.#x` outside the class), not a runtime `undefined`.
- **Must be declared up front** in the class body. You can't create `#x` dynamically (`this.#x = 1` without a declaration is a `SyntaxError`).
- **Not inherited:** a subclass instance *has* the parent's private fields (the parent's methods can use them), but the subclass's own code can't reference `#x` (SyntaxError unless the subclass declares its own `#x`, which is a separate field).
- **Accessing on the wrong object throws `TypeError`:** `Account.prototype.getBalance.call({})` fails because `{}` lacks the private field. Use the brand check `#x in obj` to test first.
- **Not on the prototype,** not enumerable, ignored by `Object.keys`, spread, `JSON.stringify`, and `structuredClone`. Copying an instance with spread **drops** private state.
- **`Object.freeze` doesn't freeze private fields,** and private fields can't be deleted.
- **Proxies break private access:** a `Proxy` wrapping an instance makes `this` the proxy inside methods, so `this.#x` throws `TypeError`. Bind methods to the target or avoid proxying such objects.
- **Each class evaluation creates distinct private names:** two classes with the same `#x` name don't share data (useful for class factories/mixins, but surprising).
- **`this` in static methods** can't access instance private fields; you need an instance (`obj.#x` works inside static methods when `obj` is an instance).
- **Testing:** private members can't be accessed from tests. Test through the public API.
- **Older environments** may need transpilation (Babel/TypeScript) for `#` syntax. Check target support.
- **Private doesn't mean secret:** a debugger can inspect them, and DevTools displays them. It's encapsulation, not security.
- **Private methods are not on the prototype** and are not overridable by subclasses; each instance gets a brand, not a copy of the function.

### Use cases
- Protecting invariants (balance, internal caches, state machines) behind a public API.
- Hiding helper methods (`#validate`, `#render`) that shouldn't be part of the API.
- Library/class design where consumers must not depend on internals.
- Brand checks for ergonomic `static isFoo(obj)` helpers.

---

## 6. Interview questions & answers

> [!tip] How to use
> Cover the answer and say it out loud first. Aim for a 20–40 second answer, then back it with a tiny example.

### Prototypes & the prototype chain

> [!question]- What is a prototype?
> An object that another object inherits properties and methods from. Every object has an internal [[Prototype]] link to its prototype (or `null`).

> [!question]- What is the prototype chain?
> The sequence of [[Prototype]] links from an object up to `Object.prototype` and finally `null`. When you access a property, JS checks the object, then each prototype in turn, and returns the first match (or `undefined`).

> [!question]- What is the difference between `__proto__` and `prototype`?
> `__proto__` (the [[Prototype]]) exists on every object and is the actual lookup link. `prototype` exists on constructor functions and is the object that becomes the `__proto__` of instances created with `new`. `new F().__proto__ === F.prototype`.

> [!question]- How do you create an object with a specific prototype?
> `Object.create(proto)`. For a dictionary with no inherited keys, `Object.create(null)`.

> [!question]- Does assigning to an inherited property change the prototype?
> No. It creates an own property on the object that shadows the inherited one. The prototype stays unchanged.

> [!question]- How do you check whether a property is own or inherited?
> `Object.hasOwn(obj, key)` (or `obj.hasOwnProperty(key)`) for own properties; the `in` operator for own or inherited ones.

> [!question]- What does `instanceof` check?
> Whether `Constructor.prototype` appears in the object's prototype chain. `[] instanceof Object` is `true`.

> [!question]- What is at the end of the prototype chain?
> `Object.prototype`, whose own prototype is `null`.

> [!question]- Why put methods on the prototype instead of in the constructor?
> A method on the prototype is created once and shared by all instances (less memory). Methods created inside the constructor are duplicated for every instance.

> [!question]- What is prototype pollution?
> An attack/bug where untrusted input modifies `Object.prototype` (e.g. via a `__proto__` key in a merge), changing behavior for every object. Prevent it by validating keys and using `Object.create(null)` or `Map`.

> [!question]- Is it a good idea to extend built-in prototypes?
> No. It can collide with future language features and break other code. Only polyfill standard methods that are missing.

### Constructor functions

> [!question]- What does the `new` operator do?
> It creates an empty object linked to `Constructor.prototype`, calls the constructor with `this` set to that object, and returns the object (unless the constructor returns another object).

> [!question]- What happens if you call a constructor function without `new`?
> `this` is the global object in sloppy mode (leaking properties) or `undefined` in strict mode (TypeError). Guard with `new.target` or use classes, which throw.

> [!question]- What if a constructor returns a value?
> Returning an object replaces the created instance; returning a primitive is ignored.

> [!question]- How would you implement `new`?
> `Object.create(Constructor.prototype)`, call the constructor with `apply(obj, args)`, return the result if it's an object, else `obj`. See coding practice.

> [!question]- How does inheritance work with constructor functions?
> Call the parent constructor with `Parent.call(this, ...)`, set `Child.prototype = Object.create(Parent.prototype)`, then restore `Child.prototype.constructor = Child`.

> [!question]- Factory function vs constructor function?
> A factory returns a new object without `new`/`this` and can use closures for real privacy, but each object gets its own copies of methods (unless you share them). A constructor uses `new` and shares prototype methods.

### Classes

> [!question]- Are ES6 classes real classes?
> No, they're syntactic sugar over prototypes and constructor functions. `typeof MyClass` is `"function"`, and methods live on `MyClass.prototype`.

> [!question]- How do classes differ from constructor functions?
> Classes aren't hoisted for use (TDZ), always run in strict mode, throw if called without `new`, have non-enumerable methods, have built-in `extends`/`super` and static inheritance, and support `#private` and class-field syntax.

> [!question]- What does `super` do?
> In a constructor, `super(...)` calls the parent constructor (required before using `this` in derived classes). In methods, `super.method()` calls the parent's version.

> [!question]- Why must you call `super()` before using `this`?
> In a derived class, `this` is created by the parent constructor. Accessing `this` first throws a `ReferenceError`.

> [!question]- What are static methods?
> Methods on the class itself (not instances), called as `Class.method()`. They're inherited by subclasses; `this` inside refers to the class they were called on.

> [!question]- What are class fields and how are they different from properties set in the constructor?
> Declared directly in the class body (`count = 0`). They're created as own properties on each instance before the constructor body (base class) or right after `super()` (derived class).

> [!question]- Why does `this` become `undefined` when passing a class method as a callback?
> Methods aren't bound, and class bodies are strict, so a detached call has no receiver. Use `bind`, an arrow wrapper, or an arrow-function field.

> [!question]- Arrow function fields vs prototype methods?
> Arrow fields auto-bind `this` but are created per instance (more memory), aren't on the prototype, and are harder to override or mock. Prototype methods are shared but need explicit binding when detached.

> [!question]- What is polymorphism in JavaScript?
> Different objects responding to the same method name with different behavior, via method overriding in subclasses or duck typing.

> [!question]- Composition vs inheritance?
> Inheritance models "is-a" and can create tight coupling and deep hierarchies. Composition ("has-a"/"can-do") builds behavior from small pieces (mixins, delegation, factories) and is usually more flexible.

> [!question]- How do you make an abstract class in JavaScript?
> There's no keyword; throw in the constructor when `new.target === AbstractClass`, and throw in methods that subclasses must override.

> [!question]- How do you create a singleton?
> Export a single instance from a module (modules are evaluated once), or keep a static instance in a class and return it from a static method.

### Getters & setters

> [!question]- What are getters and setters?
> Accessor properties that run code on read (`get`) or write (`set`), letting you compute values and validate assignments while keeping property syntax.

> [!question]- What happens if you assign to a property that has only a getter?
> Silently ignored in sloppy mode; a `TypeError` in strict mode (including inside classes).

> [!question]- Why does a setter that assigns to the same property name cause a stack overflow?
> `this.name = v` inside `set name` calls the setter again, recursively. Store the value in a different field (`#name` or `_name`).

> [!question]- Are class getters included in `JSON.stringify` or spread?
> No. They live on the prototype and aren't own enumerable properties. Getters in an object literal *are* own properties and are included (as values).

> [!question]- When would you use a getter instead of a method?
> For cheap, side-effect-free derived values that read naturally as properties (`fullName`, `isEmpty`). Use a method for expensive work or actions.

> [!question]- How do you add validation to a property?
> A setter that checks the value before storing it (in a private field), and make the constructor assign through the setter so validation always runs.

### Private fields

> [!question]- What are private class fields?
> Class members prefixed with `#` that are only accessible inside the class body and enforced by the language (accessing them outside is a `SyntaxError`).

> [!question]- How is `#private` different from `_private`?
> `_name` is only a naming convention and fully accessible; `#name` is truly inaccessible from outside, hidden from `Object.keys`/`JSON.stringify`, and not accessible from subclass code.

> [!question]- Can subclasses access a parent's private fields?
> No. The parent's methods can, but code written in the subclass can't reference the parent's `#x`. Expose a protected-style getter or method instead.

> [!question]- How can you check whether an object has a given private field?
> The brand check `#field in obj` (ES2022), usable inside the class.

> [!question]- Name other ways to achieve privacy in JavaScript.
> Closures (factory functions/module pattern), `WeakMap`-based private state, and (weakly) Symbols or underscore naming.

> [!question]- Do private fields survive spreading or `structuredClone`?
> No. They aren't own enumerable properties, so copies don't include them.

> [!question]- Why can a Proxy break private fields?
> Methods called through a proxy get `this` = the proxy, which doesn't have the target's private fields, so `this.#x` throws a `TypeError`.

---

## 7. Predict the output

```js
// 1
function Person(name) { this.name = name; }
Person.prototype.hi = function () { return "hi " + this.name; };
const p = new Person("A");
console.log(p.hi(), p.hasOwnProperty("hi"), "hi" in p);

// 2
console.log(
  p.__proto__ === Person.prototype,
  Person.prototype.constructor === Person,
  Object.getPrototypeOf(Person.prototype) === Object.prototype
);

// 3
console.log(Object.getPrototypeOf(Object.prototype));

// 4
const proto = { x: 1 };
const obj = Object.create(proto);
obj.x = 2;
console.log(obj.x, proto.x, Object.hasOwn(obj, "x"));
delete obj.x;
console.log(obj.x);

// 5
function A() {}
A.prototype.list = [];
const a1 = new A(), a2 = new A();
a1.list.push(1);
console.log(a2.list);

// 6
"use strict";
function User(name) { this.name = name; }
try { User("A"); } catch (e) { console.log(e.name); }

// 7
function F() { this.a = 1; return { b: 2 }; }
function G() { this.a = 1; return 5; }
console.log(new F(), new G());

// 8
try { new C(); } catch (e) { console.log(e.name); }
class C {}

// 9
class K {}
try { K(); } catch (e) { console.log(e.name); }

// 10
class P { m() {} }
console.log(Object.keys(P.prototype), typeof P);

// 11
class A1 { constructor() { console.log("A"); } }
class B1 extends A1 {
  constructor() { console.log("B1"); super(); console.log("B2"); }
}
new B1();

// 12
class A2 { constructor() { this.init(); } init() { console.log("A"); } }
class B2 extends A2 { x = 1; init() { console.log(this.x); } }
new B2();

// 13
class T { get v() { return 1; } }
const t = new T();
try { t.v = 5; } catch (e) { console.log(e.name); }
console.log(t.v);

// 14
class U { constructor() { this.a = 1; } get b() { return 2; } }
const lit = { a: 1, get b() { return 2; } };
console.log(JSON.stringify(new U()), JSON.stringify(lit));

// 15
class Acc { #bal = 10; get bal() { return this.#bal; } }
const acc = new Acc();
console.log(acc.bal, acc["#bal"], Object.keys(acc));

// 16
class Base { static create() { return new this(); } }
class Child extends Base {}
console.log(Child.create() instanceof Child);

// 17
console.log([] instanceof Array, [] instanceof Object, Object.create(null) instanceof Object);

// 18
class S1 { static x = 1; }
class S2 extends S1 {}
S2.x = 2;
console.log(S1.x, S2.x, Object.getPrototypeOf(S2) === S1);

// 19
class Cnt {
  n = 1;
  reg() { return this?.n; }
  arrow = () => this.n;
}
const { reg, arrow } = new Cnt();
console.log(reg(), arrow());

// 20
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return this.name + " speaks"; };
function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
const d = new Dog("Rex");
console.log(d.speak(), d.constructor === Animal, d instanceof Animal);
```

<details>
<summary>Answers</summary>

1. `"hi A" false true`
2. `true true true`
3. `null`
4. `2 1 true`, then `1` (deleting the own property reveals the inherited one)
5. `[1]` (the array lives on the shared prototype)
6. `TypeError` (`this` is `undefined` in strict mode)
7. `{ b: 2 } G { a: 1 }` (an object return replaces the instance; a primitive return is ignored)
8. `ReferenceError` (classes are in the TDZ)
9. `TypeError` (class constructor cannot be invoked without `new`)
10. `[] "function"` (class methods are non-enumerable)
11. `B1`, `A`, `B2`
12. `undefined` (B's field `x` isn't initialized until `super()` returns)
13. `TypeError`, then `1`
14. `{"a":1}` and `{"a":1,"b":2}`
15. `10 undefined []`
16. `true` (`this` in a static method is the class it was called on)
17. `true true false`
18. `1 2 true` (assigning `S2.x` creates its own static, and `S1.x` is unchanged)
19. `undefined 1` (the method loses `this`; the arrow field is bound)
20. `"Rex speaks" true true` (`constructor` still points to `Animal` because it wasn't restored)

</details>

---

## 8. Coding practice

**1. Implement `new`**
```js
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);
  const result = Constructor.apply(obj, args);
  return result !== null && (typeof result === "object" || typeof result === "function") ? result : obj;
}
```

**2. Implement `instanceof`**
```js
function myInstanceOf(obj, Ctor) {
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) return false;
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    if (proto === Ctor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

**3. `Object.create` polyfill (basic)**
```js
function objectCreate(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}
```

**4. Print an object's prototype chain**
```js
function chain(obj) {
  const names = [];
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    names.push(proto.constructor?.name ?? "(no constructor)");
    proto = Object.getPrototypeOf(proto);
  }
  return names;
}
chain([]);            // ["Array", "Object"]
chain(new (class A extends Map {})());   // ["A", "Map", "Object"]
```

**5. Constructor-function inheritance (Animal → Dog)**
```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} makes a sound`; };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.speak = function () { return `${this.name} barks`; };
```

**6. Class version with polymorphism**
```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}
class Dog extends Animal { speak() { return `${this.name} barks`; } }
class Cat extends Animal { speak() { return `${this.name} meows`; } }
[new Dog("Rex"), new Cat("Tom"), new Animal("Generic")].map(a => a.speak());
```

**7. Bank account with private state, validation, and a getter**
```js
class Account {
  #balance = 0;
  constructor(initial = 0) { this.deposit(initial); }
  get balance() { return this.#balance; }
  deposit(n) {
    if (!(n >= 0)) throw new RangeError("Invalid amount");
    this.#balance += n;
    return this;
  }
  withdraw(n) {
    if (!(n > 0) || n > this.#balance) throw new RangeError("Invalid amount");
    this.#balance -= n;
    return this;
  }
}
```

**8. Singleton**
```js
class Config {
  static #instance;
  constructor() {
    if (Config.#instance) return Config.#instance;
    this.values = {};
    Config.#instance = this;
  }
}
new Config() === new Config();   // true

// Module-based singleton (simplest): export one instance
// export const config = new Config();
```

**9. Count instances with a static field**
```js
class Widget {
  static count = 0;
  constructor() { Widget.count++; }
  static reset() { Widget.count = 0; }
}
```

**10. Simple `EventEmitter`**
```js
class EventEmitter {
  #events = new Map();
  on(name, fn) {
    if (!this.#events.has(name)) this.#events.set(name, new Set());
    this.#events.get(name).add(fn);
    return this;
  }
  off(name, fn) { this.#events.get(name)?.delete(fn); return this; }
  once(name, fn) {
    const wrapper = (...args) => { this.off(name, wrapper); fn(...args); };
    return this.on(name, wrapper);
  }
  emit(name, ...args) {
    this.#events.get(name)?.forEach(fn => fn(...args));
    return this;
  }
}
```

**11. Mixin**
```js
const Serializable = Base => class extends Base {
  serialize() { return JSON.stringify(this); }
};
class User { constructor(name) { this.name = name; } }
class SerializableUser extends Serializable(User) {}
new SerializableUser("Asha").serialize();   // '{"name":"Asha"}'
```

**12. Abstract base class**
```js
class Shape {
  constructor() {
    if (new.target === Shape) throw new TypeError("Cannot instantiate abstract class");
  }
  area() { throw new Error("Not implemented"); }
}
class Rect extends Shape {
  constructor(w, h) { super(); this.w = w; this.h = h; }
  area() { return this.w * this.h; }
}
```

**13. Getter/setter with validation and a computed property**
```js
class Person {
  #first = ""; #last = "";
  constructor(first, last) { this.first = first; this.last = last; }   // goes through the setters
  get first() { return this.#first; }
  set first(v) { if (!v?.trim()) throw new Error("first is required"); this.#first = v.trim(); }
  get last() { return this.#last; }
  set last(v) { if (!v?.trim()) throw new Error("last is required"); this.#last = v.trim(); }
  get fullName() { return `${this.#first} ${this.#last}`; }
  set fullName(v) { [this.first, this.last] = v.split(" "); }
}
```

**14. Fluent (chainable) builder**
```js
class Query {
  #parts = [];
  select(f) { this.#parts.push(`SELECT ${f}`); return this; }
  from(t)   { this.#parts.push(`FROM ${t}`);   return this; }
  where(c)  { this.#parts.push(`WHERE ${c}`);  return this; }
  build()   { return this.#parts.join(" "); }
}
new Query().select("*").from("users").where("id = 1").build();
```

**15. Custom error class**
```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}
try { throw new ValidationError("Email is invalid", "email"); }
catch (e) { console.log(e instanceof ValidationError, e instanceof Error, e.name, e.field); }
```

**16. Share a method safely and keep `this`**
```js
class Timer {
  seconds = 0;
  tick = () => { this.seconds++; };          // arrow field: safe to pass around
  start() { this.id = setInterval(this.tick, 1000); }
  stop()  { clearInterval(this.id); }
}
```

---

## 9. Quick-recall cheat sheet

| Need | Use / remember |
|---|---|
| Object with a chosen prototype | `Object.create(proto)` |
| Read a prototype | `Object.getPrototypeOf(obj)` |
| Own vs inherited property | `Object.hasOwn(obj, k)` vs `k in obj` |
| Dictionary with no inherited keys | `Object.create(null)` (or `Map`) |
| `__proto__` vs `prototype` | Every object has `__proto__`; only constructors have `prototype` |
| Shared methods | Put them on the prototype (or in a class body) |
| Per-instance mutable state | Set it in the constructor, not on the prototype |
| What `new` does | Create → link prototype → run with `this` → return object |
| Guard against a missing `new` | `if (!new.target) return new F(...)` (classes throw automatically) |
| Constructor-function inheritance | `Parent.call(this)` + `Child.prototype = Object.create(Parent.prototype)` + restore `constructor` |
| Class inheritance | `extends` + `super(...)` first in the constructor |
| Call the parent method | `super.method()` |
| Static member | `static x` (on the class, inherited by subclasses, not on instances) |
| Auto-bound method | Arrow-function class field (per instance) or `bind` |
| Abstract-style class | `new.target === Base` check + throwing methods |
| Reuse behavior without deep inheritance | Composition / mixins |
| Read-only or computed property | `get` only |
| Validate on assignment | `set` + private backing field (avoid same-name recursion) |
| True privacy | `#field`, `#method()`, `static #x` |
| Check for a private field | `#field in obj` |
| Class vs function differences | TDZ, strict, needs `new`, non-enumerable methods, `extends` |
| Privacy without classes | Closures (factory functions), `WeakMap` |

---

## Related notes
- [[JS 08 - Async JavaScript]]
- [[JS 10 - Error Handling]]
- [[JS 14 - Advanced Language Features]]
- [[JS 17 - Patterns, Testing & TypeScript]]
