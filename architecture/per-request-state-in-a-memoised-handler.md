# Per-request state doesn't belong in a memoised handler

## The thing

If you cache a handler per credential and register its behaviour once, anything request-scoped
you put into that closure silently becomes *first-request-wins* for every later caller sharing
the credential. It fails invisibly — right answers, wrong tenant.

## Where I learned it

Seam, August 2026. An MCP server where each repo binds itself to a project by URL
(`/api/mcp/<project>`), so tool calls default to that project.

## The setup

The HTTP route memoised one handler per API token:

```ts
const handlerFor = memoizeByKey(
  (authed) => createMcpHandler(async (server) => {
    registerTools(server, { db, viewer: await resolveViewer(db, authed.user) });
  }),
  (authed) => authed.tokenId,
);
```

That memoisation was itself a fix for a real leak — the MCP library expects to be constructed
once, and building one per request piled up transports until the process died. So it had to
stay.

The obvious way to add the project binding was to pass it into `registerTools` alongside
`viewer`. That's wrong, and quietly so: tools register *once* per handler and close over
whatever they were given. One token, several repos — my laptop has a dozen — and the first
repo to call would fix the project for every later call on that token. A card filed against
the wrong project, with nothing in any log to show it.

## The fix

`AsyncLocalStorage`. One handler per token, as the leak fix requires; one binding per request,
as correctness requires.

```ts
const store = new AsyncLocalStorage<string | undefined>();
export const withBoundProject = (p, fn) => store.run(p, fn);
export const boundProject = () => store.getStore();

// route:
return withBoundProject(projectFromUrl, () => handlerFor(authed)(request));
```

Tools read `boundProject()` when the caller omits an explicit argument.

## Why not just widen the cache key

`token:project` looks like the easy fix and is worse. The cache was an unbounded `Map`, and the
project name comes from the client — so an arbitrary number of distinct values would leak
handlers indefinitely, resurrecting the exact bug the memoisation existed to fix.

**The general shape: when a cache key is derived from client input, it's a memory-exhaustion
vector.** Widening a key to solve a correctness problem is usually a sign the state is in the
wrong place, not that the key is too narrow.

## Gotchas

- **Write the leak test, and name it for the failure.** Mine is *"does not leak one repo's
  binding into another's request on the same handler"* — two bindings through one registered
  toolset, asserting no bleed. Without it this regresses the first time someone "simplifies"
  the context plumbing, and no ordinary test would notice.
- **Node runtime only.** `AsyncLocalStorage` is `node:async_hooks`; it won't work on an edge
  runtime.
- **Routing may need a rewrite step.** The MCP handler matches a fixed endpoint path, so
  `/api/mcp/<project>` had to be rewritten back to `/api/mcp` before reaching it, with the
  project travelling out-of-band. Rebuilding a `Request` around a stream body needs
  `duplex: "half"`, which isn't in the DOM types.
- **The general case is broader than MCP.** Any long-lived, per-credential object — a cached
  client, a prepared statement set, a registered router — has this shape. Ask what varies per
  *request* versus per *credential*, and don't let the first one hide inside the second.
