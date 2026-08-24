# Lore

Technical things I've learned, kept somewhere they'll outlive the project that taught me them.

Projects get finished, parked or abandoned. Technical lessons not covered under I.P., that deserve to be captured, can live here.

## In Scope

A folder of markdown files.

An entry exists because something was learned: a surprise, a pattern worthy of reuse, a gotcha worth remembering, a technology behaving differently than advertised. 

## What this isn't

Not an app, a database, an MCP server, a knowledge graph, a tagging system, a search layer or a
publishing platform. No metadata schema, no automation, no build step.

It's also not for things you can get from the documentation. If a competent person could look
it up in two minutes, it doesn't need to be here.

## Out of Scope

| Repo | Holds |
|---|---|
| **[ways-of-working](https://github.com/rattling/ways-of-working)** | how I run projects: setup, light architecture, the delivery loop, general engineering standards |
| **lore** (here) | technical learnings: patterns, gotchas, architectural lessons |

If it's about running a project it goes in ways-of-working. If it's about building things, it
goes here.

E.g. "design contracts at module boundaries") is a standard and belongs in ways-of-working. A specific thing that bit me
("Azure Burstable tiers throttle on CPU credits") is a lesson and belongs here.

## Capture, then curate

**Capture.** Mid-session in some other repo you notice something. Say so:

> "Capture this in Lore: giving the agent a general Python sandbox turned out more useful than
> predefining twenty narrow tools."

That becomes `inbox/2026-08-24-agent-python-sandbox.md`. Thirty seconds.

**Curate.** Later, if it's worth it, move the entry into a topic folder, merge it with
something related, or write it up properly. Do this when you're passing anyway, not on a
schedule. There's no review ceremony.

Plenty of captures will sit in `inbox/` forever. That's fine.

## Adding something

**As a human**, write a markdown file. Put it in `inbox/` if you're moving fast, or straight
into a topic folder if you know where it goes. Five lines is a perfectly good entry.

**As an agent**, see [AGENTS.md](AGENTS.md). Short version: capture when asked, keep it short,
ground it in something that actually happened, don't invent new structure.

A rough shape:

```markdown
# What the thing is

## The thing
One or two sentences. The lesson itself.

## Where I learned it
Project and roughly when.

## The pattern
What to do, described so it's reusable.

## Why it worked
The mechanism. Usually the interesting part.

## Gotchas
What bit, or nearly did.
```

Skip any section with nothing to say.

## What must not go in here

This repo is public, so be conservative. Nothing that is:

- client confidential information or client data
- credentials or secrets
- proprietary customer implementation detail
- under NDA, or commercially sensitive
- product IP that might get commercialised
- algorithms or mathematics that form part of a proposition being sold

Nothing at all from the optimisation and scheduling work. No algorithms, heuristics,
parameters, mathematical methods or sources of differentiation, including things that look
incidental.

Generic engineering lessons learned while doing sensitive work can be fine if they abstract
cleanly:

> **No:** the scheduling heuristic, its parameters, and why it beats the alternatives.
> **Yes:** designing a deterministic interface around a solver so results are reproducible and
> diffable, without saying anything about the solver.

When in doubt, leave it out. It can stay in the private repo it came from.

Most source repos are private, so entries describe patterns rather than linking to code. Small
snippets are fine where the snippet is the point. Copying whole files isn't, because then it
has to be maintained twice.

## Keeping it boring

The thing that kills a repo like this is not neglect. It is enthusiasm: a taxonomy nobody
needs, a script to build an index, a schema that turns capture into a chore. Directories appear
when there is enough material for them, not before.

A dozen entries worth re-reading beats two hundred that restate the documentation.
