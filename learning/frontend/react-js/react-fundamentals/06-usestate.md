# State — `useState`

## The Basics

When you call `setCount`:

1. React updates the state value
2. React re-renders the component
3. `useState` gives us the new value
4. Our UI shows the updated count

```jsx
import { useState } from "react";
import { ActionButton } from "./ActionButton";

export const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <ActionButton onClick={() => setCount((prev) => prev + 1)} text={count} />
  );
};
```

## Lazy Initialization

Useful when you have any expensive computation you need to do to calculate the initial state:

- Reading from local storage
- Fetching data from an API
- Doing any other heavy computation

```jsx
const [state, setState] = useState(() => computeExpensiveInitialValue());
```

## Summary

- To add state to a component, import `useState` from React and call it with an initial value
- You can also pass a function to `useState` for lazy initialization
- It returns an array with two items: `[currentValue, setterFunction]`
- Use array destructuring to name them
- Use the state value in your JSX
- Call the setter function to update state and trigger a re-render
- You can have multiple state variables, each managing its own data
- Multiple instances of a component each have their own local state

## Golden Rules of Hooks

1. **Only call hooks at the top level** of your function — not inside loops, conditions, nested functions, or try/catch blocks
2. **Only call hooks from React functions** — from React components or custom hooks

### Why the Rules of Hooks Exist

> "Why did React make these rules? Why not just let us put hooks wherever?"

The answer is **predictability**. React needs to know that every time your component renders, it will call the exact same hooks in the exact same order.

This lets React:

- Keep track of state between renders
- Know which `useState` call corresponds to which piece of state
- Optimize performance
- Provide helpful error messages

```jsx
import { useState } from "react";

export const UserDashboard = ({ isPremium = false }) => {
  const [credits, setCredits] = useState(100);

  if (!isPremium) return <div>Upgrade to premium to see the credits</div>;

  return (
    <div>
      <p>You have {credits}</p>
      <button onClick={() => setCredits(0)}>Spend all Credits</button>
    </div>
  );
};
```

## How State Updates Work

Updating the UI is a 3-phase process:

1. **Trigger** — This happens the moment you call a state setter like `setCount()`. You're not updating the UI yet.
2. **Render** — React calls your component function again. React figures out which part of the UI, if any, needs to be updated. Re-running the function doesn't immediately change what's on screen.
3. **Commit** — React takes the changes it calculated during the render phase and applies them to the DOM.

```
<p>Count: 0</p>
        |
<p>Count: 1</p>
```

### `setCount` Update, Step by Step

1. You call `setCount(count + 1)` *(trigger phase)*
2. React marks your component as needing an update *(trigger phase)*
3. React calls your component function again *(render phase)*
4. Your function returns new JSX with the updated count *(render phase)*
5. React compares this render with the previous one and figures out what changed *(render phase)*
6. React updates only what changed in the actual DOM *(commit phase)*

```jsx
import { useState } from "react";

export const SimpleCounter = () => {
  const [count, setCount] = useState(0);

  const handleClick = () => setCount((prev) => prev + 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
};
```

## State as a Snapshot

In React, state updates are not immediate. When you call `setState`, you're scheduling an update for the *next* render. Inside your current render, the state values won't change — they're a snapshot, and snapshots don't change. If you log the count inside your event handler, you'll always get the same value.

> Think of it like taking a photo. Once you take the photo, the people in it don't move, even if the real people walk away. State in a render is like that photo — it's frozen at that moment in time.

```jsx
const handleClick = () => {
  console.log("Count value before increment", count); // 0
  setCount((prev) => prev + 1);
  console.log("Count value after increment", count); // still 0
};
```

## Batching

React waits until your event handler finishes, gathers all your state updates, and applies them in one render.

### State Update Cycle

1. Triggering updates
2. The snapshot behavior
3. Using updater functions for dependent updates
4. How React optimizes everything with batching

## `useState` with Objects

```jsx
const [user, setUser] = useState({
  name: "John Wick",
  age: 30,
  email: "johnwick@email.com",
  address: {
    city: "New York",
    country: "USA",
  },
});
```

When your state is an object, always create a **new object** instead of modifying the existing one.

```jsx
setUser({
  ...user,
  name: "John Wick 2",
});
```

For nested objects, spread every level that you're updating:

```jsx
setUser({
  ...user,
  address: {
    ...user.address,
    city: "Los Angeles",
  },
});
```

### Summary

- Treat state as **immutable** — never modify it directly
- Always use the setter function returned by `useState`
- Calling the setter with an object **replaces** the entire object
- Use the spread operator (`...`) to preserve existing properties
- For nested objects, spread both the parent object and the nested object before updating a field
- The spread operator performs a **shallow copy**, so every nested level you update must also be copied

## `useState` with Arrays

- Always use the setter function to update state — changing the array directly won't trigger a re-render
- Spread operator or `concat` to add items
- `filter` to remove items
- `map` to update items

## Sharing State (Lifting State Up)

- When two or more components need the same changing data, lift the state up
- Put that state in the closest common parent
- The component with `useState` owns the data and controls how it changes
- Children receive the state as props
- Children request changes by calling setter functions passed down from the parent

---
#react #hooks #usestate #state
