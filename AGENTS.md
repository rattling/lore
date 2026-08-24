# Working in this repo

This is a folder of markdown files holding technical lessons. It's deliberately not a product.
Your job here is to add to it without making it heavier.

Read [README.md](README.md) first, especially the section on what must not be published.

## When to write an entry

When the human asks ("capture this in Lore"), or when you've just watched something
instructive happen and you propose it.

An entry earns its place when:

- something was surprising
- a pattern proved unusually useful
- a mistake or gotcha is worth remembering
- an architectural decision generalises past its project
- a technology behaved differently than its documentation implied
- a technique is likely to come up again

It does not earn its place when it is in the official documentation, or when it is a general
principle nobody learned from anything in particular. "Use dependency injection for
testability" is filler. Enough filler and nobody reads the repo, which is the failure mode to
avoid.

## How to write

**Keep it short.** Five lines is a fine entry. Length should track how much was learned, not
how much can be said.

**Ground it.** Every entry comes from something real: a project, an experiment, a measurement,
a specific piece of research. Say where it came from and roughly when. If you cannot name where
it was learned, that is usually a sign there is nothing to record.

**Separate observation from conclusion.** "Tool selection topped out at 60–70% across nine
similar tools" is an observation. "So remove the selection step" is a conclusion. Keep the line
visible, because a later reader might accept the measurement and reject the reasoning.

**Prefer the mechanism.** Why something worked usually transfers better than what was done. If
you can only include one, include the why.

**Don't copy implementations.** Describe the pattern, and link to it if the source is public.
Most source repos here are private, so describe it fully enough to stand alone. A small snippet
is fine when the snippet is the point. Never paste a file.

## Where to put it

- `inbox/` for anything quick. Name it `YYYY-MM-DD-short-slug.md`. Don't think about where it
  belongs.
- a topic folder when you already know and one exists.

Don't create a new directory for a single entry. Put it in `inbox/` and let the folder appear
once three or four related things have piled up. Don't restructure the taxonomy because it
looks untidy. It's meant to be shallow.

## Don't build anything

No scripts, no indexes, no front matter schema, no CI, no generators, no MCP server, no tooling
of any kind, unless you're asked. The value here is that writing to it is free, and every piece
of machinery taxes that.

If you find yourself proposing automation, say so and let the human decline it.

## Before you commit

- Does this contain client data, credentials, NDA material or commercially sensitive detail?
- Does it touch the optimisation or scheduling work? If so, leave it out, including things that
  look incidental.
- Would a stranger find this useful, or is it documentation for its own sake?

If any of those gives you pause, don't publish it. It can live in the private repo it came
from.
