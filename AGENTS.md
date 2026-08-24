# Working in this repo

This is a folder of markdown files holding technical lessons. It is deliberately not a
product. Your main job here is to add to it without making it heavier.

Read [README.md](README.md) first — especially the section on what must not be published.

## When to write an entry

Write one when the human asks ("capture this in Lore"), or when you've just watched something
genuinely instructive happen and you propose it.

An entry earns its place when:

- something was surprising
- a pattern proved unusually useful
- a mistake or gotcha is worth remembering
- an architectural decision generalises past its project
- a technology behaved differently than its documentation implied
- a technique is likely to come up again

It does **not** earn its place when it's recoverable from official docs in two minutes, or
when it's a general principle nobody learned from anything in particular. "Use dependency
injection for testability" is not lore. It's filler, and filler is how this repo dies.

## How to write

**Concise.** Five lines is a fine entry. Length should track how much was actually learned,
not how much can be said.

**Grounded.** Every entry comes from something real — a project, an experiment, a measurement,
a specific piece of research. Say where it came from and roughly when. If you can't name where
it was learned, that's a strong signal it's filler.

**Separate observation from conclusion.** *"Tool selection topped out at 60–70% across nine
similar tools"* is an observation. *"Therefore remove the selection step"* is a conclusion.
Keep the line visible — a later reader may accept the measurement and reject the reasoning, and
should be able to.

**Prefer the mechanism.** *Why* something worked is usually more transferable than *what* was
done. If you can only include one, include the why.

**Don't copy implementations.** Describe the pattern and link to it if the source is public.
Most source repos here are private, so describe it fully enough to be useful standalone. A
small snippet is fine when the snippet is the point. Never paste a file.

## Where to put it

- `inbox/` — anything quick. Name it `YYYY-MM-DD-short-slug.md`. No need to think about
  where it belongs.
- a topic folder — when you already know, and one exists.

**Do not create a new directory for a single entry.** Put it in `inbox/` and let the folder
appear when three or four related things have accumulated. Do not restructure the taxonomy
because it feels untidy; it's supposed to be shallow.

## Don't build anything

No scripts, no indexes, no front matter schema, no CI, no generators, no MCP server, no
tooling of any kind — unless explicitly asked. The repo's value is that writing to it is
free. Every piece of machinery taxes that.

If you find yourself proposing automation, the honest move is to say so and let the human
decline it.

## Before you commit

- Does this contain client data, credentials, NDA material, or commercially sensitive detail?
- Does it touch the optimisation/scheduling proposition? If so, leave it out — including
  things that look incidental.
- Would a competent stranger find this useful, or is it documentation for its own sake?

If any of those gives you pause, don't publish it. It can live in the private repo it came
from.
