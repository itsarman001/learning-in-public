# useTransition & useDeferredValue

## TL;DR

Both hooks mark a **render** as non-urgent so React can keep urgent updates (typing, clicking) responsive. React renders the urgent update first, then renders the slow one in the background and **discards it if newer input arrives**.

> They make slow _renders_ interruptible. They do **not** make slow _JavaScript_ faster.

---

## Urgent vs. non-urgent updates

| Update type             | Examples                                         | Behavior                                   |
| ----------------------- | ------------------------------------------------ | ------------------------------------------ |
| Urgent                  | Typing, clicking, pressing                       | Rendered immediately, can't be interrupted |
| Transition (non-urgent) | Filtering a big list, switching tabs, navigation | Rendered at lower priority, interruptible  |

---

## useTransition

```tsx
const [isPending, startTransition] = useTransition();
```

- `isPending`: `true` while the transition render is in progress
- `startTransition(fn)`: any state updates called inside `fn` become non-urgent

### Example

```tsx
import { useState, useTransition } from "react";

const ITEMS = Array.from({ length: 20_000 }, (_, i) => `Item ${i}`);

export const UseTransitionDemo = () => {
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState("");
  const [isPending, startTransition] = useTransition();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    startTransition(() => setFilter(value));
  };

  const visible = ITEMS.filter((item) => item.includes(filter));

  return (
    <div className="space-y-3">
      <input value={query} onChange={handleChange} />
      {isPending && <p>Updating list...</p>}
      <ul style={{ opacity: isPending ? 0.5 : 1 }}>
        {visible.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
};
```

**Why it works:** the input state (`query`) renders first, so typing never lags. The heavy list render (`filter`) happens in the background and restarts if you type again.

### Common mistake (wrong usage)

```tsx
const getTotal = async () => {
  const result = await expensiveCalculation(); // blocks the main thread here
  startTransition(() => setTotal(result)); // only wraps a trivial update
};
```

- `async` does **not** make synchronous code non-blocking; the `for` loop still freezes the UI
- The transition wraps `setTotal`, which is cheap; the slow work happened _before_ the transition
- `isPending` flips for only a moment, so "Calculating..." never paints
- **Fix for pure computation:** `useMemo`, chunking the work, or a Web Worker

---

## useDeferredValue

```tsx
const deferredValue = useDeferredValue(value);
```

Returns a copy of `value` that **lags behind**. React first re-renders with the old value (fast), then re-renders in the background with the new one.

### Example

```tsx
import { memo, useDeferredValue, useState } from "react";

const SlowList = memo(({ query }: { query: string }) => {
  const items = ITEMS.filter((i) => i.includes(query));
  return (
    <ul>
      {items.map((i) => (
        <li key={i}>{i}</li>
      ))}
    </ul>
  );
});

export const UseDeferredValueDemo = () => {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SlowList query={deferredQuery} />
      </div>
    </>
  );
};
```

- **`memo` on the child is essential.** Without it the child re-renders on every parent render and you gain nothing.
- Staleness check: `value !== deferredValue`
- React 19 adds an optional initial value: `useDeferredValue(value, initialValue)`

---

## Comparison

|                            | `useTransition`                               | `useDeferredValue`            |
| -------------------------- | --------------------------------------------- | ----------------------------- |
| You wrap                   | The state **update**                          | The **value**                 |
| Needs access to the setter | Yes                                           | No                            |
| Pending signal             | `isPending`                                   | `value !== deferredValue`     |
| Best for                   | Your own handlers (tabs, navigation, filters) | Props or values you don't own |
| Interruptible render       | Yes                                           | Yes                           |
| Child needs `memo`         | No                                            | Yes                           |

---

## Gotchas

- **Not debounce or throttle:** no fixed delay, and total work doesn't shrink. React only schedules it at lower priority and can discard stale renders.
- **Don't put controlled input state inside a transition.** The input will lag or flicker. Keep it urgent.
- **React 18:** the `startTransition` callback must be synchronous.
- **React 19:** async functions are supported, but state updates after an `await` still need their own `startTransition`.
- **Suspense:** transitions don't re-show fallbacks for content already visible. React keeps the old UI and sets `isPending`.
- **Measure first.** If the render isn't slow, these hooks only add complexity. Use the React Profiler, and try `memo` or the React Compiler first.

---

## Interview Q&A

**Q: What problem does `useTransition` solve?** A: It marks a state update as non-urgent so React keeps urgent updates (typing, clicking) responsive and can interrupt the slow render if newer input arrives.

**Q: `useTransition` vs. `useDeferredValue`?** A: Same underlying mechanism. `useTransition` wraps the update when you own the setter; `useDeferredValue` defers a value you receive.

**Q: Does `useTransition` make an expensive calculation faster?** A: No. It only changes render scheduling. Synchronous heavy JS still blocks the main thread, so use `useMemo` or a Web Worker.

**Q: Why does `useDeferredValue` need `memo` on the child?** A: The first render passes the old deferred value. Without `memo`, the child re-renders anyway on the parent's urgent render, so nothing is saved.

**Q: How is this different from debouncing?** A: Debounce waits a fixed time and drops intermediate calls. Transitions start immediately, adapt to device speed, and are interruptible.
