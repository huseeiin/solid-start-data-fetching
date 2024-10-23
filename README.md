# Data Fetching

Data fetching can be complex and has evolved over the years. `createResource` primitve is the most basic way to fetch data. But this is often not enough. We recommend reading previous docs: [Data Loading](https://docs.solidjs.com/solid-start/building-your-application/data-loading)

## Data fetching in components

Any component can have data. not just routes.

```tsx
import { createAsync } from "@solidjs/router";

export function User() {
  const users = createAsync(async () => {
    "use server";

    return db.query.users();
  });

  return <h1>{users().length} users</h1>;
}
```

If the data is not instant, you need to wrap it in a `<Suspense>` boundary:

```tsx
<Suspense fallback={<p>Loading...</p>}>
  <h1>{users().length} users</h1>
</Suspense>
```

The `fallback` will be visible until the data has finished loading.

**TIP**: remove `"use server"` to load data on the client. but usually your data is on the server and you wanna keep that.

The return type of `createAsync` is the same as the return type of the function that was passed to it.

Make sure every function passed to `createAsync` is async or you might get a type error in your editor.

## Data fetching in routes

```tsx
import { cache, createAsync, redirect, revalidate } from "@solidjs/router";

const getUser = cache(() => {
  const user = db.query.userById(id);

  if (!user) throw redirect("/login");
}, "user");

export const route = { preload: () => getUser() };

export default function Home() {
  const user = createAsync(async () => getUser());

  return (
    <main>
      <h1>Hello, {user().name}</h1>
      <button
        onClick={() => {
          revalidate("user");
        }}
      >
        Revalidate user data
      </button>
    </main>
  );
}
```

**TIP**: you can `throw` or `return` a `redirect` anywhere in your `cache` function to conditionally redirect to a different page.

`cache`, signified by its name, is a useful utility to cache your data for 5-10 seconds. No network call is gonna be made during that time!

`revalidate` gives you the ability to revalidate that data before the end of the 5-10 seconds duration.

You can also set `deferStream` to `true` in the second argument to `createAsync` to force server side rendering to wait until the promise is resolved.
