# htmx-tasks

A small task list built with [htmx](https://htmx.org) and a TypeScript server. It has inline editing, live search, a running count of open tasks, and real-time updates over Server-Sent Events and WebSockets. There is no client-side JavaScript framework: every interaction is an HTTP request that returns HTML.

This is a learning project for the core htmx features and the idea behind them, HATEOAS (Hypermedia As The Engine Of Application State). The server owns all the state and sends the next set of available actions as HTML.

**The full step-by-step tutorial for building this app is in [htmx Foundations.md](htmx%20Foundations.md)** at the root of this repository. It has 8 chapters, from the first server-rendered page through to SSE and WebSockets.

## Features

| Feature | How it works |
| --- | --- |
| Inline edit | `hx-get` loads an edit form that replaces the row (`hx-swap="outerHTML"`); `hx-put` saves it and returns the updated row |
| Add a task | `hx-post` appends the new row to the list (`hx-swap="beforeend"`) |
| Delete a task | `hx-delete` returns an empty body, so the row is replaced with nothing |
| Mark as done | A checkbox sends `hx-put` on `change` |
| Live search | `hx-trigger="keyup changed delay:300ms, search"` replaces the list contents (`hx-swap="innerHTML"`) |
| Open-task count | Out-of-band swaps (`hx-swap-oob`) update the heading from the same response |
| Server stats | Polled with `hx-trigger="every 2s"` |
| Navigation | `hx-boost` turns normal links into AJAX page loads |
| Activity feed | The SSE extension pushes HTML to every open browser |
| Team notes | The WebSocket extension broadcasts notes to every connected browser |

Completed tasks don't show an Edit button. That rule lives only on the server: the browser renders whatever actions the HTML contains.

## Tech stack

- [htmx](https://htmx.org) 2.0.11, with the [htmx-ext-sse](https://htmx.org/extensions/sse/) and [htmx-ext-ws](https://htmx.org/extensions/ws/) extensions (loaded from jsDelivr)
- [Hono](https://hono.dev) with JSX for server-side HTML templates
- [@hono/node-server](https://github.com/honojs/node-server) and [ws](https://github.com/websockets/ws) for Node.js and WebSocket support
- TypeScript, run with [tsx](https://tsx.is)

## Getting started

Requires Node.js 20 or later.

```bash
npm install
npm run dev
```

Then open http://localhost:3000. `npm run dev` watches the source files and restarts the server when they change.

To see the live updates, open the app in two browser tabs. Changes and notes made in one tab appear in the other.

## Project structure

```
htmx-tasks/
├── src/
│   ├── server.tsx   # Hono app: pages, HTML fragment routes, SSE and WebSocket endpoints
│   ├── views.tsx    # JSX components: Layout, TaskRow, TaskEditRow, OpenCount, …
│   └── store.ts     # In-memory task store and change event bus
├── static/
│   └── styles.css   # Served at /static/styles.css
├── htmx Foundations.md  # The full tutorial
├── package.json
└── tsconfig.json
```

## Routes

| Method | Path | Returns |
| --- | --- | --- |
| `GET` | `/` | The full page |
| `GET` | `/about` | About page (used to demonstrate `hx-boost`) |
| `GET` | `/tasks?q=` | `<li>` rows matching the search |
| `POST` | `/tasks` | The new row, plus the open count out of band |
| `GET` | `/tasks/:id` | The row in display mode |
| `GET` | `/tasks/:id/edit` | The row in edit mode |
| `PUT` | `/tasks/:id` | The updated row |
| `PUT` | `/tasks/:id/toggle` | The updated row, plus the open count out of band |
| `DELETE` | `/tasks/:id` | Only the open count out of band (the row is removed) |
| `GET` | `/stats` | Task and open-task counts as text |
| `GET` | `/events` | Server-Sent Events stream of `activity` events |
| `GET` | `/ws` | WebSocket for team notes |

## Notes

- **Data isn't saved.** Tasks are kept in memory and reset to the three sample tasks when the server restarts.
- **htmx version.** This project uses htmx 2. htmx 4 (on npm's `next` tag) moves to `fetch()`, builds SSE into the core and makes attribute inheritance opt-in, so some attributes here would need changes to upgrade.
- **TypeScript config.** `tsconfig.json` sets `"types": ["node"]` so imports like `node:events` resolve, and `"jsxImportSource": "hono/jsx"` so JSX renders with Hono instead of React.
- **No auth or CSRF protection.** Don't deploy this as-is.
