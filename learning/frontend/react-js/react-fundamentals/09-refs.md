# Refs and `useRef`

A ref is a way to store a mutable value that:

- Persists across renders
- Doesn't cause re-renders when it changes

## Summary

Refs are used to store values that don't affect the UI but need to persist across renders.

- **State** is for rendering
- **Refs** are for remembering

We create a ref using the `useRef` hook. It accepts an initial value and returns an object with a single property, `current`, which holds the value of the ref.

Common uses: interval IDs, timeout IDs, previous values, etc.

## Manipulating the DOM with Refs

React is declarative — you describe what the UI should look like, and React updates the DOM for you. But there are certain situations where you need a direct handle to the DOM:

- Focusing an input
- Scrolling an element into view
- Measuring the size or position of an element

Refs are how you get that handle.

```jsx
import { useRef } from "react";

export const FocusInput = () => {
  const inputRef = useRef(null);

  const handleInputFocus = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Focus Ref" />
      <button onClick={handleInputFocus}>Focus Input</button>
    </div>
  );
};
```

## A Note About Refs

1. Refs are an escape hatch
2. A necessary one, but still an escape hatch
3. You should use them sparingly
4. If you find yourself constantly manipulating the DOM, it usually means you're fighting React instead of working with it

---
#react #refs #useref
