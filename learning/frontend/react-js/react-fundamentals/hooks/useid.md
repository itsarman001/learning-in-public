## useId

`useId` is a React Hook that generates a **unique, stable ID string** that you can use for accessibility attributes like `id`, `htmlFor`, and `aria-*`.

### How It Works

- **Generating an ID:** Call `useId()` inside a component. React returns a string that is unique to that component instance.
- **Stable Between Renders:** The ID stays the same across re-renders, so it doesn't change when state updates.
- **Unique per Instance:** If you render the same component multiple times, each instance gets a different ID.
- **Server and Client Match:** The same ID is generated on the server and the client, which avoids hydration mismatches in SSR frameworks like Next.js.

### The Problem

If you hardcode an `id` inside a reusable component, every instance gets the same one. Duplicate IDs are invalid HTML, and clicking a `<label>` may focus the wrong input.

|Approach|Unique per instance|Stable across re-renders|SSR safe|
|---|---|---|---|
|Hardcoded `id="email"`|No|Yes|Yes|
|`Math.random()`|Yes|No|No (hydration mismatch)|
|`useId()`|Yes|Yes|Yes|

### When to Use It

- **Form Inputs:** Linking a `<label>` to an `<input>` using `htmlFor` and `id`.
- **Accessibility Attributes:** Connecting elements with `aria-describedby`, `aria-labelledby`, or `aria-controls`.
- **Reusable Components:** Any component that may appear more than once on a page.

### When NOT to Use It

- **List Keys:** Don't use it as a `key` in a loop. Keys should come from your data (like a database ID), so React can track each item correctly.
- **Data IDs:** Don't use it for IDs you store or send to an API. It's meant for DOM attributes only.
- **CSS Selectors:** Don't rely on its format in `querySelector` or CSS. The generated string can contain special characters and may change between React versions.
- **Needing a Custom Value:** If the caller passes their own `id`, use theirs (see the example below).

### Example

```tsx
import { useId } from "react";
import type { InputHTMLAttributes } from "react";

type InputProps = InputHTMLAttributes<HTMLInputElement> & {
  label?: string;
};

export const Input = ({ label, id, type = "text", ...props }: InputProps) => {
  const generatedId = useId();
  const inputId = id ?? generatedId;

  return (
    <div>
      {label && (
        <label className="mr-2" htmlFor={inputId}>
          {label}
        </label>
      )}

      <input id={inputId} type={type} {...props} />
    </div>
  );
};
```

Each `<Input label="..." />` gets its own ID, so clicking a label focuses the correct input. If the parent passes an `id` prop, that one is used instead.

> **Note:** `useId` should not be used as a `key` in a loop.

### Gotchas

- **Hook is called unconditionally:** `useId()` runs on every render, and `id ?? generatedId` picks which one to use afterwards. This is correct, since hooks can't be called conditionally.
- **Multiple related IDs:** Call `useId` once and add suffixes, for example `` `${generatedId}-error` `` for an error message element, instead of calling the hook several times.
- **React Compiler:** It doesn't replace `useId`. This hook is about unique IDs, not memoization.