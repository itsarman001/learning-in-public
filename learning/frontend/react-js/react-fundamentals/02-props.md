# Props

## Summary

- Props are how components talk to each other
- They're like the arguments to a function
- They make components dynamic and truly reusable
- You pass them when you invoke a component
- You access them in the component using the `props` object
- You can pass any JavaScript value, including functions, arrays, objects, and JSX
- Destructuring is a cleaner way to access props

## Patterns

### Default Props

- The default value is only used if the **prop is missing** or if you pass **`undefined`**
- If you pass **`null`** or **`0`**, the default value won't be used

### Forwarding Props with Spread Operator

- Useful for passing along a bunch of props to a child without naming each one individually

### Passing JSX as Children

- In HTML, it's natural to nest elements inside each other
- In React, you can do the exact same thing with components
- Common use cases: Layouts, Cards, Modals, etc.

---
#react #props
