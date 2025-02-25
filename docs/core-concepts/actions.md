---
section: core-concepts
title: Actions
order: 98
---

# Actions

One question you will likely have when developing any sort of app is "how do I communicate new information to my server?". The user did something. What next? Solid's answer to this is _actions_. Actions give you the ability to specify an async action processing function and gives you elegant tools to help you easily manage and track submissions. 

They generally represent a `POST` request.

Actions are isomorphic. This means that a submission can be handled on the server _or_ the client, which ever is optimal. They represent the server component of an [HTML form](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form), and even help you use HTML forms to submit data.

## Creating actions

Let's stop getting ahead of ourselves! First let's create an action!

```tsx twoslash
import { createRouteAction } from "solid-start/data";

export function MyComponent() {
  const [_, logMessage] = createRouteAction(async (message: string) => {
    // Imagine this is a call to fetch
    await new Promise((resolve, reject) => setTimeout(resolve, 1000));
    console.log(message);
  });
}
```

This `echo` action will act as your backend, however you can substitute it for any API, provided you are ok with it running on the client. Typically, route actions are used with some sort of solution like fetch or graphql, and return either a `Response` such as a redirect (we are not returning anything quite yet!) or any value. If you want to ensure the action only runs on the server for things like databases, you will want to use `createServerAction$`. It's been introduced below.

Naturally, this action won't do anything quite yet. We still need to call it somewhere! For now, let's call it manually from some component using the `submit` method.

```ts twoslash {3,8}
import { createRouteAction } from "solid-start/data";
export function MyComponent() {
  const [, logMessage] = createRouteAction(async (message: string) => {
    // Imagine this is a call to fetch
    await new Promise((resolve, reject) => setTimeout(resolve, 1000));
    console.log(message);
  });
  logMessage("Hello from solid!");
}
```

You should see `Hello from solid!` back in the console!

### Returning from actions

In many cases, after submitting data the server sends some data back as well. Anything returned from your action function can be accessed using the reactive `action.value` property. The value of this property can change each time you submit your action.

```tsx twoslash {2,4,7-9}
import { createRouteAction } from "solid-start/data";
// ---cut---
export function MyComponent() {
  const [echoing, echo] = createRouteAction(async (message: string) => {
    await new Promise((resolve, reject) => setTimeout(resolve, 1000));
    return message;
  });

  echo("Hello from solid!");
  setTimeout(() => echo("This is a second submission!"), 1500);
  return <p>{echoing.result}</p>;
}
```

While this method of using actions works, it leaves the implementation details of how you trigger `echo` up to you. When handling explicit user input, it's better to use a `form` for a multitude of reasons.

## Using forms to submit data

We highly recommend using HTML forms as your method to submit data with actions. HTML forms can be used even before JavaScript loads, leading to instantly interactive applications. They have the added benefit of implicit accessibility, and can save you valuable time that would have otherwise been spent designing a UI library that will never have the aforementioned benefits.

When forms are used to submit actions, the first argument is an instance of [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData). Writing forms using actions is trivial, simply use the `Form` method of your action instead of the normal `<form>` tag, and walk away with amazing, progressively enhanced forms!

If you don't return a `Response` from your action, the user will stay on the same page and your resources will be re-triggered. You can also return a `redirect` or `ResponseError`.

```tsx twoslash
import { createRouteAction } from "solid-start";
import { redirect } from "solid-start/server";
// ---cut---
export function MyComponent() {
  const [_, { Form }] = createRouteAction(async (formData: FormData) => {
    await new Promise((resolve, reject) => setTimeout(resolve, 1000));
    const username = formData.get("username");
    if (username === "admin") {
      return redirect("/admin");
    } else {
      throw new Error("Invalid username");
    }
    return redirect("/home");
  });

  return (
    <Form>
      <label for="username">Username:</label>
      <input type="text" name="username" />
      <input type="submit" value="submit" />
    </Form>
  );
}
```

This `Form` is an enhanced version of the normal `form`. It submits handler has already been wired up as well. 

## Retriggering resources

- retriggers route resources

## Errors

- Errors, error field that's populated if the submission errored, and a status field that's set to `error`
  - if you read the `submissionState.error` field in your code (JSX, or effects) then the error is considered user-handled, and we don't trigger ErrorBoundaries.
  - if you don't use the error field, then we trigger the error boundary on an error because we assume it's unexpected for you
- How to do form errors? Where to put... Here or `ResponseError`?

## Server Actions

Sometimes we need to make sure our action _only_ runs on the server. This is useful for:

- accessing internal APIs
- proxying external APIs
  - To use server secrets
  - To reduce the response payload by postprocessing
  - To bypass CORS
- running code incompatible with browsers
- or even connecting directly to a database (Take caution, opinions on if this is a good idea are mixed. You should consider separating your backend and frontend.)

To do this, simply replace `createRouteAction` with `createServerAction$` and the action will always be run on the server.
