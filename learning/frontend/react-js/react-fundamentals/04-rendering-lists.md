# Rendering Lists

A common scenario when building web applications is to display a list of items:

- A list of products in an e-commerce app
- Courses in a learning platform
- Comments on a blog post

To render a list of data in React, take your data, put it in an array, and transform that array into the HTML you need.

Rendering lists in React is just JavaScript: take an array, transform it with `map` into an array of JSX elements, and React renders them all.

## The Key Prop

When you render a list of elements, you need to add a `key` prop to each element with a unique value within the list.

- If your data comes from a database, use the database keys/IDs — they're unique by nature
- If your data is created and persisted locally, use an incrementing counter, `crypto.randomUUID()`, or a package like `uuid`

### Rules of Keys

- The `key` prop goes on the outermost repeated element
- `key` is a special prop that React uses internally — it's not a prop that you pass to a child component and destructure to access

### Why Does React Need Keys?

- Keys are used by React to track the items in the list
- They're crucial for React to efficiently update the UI

### Keys — Summary

- `key` is a special prop that React uses internally to track items in a list
- It needs to be unique within the list
- The `key` prop goes on the outermost repeated element
- Keys are crucial for React to efficiently update the UI

## Using Item Index as Key

You'll see tons of examples where developers use the array index as the key when rendering lists. It looks clean, it's simple, and it makes the warning go away — but there's a reason this is considered an anti-pattern.

### Index-as-Key Anti-Pattern

- The problem with using index as a key is that the index represents the *position*, not the *item itself*
- This can lead to subtle and confusing bugs when the order of items in the list changes

### When Is It Safe to Use Index as a Key?

- Your items don't have a unique ID
- Your list is completely static — you never add or remove items
- Your list is never reordered or filtered

> **Debugging tip:** If you ever see weird behavior in your lists — like data jumping between items, inputs losing focus, or animations glitching — check your keys first.

---
#react #lists #keys
