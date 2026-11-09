# Prop Drilling and Context

## Prop Drilling

Sending props through layers of components just to reach the ones that actually need them.

### Two Main Problems

1. **Maintenance** — Every time you need to add a new property to `user`, update an existing one, or remove one you no longer need, you have to touch every single component in the chain
2. **Performance** — If a prop changes in a parent, every child that receives that prop re-renders, even if it doesn't use the prop. When `user` updates, all those middle components re-render unnecessarily

## Context

Context lets you share data across your component tree without having to manually pass props through every intermediate component.

- Context lets you broadcast a value at a high level
- Any component nested inside can subscribe to it directly

```jsx
import { createContext, useContext, useState } from "react";

const UserContext = createContext(null);

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState({ name: "John Wick" });
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

// Any nested component can now do:
// const { user } = useContext(UserContext);
```

## The `use` API

In React 19, Meta introduced the `use` API, which works like the Context API and follows the same structure — but it is **not a hook**, so it doesn't need to follow the Rules of Hooks.

In short, `use` is simply a more flexible alternative to `useContext`. It does the same job (reading values from Context) but without the restrictions that come with hooks — for example, it can be called conditionally.

---
#react #context #prop-drilling
