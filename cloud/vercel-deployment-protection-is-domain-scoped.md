# Vercel Deployment Protection can't exempt a path

## The thing

Deployment Protection is scoped by domain, never by path. You can't leave
`/api/machine-endpoint` open while the rest of the deployment stays behind auth.

## Where I learned it

Seam, August 2026, trying to drop a shared bypass secret from the setup instructions for an MCP
endpoint.

## The detail

Three things look like they might do it and none does:

- **OPTIONS Allowlist.** CORS preflight only, so useless for a POST.
- **Deployment Protection Exceptions.** Domain-based, and behind the Advanced Deployment
  Protection add-on ($150/month at the time).
- **Standard Protection.** Protects everything except production custom domains. A generated
  `*.vercel.app` production URL is covered.

Verified rather than assumed. A POST with a valid bearer token and no bypass header returns:

```json
{"error":{"message":"Protected deployment","code":"401"},
 "protection":{"vercel_auth_enabled":true,"password_enabled":false}}
```

The edge rejects it before the app runs, so your own auth never sees it.

## What that leaves you

1. **Send `x-vercel-protection-bypass`** (Protection Bypass for Automation). Works, but it's a
   second secret every client holds, and not every client can set custom headers.
2. **Put production on a custom domain.** Unprotected under Standard Protection, but that
   exempts the whole app rather than the endpoint, so your own auth becomes the only gate.

There isn't a third option, and the second is a bigger decision than it looks.

## Gotcha

It looks like a routing bug. You get an HTML login page or a 404 where you expected JSON, so the
instinct is to check your route. Check deployment protection first. The request never reached
your code.
