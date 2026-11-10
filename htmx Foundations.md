[**htmx Foundations**TypeScript · Hono · htmx 2](#overview)

0 of 8 chapters complete

About 30 minutes in total. Your progress is saved in this browser only.

Free course · \~30 minutes

# Learn htmx by building a task list

Build a tiny task list with inline editing, live search and real-time updates, without writing any client-side JavaScript. Every interaction is an HTTP request that returns HTML.

**8** chapters**8** quizzesStack: **TypeScript, Hono, htmx 2.0.11**

## What you'll build

A one-page task list served by a small TypeScript server. You'll add inline editing, adding and deleting, a search box, a live counter, boosted navigation, a Server-Sent Events activity feed and a WebSocket notes panel.

Each chapter has a **live demo** running real htmx in your browser. Its requests go to a small simulated server built into this page, and a request log under each demo shows every request and the HTML that came back.

## Chapters

> **Before you start:**
> 
> - You'll need Node.js 20 or later.
> - You should be comfortable with HTML and TypeScript. No htmx knowledge is needed.

[Start Chapter 1](#chapter-1)

Chapter 1

# The Big Idea

htmx is a small library that extends HTML. It lets *any* element send an HTTP request and put the HTML that comes back somewhere on the page. That's the whole library; the rest of this course is details.

In this chapter, here are the topics we'll cover

- HATEOAS: the idea htmx is built on.
- How it differs from a JSON API plus a front-end framework.
- Setting up a TypeScript server with Hono.

## Hypermedia as the engine of application state

HATEOAS stands for **Hypermedia As The Engine Of Application State**. It describes how the web worked before single-page apps:

- **The server owns the state.** The truth lives in your database, not in a client-side store.
- **Responses are hypermedia.** The server sends HTML, not data. That HTML describes the current state *and* includes the actions available next, as links, forms and buttons.
- **The client is generic.** The browser doesn't need to know your API or your business rules. It renders what it's given and follows the controls in it.

Plain HTML only lets `<a>` and `<form>` make requests, and every request replaces the whole page. htmx removes those two limits. Here's one round trip in the app you're about to build:

**Browser**User clicks `Save` on\
`<form hx-put="/tasks/1">`

PUT /tasks/1 →← 200 text/html

**Server**Saves the title and returns\
`<li>…<button hx-get="/tasks/1/edit">Edit</button></li>`

Notice what the client *didn't* do: parse JSON, update a store, or decide which buttons to show. The response already contains the next actions. Keep this in mind; we'll come back to it in Chapter 8.

## Setting up the project

htmx works with any server that returns HTML. We'll use [Hono](https://hono.dev) on Node.js: it's typed end to end, its JSX templates escape output by default, and it supports Server-Sent Events and WebSockets.

`"types": ["node"]` makes the Node.js type definitions available, so imports like `node:events` resolve in your editor. Recent TypeScript versions don't include them automatically.

> **If you're a Next.js developer:**
> 
> - JSX here is *server-only* templating, like a Server Component that renders to a string. There's no hydration and no client bundle.
> - You could return HTML fragments from Next.js route handlers, but they can't host WebSockets, which Chapter 7 needs. A small dedicated server keeps things simpler.

### An in-memory store

The server owns the state, so here it is. The `bus` event emitter will power live updates in Chapter 7.

### Views and the first page

htmx itself is one `<script>` tag. We'll also load the SSE and WebSocket extensions now so the layout doesn't change later.

### Styles

htmx doesn't care how things look, but a few styles make the demo easier to follow. They also include the `.done` strike-through you'll use in Chapter 4. Create this file and the server above will serve it:

Run `npm run dev` and open `http://localhost:3000`. You'll see three tasks rendered on the server and styled like the demos in this course. Nothing interactive yet.

> **Which htmx version?**
> 
> - This course uses **htmx 2.0.11**, which is still the default (`latest`) version on npm.
> - htmx 4.0 was released in August 2026 under the `next` tag. It switches to `fetch()`, builds SSE into the core, and makes attribute inheritance opt-in. The core ideas in this course carry over unchanged.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

In a HATEOAS application, how does the browser know that a task can be edited?

The available actions travel inside the hypermedia. If the server doesn't render an Edit button, there is no way to edit, and the client needs no rules of its own.

Chapter 2

# Requests & Swaps

Every htmx interaction answers three questions: **what request** to send, **where** the response goes, and **how** it's inserted. You'll answer all three by building inline editing.

In this chapter, here are the topics we'll cover

- Sending requests with `hx-get` and `hx-put`.
- Choosing where the response goes with `hx-target`.
- Choosing how it's inserted with `hx-swap`: `innerHTML` vs `outerHTML`.

## The request attributes

htmx has one attribute per HTTP method: `hx-get`, `hx-post`, `hx-put`, `hx-patch` and `hx-delete`. Put one on any element and it sends that request when triggered. A button triggers on click; a form triggers on submit and sends its fields.

## Where and how: hx-target and hx-swap

- **`hx-target`** takes a CSS selector for the element to update. By default it's the element that made the request. It also understands relative selectors like `closest li`, `next .panel` and `this`.
- **`hx-swap`** says how to insert the response relative to the target. The default is `innerHTML`.

| hx-swap | What happens to the target | Used in this course for |
| --- | --- | --- |
| `innerHTML` | Its children are replaced; the element itself stays | Search results (Chapter 4) |
| `outerHTML` | The whole element is replaced by the response | Swapping a row between view and edit modes |
| `beforeend` | Response is appended as its last child | Adding a task (Chapter 3) |
| `afterbegin` | Response is inserted as its first child | Newest-first activity feed (Chapter 7) |
| `beforebegin` / `afterend` | Inserted as a sibling before or after it |  |
| `delete` / `none` | Target is removed / nothing is swapped |  |

## Building inline edit

A row has two states, and the server renders each one. The display row gets an Edit button that fetches the edit form and swaps it in *over* the row:

Each of those requests needs a route that returns the right fragment:

Here's the whole cycle: **Edit** sends `GET /tasks/1/edit`, and the form replaces the row. **Save** sends the form fields with `PUT /tasks/1`, and the updated row replaces the form. **Cancel** fetches the unchanged display row. The server never sends JSON, and the browser never holds the task in memory.

> **Why `outerHTML`?**
> 
> - The response is a complete `<li id="task-1">`. With `innerHTML` it would land *inside* the existing `<li>`, giving you a nested list item with a duplicate ID. When a response is a whole element, replace the whole element.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

A button has `hx-get="/tasks/1/edit"` and `hx-target="#task-1"`, but no `hx-swap`. The server returns `<li id="task-1">…</li>`. What ends up on the page?

The default swap is `innerHTML`, which keeps the target and replaces its children. Use `outerHTML` when the response is the whole element.

Chapter 3

# Adding & Deleting

You can edit tasks. Now let's create and remove them, which introduces two more methods and a new swap style.

In this chapter, here are the topics we'll cover

- Appending to a list with `hx-post` and `hx-swap="beforeend"`.
- Resetting a form after a request with `hx-on`.
- Removing an element with `hx-delete`.

## Adding with hx-post

The add form posts to `/tasks`. The server returns just the new row, and `beforeend` appends it to the end of the list:

`hx-on` runs a little inline script when an htmx event fires. In HTML you'd write `hx-on::after-request`; JSX doesn't allow `::` in attribute names, so htmx also accepts the `hx-on--after-request` form.

## Deleting with hx-delete

Add a Delete button to `TaskRow`. It targets its own row with `outerHTML`, and the server returns an empty body, so the row is replaced with nothing:

> **Good to know:**
> 
> - Return `200`, not `204`. htmx doesn't swap on `204 No Content`, so the row would stay.
> - In htmx 2, `DELETE` sends parameters in the query string like `GET`, not in a body.
> - Add `hx-confirm="Delete this task?"` to ask before sending the request.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

You want new tasks to appear at the *top* of the list instead. What do you change?

`afterbegin` inserts as the target's first child. `beforebegin` would put the row before the `<ul>` itself, outside the list. Option D works too, but sends far more HTML than needed.

Chapter 4

# Triggers

So far requests fired on the natural event: a click or a form submit. `hx-trigger` lets you choose any event and add modifiers to it.

In this chapter, here are the topics we'll cover

- The default trigger for each element.
- Toggling a task on `change`.
- Live search with `keyup changed delay:300ms`.
- Polling with `every 2s`.

## Default triggers

Without `hx-trigger`, forms fire on `submit`; `input`, `select` and `textarea` fire on `change`; everything else fires on `click`.

## A checkbox on change

Add a checkbox to `TaskRow` that toggles the task. `change` is already the default for inputs; it's written out here so you can see it. While you're there, only show Edit for tasks that aren't done. We'll come back to why that line matters in Chapter 8.

## Live search

The search box sends `GET /tasks?q=…` as you type and replaces the *contents* of the list, so `innerHTML` is right here:

Reading the trigger from left to right:

- **`keyup`**: listen for key releases.
- **`changed`**: only fire if the value actually changed, so arrow keys and Shift don't send requests.
- **`delay:300ms`**: wait until typing pauses for 300ms. Each new keyup restarts the timer (a debounce).
- **`, search`**: a second trigger. Browsers fire `search` when you clear the box with its ✕ button.

Because the element has `name="q"`, htmx includes its value as a query parameter.

## Polling

`every 2s` fires on a timer. It's the simplest way to keep something fresh:

> **Other useful triggers:**
> 
> - `load` fires once when the element is added; `revealed` fires when it scrolls into view (good for infinite scroll).
> - `once`, `throttle:1s` and `from:body` are more modifiers you can combine.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

What does `changed` add to `keyup changed delay:300ms`?

`changed` compares the value with the last one sent. Keys like arrows, Tab and Shift produce keyups without changing the value, so no request is made.

Chapter 5

# Out-of-Band Swaps

Each request has one target, but one action often changes several things. Ticking a task changes its row *and* the "open" count in the heading. Out-of-band swaps handle this.

In this chapter, here are the topics we'll cover

- How `hx-swap-oob` works.
- Adding a live "open" count to the heading.
- Removing a row and updating the count from one response.

## How it works

Mark any top-level element in a response with `hx-swap-oob="true"` and an `id`. htmx pulls it out of the response and swaps it into the element on the page with the same `id`. Everything else in the response goes to the normal target as usual.

Render it normally in the heading, then add an out-of-band copy to every response that changes the count:

The delete response is a neat trick: after htmx removes the out-of-band span, the main content is empty, so the row is still replaced with nothing.

> **More forms of hx-swap-oob:**
> 
> - `hx-swap-oob="true"` means `outerHTML` on the element with the same id.
> - You can use any swap style: `hx-swap-oob="beforeend"` appends the element's children to the matching id, and `hx-swap-oob="beforeend:#log"` targets a different selector. You'll use the first form with WebSockets in Chapter 7.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

A response contains a `<li>` and a `<span id="open-count" hx-swap-oob="true">`. Where does the span go?

Out-of-band elements are matched by `id`, independent of the request's target. The rest of the response still goes to the target.

Chapter 6

# hx-boost

Everything so far has used explicit `hx-*` request attributes. `hx-boost` is different: it upgrades the plain links and forms you already have.

In this chapter, here are the topics we'll cover

- What a boosted link or form does.
- Adding navigation and an About page.
- Progressive enhancement: why boost degrades gracefully.

## Boosting the navigation

Add a nav bar to the layout and put `hx-boost="true"` on it. Every `<a>` and `<form>` inside inherits it:

When you click a boosted link, htmx:

- sends the `GET` with AJAX instead of navigating;
- swaps the response's `<body>` into the current page and updates the `<title>`;
- pushes the URL into history, so Back and Forward still work.

The browser doesn't re-parse `<head>` or re-run scripts, which makes navigation feel instant. Boosted forms work the same way: the form is submitted with AJAX, and any redirect the server sends is followed.

> **Progressive enhancement:**
> 
> - The server still returns full pages at real URLs. If JavaScript fails to load, the links are ordinary links and everything still works.
> - Many apps put `hx-boost="true"` on `<body>`. Opt individual links out with `hx-boost="false"`, for example file downloads.

In the demo below, the boost is scoped to the demo box (with `hx-target` and `hx-select`) so it doesn't replace this whole page, and history is turned off.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

A user has JavaScript disabled and clicks a boosted `<a href="/about">`. What happens?

A boosted link is still a real link to a real page. Without JavaScript it simply behaves as a link. That's progressive enhancement.

Chapter 7

# Live Updates: SSE & WebSockets

Polling asks the server "anything new?" every few seconds. With Server-Sent Events and WebSockets the server tells you instead, and in htmx what it sends is still HTML.

In this chapter, here are the topics we'll cover

- Enabling extensions with `hx-ext`.
- An activity feed over SSE with `sse-connect` and `sse-swap`.
- A shared notes panel over WebSockets with `ws-connect` and `ws-send`.
- Choosing between SSE and WebSockets.

## Server-Sent Events

SSE is a one-way stream from server to browser over a normal HTTP response. Each message has an event name and some data. With the SSE extension, the data is HTML:

- **`hx-ext="sse"`** turns the extension on for this element and its children.
- **`sse-connect`** opens the stream.
- **`sse-swap="activity"`** swaps in the data of every `activity` event. Here it's newest-first with `afterbegin`.

Open the app in two browser tabs. Every add, edit, toggle or delete in one tab now appears in the other's feed. With this in place, you could remove the `every 2s` polling from Chapter 4.

## WebSockets

WebSockets are two-way. The `ws-send` attribute sends a form's fields over the socket as JSON. Whatever HTML the server sends back is swapped in by `id`, just like out-of-band swaps:

The message htmx sends contains the form fields plus a `HEADERS` object (with values like `HX-Trigger`), so the server knows which element sent it.

> **SSE or WebSockets?**
> 
> - **SSE** for server-to-browser updates: feeds, notifications, progress. It's plain HTTP, works through most proxies, and the browser reconnects automatically. User actions still go through normal `hx-post` requests.
> - **WebSockets** when the browser also sends a steady stream of messages: chat, collaborative editing, games.

The demo below shows two "tabs" connected to the same simulated server. Change a task or send a note in Tab A and watch Tab B update.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

The server writes an SSE message with `event: activity`. Which elements update?

`sse-swap` matches by event name. Messages with no event name use the name `message`.

Chapter 8

# Thinking in Hypermedia

You've used every core htmx feature. Let's step back and see what HATEOAS bought you, then look at the finished code.

In this chapter, here are the topics we'll cover

- Where the business rules live.
- When htmx is and isn't a good fit.
- The finished views and server.
- What to learn next.

## The server sends the next actions

Look again at one line from `TaskRow`:

"Completed tasks can't be edited" is a business rule, and it exists in exactly one place: the server. When you tick a task, the response is a row *without* an Edit button. Untick it and the button comes back. The browser never knew the rule; it rendered the actions it was given.

In a JSON API with a React front end, that rule typically lives in two places: the API rejects the edit, and the client hides the button. Change the rule and you deploy both, and hope they agree. With hypermedia you change one component.

The same goes for state: there's no client-side store to keep in sync. After every action, the server sends back HTML for the new state.

> **When to reach for something else:**
> 
> - htmx is great for CRUD apps, dashboards, admin tools, forms and content sites.
> - It's a poor fit for UIs that need rich local state between requests, such as drawing tools, spreadsheets, offline-first apps or anything that must react on every keystroke without a server. You can mix: use htmx for the page and a small island of client-side JavaScript where you need it.

## The finished code

## Where to go next

- **Feedback while loading:** `hx-indicator` shows a spinner during a request, and `hx-disabled-elt` disables buttons so they can't be double-clicked.
- **Server-driven behaviour:** response headers such as `HX-Trigger` (fire a client event), `HX-Redirect`, `HX-Retarget` and `HX-Reswap` let the server change what happens next.
- **Validation errors:** htmx doesn't swap 4xx/5xx responses by default. Return the form with error messages using a 200, or handle errors with the `htmx:responseError` event.
- **Picking parts of a response:** `hx-select` takes only part of the returned HTML, which is handy when an endpoint returns a full page.
- **Security:** keep templates auto-escaping (Hono JSX does), and protect state-changing routes with CSRF tokens, for example with `hx-headers`.
- **htmx 4:** when you're ready, read the "Changes in htmx 4.0" guide on four.htmx.org.

## It's time to take a quiz!

Test your knowledge and see what you've just learned.

A new rule: completed tasks can no longer be deleted. What do you need to change?

The server decides which actions to offer, so you change the template that renders them. Validating on the server as well protects against hand-crafted requests.

**Sources**

- [htmx 2 documentation](https://htmx.org/docs/) and [attribute reference](https://htmx.org/reference/)
- [htmx SSE extension](https://htmx.org/extensions/sse/) and [WebSocket extension](https://htmx.org/extensions/ws/)
- [HATEOAS essay](https://htmx.org/essays/hateoas/) (htmx.org)
- [Changes in htmx 4.0](https://four.htmx.org/htmx-4)
- [Hono streaming helper](https://hono.dev/docs/helpers/streaming) and [@hono/node-server](https://github.com/honojs/node-server)

The finished app was run and tested in Chrome with htmx 2.0.11, htmx-ext-sse 2.2.4, htmx-ext-ws 2.0.4, Hono 4.13, @hono/node-server 2.1 and TypeScript 5.9. The live demos on this page use the same htmx files with a simulated server.