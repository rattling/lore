# Per-request state doesn't belong in a memoised handler

## The thing

Cache a handler per credential, register its behaviour once, and anything request-scoped in that
closure becomes first-request-wins for every later caller sharing the credential. It fails
invisibly: right answers, wrong tenant.

## Where I learned it

Seam, August 2026. An MCP server where each repo binds itself to a project by URL
(`/api/mcp/<project>`), so tool calls default to that project.

## The setup

The route memoised one handler per API token:

```ts
const handlerFor = memoizeByKey(
  (authed) => createMcpHandler(async (server) => {
    registerTools(server, { db, viewer: await resolveViewer(db, authed.user) });
  }),
  (authed) => authed.tokenId,
);
```

That memoisation was itself a fix. The MCP library expects to be constructed once, and building
one per request piled up transports until the process died. So it had to stay.

The obvious way to add the project binding was to pass it into `registerTools` alongside
`viewer`. Tools register once per handler and close over what they're given. One token, several
repos, and the first repo to call fixes the project for every later call on that token. A card
filed against the wrong project, nothing in any log.

## The fix

`AsyncLocalStorage`. One handler per token, one binding per request.

```ts
const store = new AsyncLocalStorage<string | undefined>();
export const withBoundProject = (p, fn) => store.run(p, fn);
export const boundProject = () => store.getStore();

// route:
return withBoundProject(projectFromUrl, () => handlerFor(authed)(request));
```

Tools read `boundProject()` when the caller omits an explicit argument.

## Why not widen the cache key

`token:project` looks easier and is worse. The cache is an unbounded `Map` and the project name
comes from the client, so arbitrary values leak handlers indefinitely, which is the bug the
memoisation existed to fix.

A cache key derived from client input is a memory-exhaustion vector. Widening a key to fix a
correctness problem usually means the state is in the wrong place.

## Gotchas

- Write the leak test and name it for the failure. Mine is "does not leak one repo's binding
  into another's request on the same handler": two bindings through one registered toolset,
  asserting no bleed. Without it this regresses the first time someone simplifies the context
  plumbing, and no ordinary test notices.
- Node runtime only. `AsyncLocalStorage` is `node:async_hooks`.
- Routing may need a rewrite step. The MCP handler matches a fixed endpoint, so
  `/api/mcp/<project>` is rewritten back to `/api/mcp` with the project travelling out of band.
  Rebuilding a `Request` around a stream body needs `duplex: "half"`, which isn't in the DOM
  types.
- Broader than MCP. Any long-lived per-credential object (a cached client, a prepared statement
  set, a registered router) has this shape. Ask what varies per request versus per credential.
