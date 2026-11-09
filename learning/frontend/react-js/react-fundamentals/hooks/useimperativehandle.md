## useImperativeHandle

`useImperativeHandle` is a React Hook that lets a child component **choose which methods and values to expose to its parent through a `ref`**, instead of exposing the whole DOM element.

### Syntax

```tsx
useImperativeHandle(ref, () => ({ /* exposed methods */ }), [deps]);
```

- **`ref`:** The ref passed down from the parent.
- **Create function:** Returns the object the parent will receive in `ref.current`.
- **`deps` (optional):** When it changes, React re-creates the handle.

### How It Works

- **Parent Creates the Ref:** The parent calls `useRef<InputHandle>(null)` and passes it to the child.
- **Child Defines the Handle:** The child calls `useImperativeHandle` and returns only the methods it wants to share.
- **Parent Calls the Methods:** `inputRef.current?.focus()` runs the child's function, not the DOM's.
- **Controlled API:** The parent can't touch anything the child didn't expose.

### Without vs With

| |Plain `ref` on `<input>`|`useImperativeHandle`|
|---|---|---|
|What the parent gets|The full DOM node|Only the methods you expose|
|Access to child's state|No|Yes, through exposed methods (e.g. `clear`)|
|Encapsulation|Low|High|

### When to Use It

- **Imperative Actions:** Things that can't be expressed as props, like `focus()`, `scrollTo()`, `play()`, or `reset()`.
- **Exposing Custom Methods:** Actions that change the child's internal state (like `clear` here).
- **Reusable Components:** Input, modal, or video wrappers where you want a small, safe public API.

### When NOT to Use It

- **State Can Be a Prop:** If the action can be a prop (`isOpen`, `value`), use props. Declarative code is easier to follow.
- **Exposing the Whole DOM Node:** If the parent just needs the element, pass a normal ref.
- **Replacing Data Flow:** Don't use it to push data between components. Use props, state, or context.
- **Overusing It:** Every exposed method makes the child harder to change.

### Example

```tsx
import { Button } from "@/components/ui/button";
import {
  UseImperativeInput,
  type InputHandle,
} from "@/components/useImperativeHandle/use-imperative-input";
import { useRef } from "react";

export const UseImperativeParent = () => {
  const inputRef = useRef<InputHandle>(null);

  return (
    <div className="space-y-3">
      <div>UseImperativeHandle</div>

      <UseImperativeInput ref={inputRef} />

      <div className="space-x-2">
        <Button onClick={() => inputRef.current?.focus()}>Focus</Button>

        <Button onClick={() => inputRef.current?.clear()}>Clear</Button>
      </div>
    </div>
  );
};
```

```tsx
import { useState, useRef, useImperativeHandle, type Ref } from "react";

export type InputHandle = {
  focus: () => void;
  clear: () => void;
};

type Props = {
  ref: Ref<InputHandle>;
};

export const UseImperativeInput = ({ ref }: Props) => {
  const [value, setValue] = useState<string>("");
  const inputRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
    clear: () => setValue(""),
  }), []);

  return (
    <input
      type="text"
      className="rounded-md border border-gray-300 px-3 py-2 mb-4 outline-none focus:border-blue-500"
      value={value}
      onChange={(e) => setValue(e.target.value)}
      ref={inputRef}
    />
  );
};
```

Click **Focus** and the input gets focus. Click **Clear** and the input's state is reset. The parent never touches the `<input>` or the `value` state directly, only the two methods in `InputHandle`.

> **Note:** `ref` is only a regular prop for function components in **React 19**. In React 18 and earlier, wrap the child in `forwardRef`.

### Gotchas

- **Export `InputHandle`:** In your child file, `type InputHandle` has no `export`, but the parent imports it. TypeScript will error. Use `export type InputHandle = ...` (fixed in the example above).
- **Dependency array:** Without one, the handle is re-created on every render. That's fine here, but you can add `[]` since `setValue` and `inputRef` are stable. If a method reads props or state, add them to `deps` or it will use stale values.
- **Optional chaining:** `inputRef.current?.focus()` is needed because the ref is `null` until the child mounts.
- **Keep the API small:** Expose actions (`focus`, `clear`), not state setters or raw DOM nodes.