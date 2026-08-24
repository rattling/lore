# Lore

Technical things I've learned, kept somewhere they'll outlive the project that taught me them.

Projects get finished, parked or abandoned. Technical lessons not covered under I.P., that deserve to be captured, can live here.

**This is a content repo, not a tool.** Fork it, delete my entries, keep the shape. The value
isn't what's in here, it's having somewhere cheap enough that you actually write things down.

The habit it's built around: while working in some other repo you notice something, say "capture
that in lore", and carry on. Later, if it earns it, you tidy. Mostly you don't.

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

E.g. "design contracts at module boundaries" is a standard and belongs in ways-of-working. A specific thing that bit me
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

Write a markdown file. `inbox/` if you're moving fast, a topic folder if you know where it
goes. Five lines is a perfectly good entry.

**To capture from your other repos**, the instruction has to be somewhere the agent reads while
it's working over there, which is not this repo. Two places it can go:

- your machine-level agent file (`~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`), which applies in
  every repo on that box
- each repo's own `AGENTS.md`, which is more explicit and travels with the repo

Either way, point it at your lore checkout and tell it to follow [AGENTS.md](AGENTS.md) from
here. That file carries the quality bar and the confidentiality checks. Pasting its contents
into your own repo's `AGENTS.md` works just as well.

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

Generic lessons learned while doing sensitive work are fine if they abstract cleanly. When in
doubt, leave it out. It can stay in the private repo it came from.

Most source repos are private, so entries describe patterns rather than linking to code. Small
snippets are fine where the snippet is the point. Copying whole files isn't, because then it
has to be maintained twice.

### Keeping it out

Your repo, your reputation, so caveat emptor. But a few things help:

- **Draft, then review.** Have the agent write to `inbox/` and stop, and you commit. Ten
  seconds, and it's the only control that catches what a word list can't.
- **A pre-commit hook** grepping the staged diff for client names, project code names and
  private repo names. Catches the copy-paste case, which is the common one.
- **Keep that list out of this repo.** If your hook's list of client names is committed here,
  you've published your client list. Put it in `~/.config/` and read it from there.
- **Watch anything you copied in.** The likeliest leak isn't something you wrote. It's a
  template or worked example carried over from elsewhere that you didn't read all of.
- **Scan before you push, not after.** A public repo is public immediately, and force-pushing
  afterwards is damage limitation rather than a fix.

## Keeping it simple

Enthusiasm kills these kinds of repos. KISS.
