# Architecture

## Layers

Lay out a Next.js app as the Next.js guide of Feature-Sliced Design 2.1 does: put the files that Next.js reads by name in `app/` and in the root of the app, and the rest of the code, other than the configuration files, in the layers of `src/`. Steiger, with `@omni/steiger-config`, and the `next` preset of `@omni/eslint-config` check the layers, the slices, and the imports between them.

| Layer | Holds | Doesn't hold |
| --- | --- | --- |
| `src/_app/` | What the whole app sets up, such as its global styles, its layouts in `layouts/`, the handlers of the `route.ts` files of `app/` in `api-routes/`, and the code of `proxy.ts` in `proxy/` | A screen |
| `src/_pages/` | One slice per screen, or one for several very similar screens, with the code that only it uses | Code that another page also needs |
| `src/widgets/` | One slice per large, self-contained block of UI that several pages reuse, or that is one of several large independent blocks of a page | A block that makes up most of one page and that no other page uses |
| `src/features/` | One slice per main interaction that users want to do with the entities, usually one that several pages reuse, with the UI that performs it | A concept that the app works with |
| `src/entities/` | One slice per business concept that the app works with | What a user does with a record |
| `src/shared/` | The access to the backends, and the code that the whole app reuses, which can be about the business | Business logic |

- Put new code in the slice of the page that uses it, and move it to a lower layer when another slice uses it.
- Make the UI of an entity take what a user does with a record through its props or its children. Keep the cross-import API (`@x`) to what the `model/` of one entity needs of another.

## Slices

A slice groups the code of a layer by what it means to the product.

- Name a page slice after its screen, not after its URL. Name a widget slice after its block. Name a feature slice after its action, starting with the verb, so that it doesn't read as the name of an entity. Name an entity slice after its Omni term.
- Keep in a slice most of the code that its purpose needs, so that a change to one purpose stays in one slice.
- Export from the `index.ts` of a slice only what code outside the slice uses.

## Segments

A segment groups the code of a slice, or of `src/_app/` and `src/shared/`, by its technical nature. Use the segments below, and name any other segment after the purpose of its code.

| Segment | Holds |
| --- | --- |
| `ui/` | What the code displays: components, their styles, and the formatting of values for display, such as a label for each value of a closed set |
| `model/` | The data model and the business logic, such as an input schema or a store |
| `api/` | The interactions with a backend. In a page, a widget, or a feature, these are its Server Actions and the functions that combine several reads of `src/shared/api/` |
| `lib/` | Library code that the other segments need |
| `config/` | Configuration and feature flags |

## Shared

Lay out `src/shared/` as below, with `<term>` the Omni term of the data.

| Path | Holds |
| --- | --- |
| `src/shared/api/<backend>-client.ts` | The client of a backend, with the server environment variables that it reads |
| `src/shared/api/<term>.ts` | The functions that read and change the data of a term through a backend, and the types of what they return |
| `src/shared/api/<term>.schema.ts` | The zod schemas and the closed sets of the data of a term |
| `src/shared/api/index.ts` | The exports that client code can import |
| `src/shared/api/index.server.ts` | The exports of the modules that reach a backend |
| `src/shared/ui/` | The UI kit of the app |
| `src/shared/lib/` | Internal libraries, one folder for each area of focus |

## Server and client

- Mark with `'use client'` only the module at the entry of a client subtree, when it uses state, an effect, a context, an event handler, or a browser API. Everything that a client module imports goes to the browser.
- Move the exports of a slice that only server code can import from its `index.ts` to an `index.server.ts` when the build fails because client code imports the slice.
- Export a module that starts with `'use server'` from the `index.ts`, not from the `index.server.ts`. Client code calls its Server Actions, and Next.js sends the client only a reference to each of them.

## Data access

Reach a backend only in `src/shared/api/`, and call its functions from the rest of the app, so that one place parses every answer and holds the secrets.

- Read data in a Server Component and change it in a Server Action.
- Have the browser send a file that a user uploads straight to storage, with the signed upload form that a function of `src/shared/api/` gets from a backend. A Server Action holds the whole file in the memory of the server, and its upload reports no progress.
- Write a function of `src/shared/api/` that changes data through the client of a backend in a module that doesn't start with `'use server'`, so that the browser can't call it with arguments of its own.
- Parse the answer of a backend with a zod schema of that answer.
- Return from a function of `src/shared/api/` only the fields that the app shows or sends back. The props that a Server Component passes to a client component and the value that a Server Action returns reach the browser, so a field that no screen shows still reaches the user.
- Draw the screen again with the data of the backend after every change that the backend may have made, including one whose answer didn't come or couldn't be read, so that the user doesn't make the change again.
- Parse what a request brings with a zod schema before using it: the `params` and `searchParams` of a route, its headers and cookies, the body of a request to a Route Handler, and the arguments of a Server Action. Anyone can call a Server Action with a direct POST request and any arguments.

## Failures and loading

- Have each function of `src/shared/api/` that reads a record tell the screen when the backend's answer can mean that the record doesn't exist, such as an answer that the backend gives alike for a record that doesn't exist and for one that the user may not see.
- In a page, call `notFound()` when that function tells it that the record that the page's route names may not exist. Read that record in the page itself, outside any `<Suspense>` boundary, because a page that has started to stream answers with status 200 even when it then calls `notFound()`.
- Show any other failed read as a message in place of what the read would show. Keep showing the parts of the screen that don't need the read.
- Put a `<Suspense>` boundary around each Server Component that awaits a read that the rest of the page doesn't need, so that the rest of the page shows first. Give the boundary a fallback of about the size of what the component shows.
