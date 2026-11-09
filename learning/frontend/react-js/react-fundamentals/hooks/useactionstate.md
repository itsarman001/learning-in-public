## useActionState

`useActionState` is a React 19 Hook that manages the **state of a form action** in one place. It runs your action when the form is submitted, stores whatever the action returns, and tells you whether it's still pending.

### Syntax

```tsx
const [state, formAction, isPending] = useActionState(action, initialState);
```

|Part|What it is|
|---|---|
|`action`|`(prevState, formData) => newState`, can be `async`|
|`initialState`|The value of `state` before the form is ever submitted|
|`state`|The latest value returned by the action|
|`formAction`|Pass this to `<form action={formAction}>`|
|`isPending`|`true` while the action is running|

### How It Works

- **Form Submit:** React calls your action with the previous state and the form's `FormData`.
- **Return Value Becomes State:** Whatever the action returns (success data, an error message) becomes the new `state`.
- **Pending Flag:** `isPending` is `true` until the action finishes, so you don't need a manual `isSubmitting`.
- **No `preventDefault`:** React handles the submit for you when you use `action={formAction}`.

### The Problem

A simple async form needs a lot of manual wiring: a `useState` for every input, plus `error`, `user`, and `isSubmitting`, and a `try/catch/finally` that sets them all in the right order.

### Without vs With

||Without `useActionState`|With `useActionState`|
|---|---|---|
|Inputs|Controlled (`value` + `onChange` for each)|Uncontrolled (`name` + `FormData`)|
|Result and error|Separate `useState`s|One `state` object|
|Loading flag|Manual `isSubmitting`|`isPending` provided|
|Submit handler|`onSubmit` + `event.preventDefault()`|`action={formAction}`|
|`try/catch/finally`|In the component|Inside the action; just return the result|

### When to Use It

- **Form Submissions:** Login, signup, contact forms, anything that submits and shows a result or error.
- **Server Actions:** Works with Next.js Server Actions as well as client-side async functions.
- **Replacing Boilerplate:** When you have several `useState`s that only exist to track one submit.

### When NOT to Use It

- **Live Validation:** If you validate on every keystroke or need controlled inputs, regular state is a better fit.
- **Non-Form Events:** For a button click or fetch on mount, use `useState`, `useEffect`, or `useTransition`.
- **React 18 or Older:** It only exists in React 19 (it was called `useFormState` in earlier canary releases).
- **Simple Synchronous Updates:** If no async work is involved, `useState` or `useReducer` is simpler.

### Example

#### API (shared by both versions)

```ts
const user = {
  name: "John Doe",
  email: "johndoe@email.com",
  userid: "johndoe",
  password: "1234",
};

type UserCredentials = {
  userid: string;
  password: string;
};

type User = {
  name: string;
  email: string;
  userid: string;
};

type UserApiResponse = {
  success: true;
  message: string;
  user: User;
};

export const userApi = ({
  userid,
  password,
}: UserCredentials): Promise<UserApiResponse> => {
  return new Promise<UserApiResponse>((resolve, reject) => {
    setTimeout(() => {
      if (userid === user.userid && password === user.password) {
        resolve({
          success: true,
          message: "Login successful!",
          user: {
            name: user.name,
            email: user.email,
            userid: user.userid,
          },
        });
      } else {
        reject(new Error("Invalid user ID or password"));
      }
    }, 2000);
  });
};
```

#### Without useActionState

```tsx
import { useState, type FormEvent } from "react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { userApi } from "@/api/user";

type User = {
  name?: string;
  email?: string;
  userid?: string;
};

export const UseActionStateDemo = () => {
  const [user, setUser] = useState<User | null>(null);
  const [userId, setUserId] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);

  async function fetchUser({ userid, password }: { userid: string; password: string }) {
    setIsSubmitting(true);
    setError("");

    try {
      const data = await userApi({ userid, password });
      setUser(data.user);
    } catch (err) {
      setUser(null);
      setError(err instanceof Error ? err.message : "Something went wrong");
    } finally {
      setIsSubmitting(false);
    }
  }

  const handleSubmit = async (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    await fetchUser({ userid: userId, password });
  };

  return (
    <div className="space-y-4">
      <form onSubmit={handleSubmit} className="space-y-3">
        <Input
          type="text"
          placeholder="User ID"
          value={userId}
          onChange={(e) => setUserId(e.target.value)}
        />
        <Input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Submitting..." : "Submit"}
        </Button>
      </form>

      <div>
        {error ? <p className="text-red-500">{error}</p> : null}

        {user ? (
          <>
            <h1>User</h1>
            <h3>Welcome back, {user.name}</h3>
            <p>Email: {user.email}</p>
            <p>User ID: {user.userid}</p>
          </>
        ) : null}
      </div>
    </div>
  );
};
```

#### With useActionState

```tsx
import { useActionState } from "react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { userApi } from "@/api/user";

type User = {
  name: string;
  email: string;
  userid: string;
};

type FormState = {
  user: User | null;
  error: string;
};

const initialState: FormState = { user: null, error: "" };

// The action: receives previous state + FormData, returns the new state
async function loginAction(
  _prevState: FormState,
  formData: FormData,
): Promise<FormState> {
  const userid = formData.get("userid") as string;
  const password = formData.get("password") as string;

  try {
    const data = await userApi({ userid, password });
    return { user: data.user, error: "" };
  } catch (err) {
    return {
      user: null,
      error: err instanceof Error ? err.message : "Something went wrong",
    };
  }
}

export const UseActionStateDemo = () => {
  const [state, formAction, isPending] = useActionState(
    loginAction,
    initialState,
  );

  return (
    <div className="space-y-4">
      <form action={formAction} className="space-y-3">
        <Input name="userid" type="text" placeholder="User ID" />
        <Input name="password" type="password" placeholder="Password" />
        <Button type="submit" disabled={isPending}>
          {isPending ? "Submitting..." : "Submit"}
        </Button>
      </form>

      <div>
        {state.error ? <p className="text-red-500">{state.error}</p> : null}

        {state.user ? (
          <>
            <h1>User</h1>
            <h3>Welcome back, {state.user.name}</h3>
            <p>Email: {state.user.email}</p>
            <p>User ID: {state.user.userid}</p>
          </>
        ) : null}
      </div>
    </div>
  );
};
```

The component goes from five `useState`s, a submit handler, and `preventDefault` to one hook. The `try/catch` moves into the action, and the action returns a result instead of calling setters.

> **Note:** `useActionState` is available in **React 19**. Import it from `"react"`, not `"react-dom"`.

### Gotchas

- **Form resets after submit:** With `<form action={...}>`, React clears uncontrolled inputs when the action finishes, **even on error**. So the user ID field empties after a wrong password. To keep it, return the submitted value in state (`{ ..., userid }`) and set `defaultValue={state.userid}` on the input.
- **Errors are returned, not thrown:** Catch errors inside the action and return them as state. An uncaught throw goes to the nearest error boundary instead of your `state`.
- **Every `name` matters:** Inputs need a `name` attribute, or `formData.get(...)` returns `null`.
- **`formData.get` is untyped:** It returns `FormDataEntryValue | null`, so you need a cast (`as string`) or a check, as in the example.
- **First argument is the previous state:** The action's signature is `(prevState, formData)`. Use `_prevState` if you don't need it.
