# Lore

Technical things I've learned, kept somewhere they'll outlive the project that taught me them.

Projects get finished, parked or abandoned. The useful part is rarely the code — it's the two
or three things you'd do differently next time. Those tend to die with the repo. This is where
they don't.

---

## What this is

A folder of markdown files. That's the whole design.

An entry exists because something was **actually learned** — a surprise, a pattern that proved
its worth, a gotcha worth remembering, a technology behaving differently than advertised. Not
because a topic exists and ought to be documented.

## What this isn't

Not an app, a database, an MCP server, a knowledge graph, a tagging system, a search layer, or
a publishing platform. No metadata schema, no automation, no build step.

If it ever grows one of those, something has gone wrong. The value here is that adding to it
costs nothing, and that property is easy to destroy.

It's also not a place for things easily recovered from the documentation. If a competent person
could get it from the official docs in two minutes, it doesn't need to be here.

## Where it sits

| Repo | Holds |
|---|---|
| **jmem** | what I think and remember — notes, ideas, decisions, diary |
| **[ways-of-working](https://github.com/rattling/ways-of-working)** | *how* I work — process, conventions, templates, agent operating instructions |
| **lore** (here) | what I've learned *technically* — patterns, gotchas, architectural insight |

The line that usually settles it: **if it's about running a project, it's ways-of-working; if
it's about building the thing, it's lore.**

---

## Capture, then curate

Two stages, and the first one has to be cheap or it won't happen.

**Capture.** Mid-session, in some other repo, you notice something. Say so:

> "Capture this in Lore: giving the agent a general Python sandbox turned out more useful than
> predefining twenty narrow tools."

That becomes `inbox/2026-08-24-agent-python-sandbox.md`. Thirty seconds. Done.

**Curate.** Later, if it earns it, an entry gets moved into a topic folder, merged with
something related, or fleshed out. Opportunistically — when you're passing anyway, not on a
schedule. There is no review ceremony and there shouldn't be one.

Plenty of captures will stay in `inbox/` forever. That's a working outcome, not a backlog.

## Adding something

**As a human** — write a markdown file. Put it in `inbox/` if you're moving fast, or straight
into a topic folder if you know where it goes. Loose structure below; five lines is a perfectly
good entry.

**As an agent** — see [AGENTS.md](AGENTS.md). Short version: capture when asked, keep it
concise, ground it in something that actually happened, don't invent taxonomy.

A rough shape, not a schema:

```markdown
# What the thing is

## The thing
One or two sentences. The lesson itself.

## Where I learned it
Project and roughly when. Provenance matters more than polish.

## The pattern
What to do, described so it's reusable.

## Why it worked
The mechanism. This is usually the interesting part.

## Gotchas
What bit, or nearly did.
```

Skip any section that has nothing to say. Some entries are a paragraph.

---

## What must not go in here

This repo is public. Be conservative:

- client confidential information or client data
- credentials or secrets
- proprietary customer implementation detail
- anything under NDA, or commercially sensitive
- product IP that might be commercialised
- algorithms or mathematics that form part of a proposition being sold

**In particular, nothing from the optimisation and scheduling work.** Algorithms, heuristics,
parameters, mathematical methods, sources of differentiation — none of it, including things
that look incidental.

Generic engineering lessons learned *while* doing sensitive work can be fine, if they abstract
cleanly:

> **No** — the scheduling heuristic, its parameters, and why it beats the alternatives.
> **Yes** — designing a deterministic interface around a solver, so results are reproducible
> and diffable, without saying anything about the solver.

**When in doubt, leave it out.** It can stay in the originating private repo. Nothing here is
so valuable that it's worth a judgement call about disclosure.

Most source repos are private, so entries describe patterns rather than linking to code. Small
snippets are fine where the snippet *is* the point; wholesale copying isn't, because then it
has to be maintained in two places.

---

## Keeping it boring

The failure mode isn't neglect, it's enthusiasm — a taxonomy nobody needs, a script to
generate an index, a schema that makes capture a chore. Directories appear when there's enough
material to justify them, not in anticipation.

Quality over volume, and by some distance. A dozen entries worth re-reading beats two hundred
that restate the docs.
