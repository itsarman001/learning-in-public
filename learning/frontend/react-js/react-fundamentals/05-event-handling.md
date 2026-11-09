# Event Handling

Responding to events in React is a simple two-step process:

1. You define a function that should be executed when the event occurs
2. You assign the function to a special prop that starts with **`on`**

React supports all the events from regular JavaScript:

- `onChange` for inputs
- `onSubmit` for forms
- `onMouseEnter` for hovering

> Note: Event names are written in camelCase.

```jsx
export const AlertButton = () => {
  const handleClick = () => {
    alert("Button Clicked!");
  };

  return (
    <button
      onClick={handleClick}
      className="px-2 py-1 bg-slate-600 text-slate-400 rounded"
    >
      Click Me!
    </button>
  );
};
```

## Summary

- Handling events in React is all about passing functions to special props like **`onClick`**
- Remember to pass the function, not call it (`onClick={handleClick}`, not `onClick={handleClick()}`)
- Use the event object when you need info about the event
- Event handlers have access to all the component's variables and props since they're defined inside the component

## Event Handlers as Props

- When you need a child component to communicate with its parent, you pass event handlers as props
- The child says something happened, and the parent decides what to do about it
- You'll use this pattern every time you build a reusable component that needs different behavior in different places

```jsx
export const ActionButton = ({ text, onClick }) => {
  return <button onClick={() => onClick(text)}>{text}</button>;
};
```

---
#react #events
