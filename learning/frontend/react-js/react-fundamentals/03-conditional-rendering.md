# Conditional Rendering

Conditional rendering is how we make our components show different content based on different conditions:

- Show a login button if the user isn't logged in
- Show their profile if they are logged in
- Show an error message only when there's an error

## 4 Ways to Do Conditional Rendering

### 1. If/Else Statements

Great for completely different renders, or returning `null`.

### 2. The Ternary Operator

```jsx
let onlineStatus = isOnline ? "🟢 Online" : "🔴 Offline";
return (
  <div>
    <span>{onlineStatus}</span>
  </div>
);
```

Perfect for either/or situations.

### 3. The AND Operator (`&&`)

Ideal for show/hide scenarios — but watch out for falsy numbers being rendered:

```jsx
const messages = ["a", "b", "c", "d"];

// If messages.length === 0, React renders "0" — a common gotcha
{messages.length && <span>{messages.length} Messages</span>}

// Be explicit instead:
{messages.length > 0 && <span>{messages.length} Messages</span>}
```

### 4. Variables for Complex Logic

Best for complex logic that would make your JSX messy.

## Summary

| Technique | Best for |
| --- | --- |
| If Statements | Completely different renders, or returning `null` |
| Ternary Operator (`?:`) | Either/or situations |
| AND Operator (`&&`) | Show/hide scenarios |
| Variables | Complex logic that would clutter your JSX |

> Also worth knowing: the **Activity Component** (introduced in React 19.2) is a newer option for conditional rendering.

---
#react #conditional-rendering
