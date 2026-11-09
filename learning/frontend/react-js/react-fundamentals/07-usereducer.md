# `useReducer`

`useReducer` is another hook for state management in React.

The hook accepts two parameters:

- A reducer function
- The initial state

The reducer function itself accepts two parameters:

- The current state
- An action

Based on the inputs, it returns the new state — along with a `dispatch` function. You call `dispatch` to trigger state updates.

## Mechanics

- **Action** describes what happened
- **Dispatch** sends that action to the reducer
- The **reducer** decides how to update state
- The component re-renders with the new state

```jsx
import { useReducer } from "react";

const initialState = 0;

const counterReducer = (state, action) => {
  switch (action) {
    case "increment":
      return state + 1;
    case "decrement":
      return state - 1;
    case "reset":
      return initialState;
    default:
      return state;
  }
};

export const CounterWithReducer = () => {
  const [count, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <div>Count: {count}</div>
      <div>
        <button onClick={() => dispatch("increment")}>Increment</button>
        <button onClick={() => dispatch("decrement")}>Decrement</button>
        <button onClick={() => dispatch("reset")}>Reset</button>
      </div>
    </div>
  );
};
```

## Lazy Initialization

- `useReducer` accepts an optional third argument: an `init` function
- React calls `init` once, passing it the second argument
- Whatever `init` returns becomes the actual initial state
- The `init` function only runs when the component mounts, not on every render
- Ideal for expensive initial computations or reading from external sources like local storage

## `useState` vs. `useReducer`

`useState` is really just a specialized version of `useReducer` where:

- The reducer always returns the action as the new state
- If the action is a function, it calls it with the current state first
- The dispatch function is wrapped in a `setState` function for a nicer API

**Takeaway:** `useReducer` is the most fundamental hook. `useState` is simply a convenience built on top of it for the most common use case — replacing state with a new value.

---
#react #hooks #usereducer
