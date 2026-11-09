# Proxy and Reflect

## Proxy

**Definition:** a `Proxy` wraps a **target** object and intercepts operations on it (get, set, delete, etc.) through a **handler** containing **traps**.

```js
const proxy = new Proxy(target, handler);
```

**Rules:**

- `target` and `handler` must both be objects.
- A trap that is not defined falls through to the target (default behavior).
- `proxy !== target`. Operate through the proxy for traps to run.
- The `set` trap must **return truthy**, or the assignment fails.
- `Proxy.revocable(target, handler)` returns `{ proxy, revoke }`.

|Trap|Intercepts|
|---|---|
|`get`|`obj.x`, `obj[x]`|
|`set`|`obj.x = v`|
|`has`|`x in obj`|
|`deleteProperty`|`delete obj.x`|
|`ownKeys`|`Object.keys()`, `for...in`, `Reflect.ownKeys()`|
|`apply`|calling a function proxy|
|`construct`|`new proxy()`|

13 traps in total. The rest cover descriptors, prototype and extensibility.

**Exceptions:**

- **Invariants:** a trap cannot lie about a **non-configurable, non-writable** target property (`TypeError`).
- **Internal slots:** built-ins like `Map`, `Set` and `Date` break through a proxy. Fix: bind methods to the target.
- **Private fields:** `#x` accessed through a proxy `this` → `TypeError`.

**Errors:**

|Case|Error|
|---|---|
|`new Proxy(1, {})`|`TypeError: Cannot create proxy with a non-object as target or handler`|
|`Proxy({}, {})` without `new`|`TypeError: Constructor Proxy requires 'new'`|
|Falsy return from `set` (strict mode)|`TypeError: 'set' on proxy: trap returned falsish for property 'x'` (silently ignored in sloppy mode)|
|Any operation after `revoke()`|`TypeError: Cannot perform 'get' on a proxy that has been revoked`|
|`new Proxy(new Map(), {}).get(1)`|`TypeError: Method Map.prototype.get called on incompatible receiver`|

---

## Reflect

**Definition:** `Reflect` is a built-in **namespace object** (not a constructor, not callable) with static methods that match every Proxy trap, one for one, with the same arguments.

**Rules:**

- `Reflect.get/set/has/deleteProperty/ownKeys/apply/construct...` perform the **default behavior** of the operation.
- It is the functional form of operators: `Reflect.has(o, k)` = `k in o`, `Reflect.construct()` = `new`.
- `Reflect.set`, `defineProperty` and `deleteProperty` **return a boolean** instead of throwing. `Object.defineProperty` throws on failure.

**Why use `Reflect` inside traps:**

- Forwards the default behavior correctly.
- Passing **`receiver`** keeps `this` correct inside getters/setters and with inheritance.
- Its boolean return can be returned directly from the `set` trap.

**Errors:**

- `Reflect.get(1, "a")` → `TypeError: Reflect.get called on non-object`
- `new Reflect()` → `TypeError: Reflect is not a constructor`

---

## Example: Validation Proxy

```js
const person = { fullName: "John Doe", age: 19 };

const handler = {
  get(target, prop, receiver) {
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    if (!Object.hasOwn(target, prop)) return false; // unknown property
    switch (property) {
      case "fullName":
        if (typeof value !== "string")
          throw new Error("Full Name must be a string");

        break;

      case "age":
        if (typeof value !== "number") throw new Error("Age must be a number");

        if (value <= 0 || value >= 100)
          throw new Error("Age must be between 0 and 100");

        break;

      default:
        break;
    }
    return Reflect.set(target, prop, value, receiver); // default behavior
  },
};

const p = new Proxy(person, handler);
p.age = -99; // Error: Age must be between 0 and 100 (age stays 19)
```

**Edge cases:**

- `return false` from `set` throws a `TypeError` in strict mode and fails silently in sloppy mode.
- `prop in target` also checks the **prototype chain** (`"toString" in target` is `true`). Use `Object.hasOwn()` for own properties only.
- `prop` can be a **Symbol**. `${prop}` in a template literal throws `TypeError: Cannot convert a Symbol value to a string`.
- Check boundaries carefully: `value <= 0` rejects `0`.

---

## Quick Interview Answers

- **What is a Proxy?** An interceptor layer around an object, defined by a handler with traps.
- **What is Reflect?** A namespace of static methods mirroring the traps, giving default behavior.
- **Why `Reflect` inside a trap?** Correct default behavior, correct `this` via `receiver`, boolean return values.
- **Proxy use cases?** Validation, logging, access control, reactivity (Vue 3), default values.
- **Proxy vs `Object.defineProperty`?** Proxy intercepts all operations, including new properties, `delete` and `in`. `defineProperty` only covers properties you define up front.