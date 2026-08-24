# Vercel Deployment Protection can't exempt a path

## The thing

Deployment Protection is scoped **by domain, never by path**. There is no way to leave
`/api/machine-endpoint` open while the rest of the deployment stays behind auth.

## Where I learned it

Seam, August 2026 — trying to drop a shared bypass secret from the setup instructions for an
MCP endpoint.

## The detail

Three things look like they might do it, and none does:

- **OPTIONS Allowlist** — CORS preflight only. Useless for a POST.
- **Deployment Protection Exceptions** — domain-based, *and* behind the Advanced Deployment
  Protection add-on ($150/month at the time).
- **Standard Protection** — protects everything *except* production **custom** domains. A
  generated `*.vercel.app` production URL is covered.

Verified rather than assumed: a POST with a valid bearer token and no bypass header returns

```json
{"error":{"message":"Protected deployment","code":"401"},
 "protection":{"vercel_auth_enabled":true,"password_enabled":false}}
```

The edge rejects it before the app runs, so your own auth never gets a look in.

## What this means in practice

If you need a machine-reachable endpoint on a protected deployment, the options are:

1. **Send `x-vercel-protection-bypass`** (Protection Bypass for Automation). Works, but it's a
   second secret every client must hold — and headers are the one thing not every client can
   set, so it constrains which tools can reach you.
2. **Put production on a custom domain.** Unprotected under Standard Protection — but that
   exempts the *whole app*, not the endpoint. Your own auth becomes the only gate.

There isn't a third option, and the second isn't a smaller decision than it looks.

## Gotcha

The failure looks like a routing bug, not an auth one. You get an HTML login page or a 404
where you expected JSON, so the instinct is to go and check your route. Check the deployment
protection setting first — the request never reached your code.
