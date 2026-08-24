# Two agent sessions, one working tree, one bad commit

## The thing

Two coding-agent sessions sharing a working tree. One ran `git add -A` and swept the other
session's in-progress files into its commit. The content survived; the history is a lie about
who did what and why.

**A shared working tree is the hazard.** Not the model, not the prompt — the directory.

## Where I learned it

Toolpost, August 2026. Filed as evidence rather than speculation, because it had already
happened while planning the very loop that was meant to run agents in parallel.

## The fix

Git worktrees. One per session:

```bash
git worktree add ../repo-feature-x feature/x
```

Cheap, native, and it removes the whole class. Any runner dispatching agents needs this
anyway; anyone running two Claude Code windows on one repo needs it today.

## The part worth generalising

Three concerns get conflated when people say "concurrency", and they have very different costs:

**1 · Filesystem and git isolation.** Solved by worktrees. Cheap. Do it.

**2 · Logical work isolation.** Two agents touching the same ticket, or the same shared config
— one rewriting a document while another reads it. This is what leases and locks are for, and
they're expensive: acquisition, expiry, renewal, deadlock, recovery from a crashed holder.

**3 · Genuine parallelism** — actually wanting two things to run at once.

The move that saved the most work: **sequencing is the cheap substitute for leases.** Run the
loops in order rather than concurrently and the contention doesn't need managing, because it
doesn't exist. Leases only earn their keep once concurrent execution is genuinely wanted — and
it usually isn't yet.

That's a general shape. When you're about to build coordination machinery, check whether you
can remove the concurrency instead. Ordering is nearly free; correct locking is not.

## Gotchas

- **The damage is to history, not content.** Nothing is lost, so nothing alarms you. You find
  out weeks later reading a diff that makes no sense, which is a bad time to find out.
- `git add -A` is the specific trigger, and agents reach for it constantly. Worth banning in
  favour of explicit paths even with worktrees.
- Worktrees share one `.git` — so branches, stashes and hooks are common. Two sessions still
  can't sit on the same branch.
- This applies to *humans* running parallel sessions too, not just to an orchestrator. The
  first victim here was somebody's own two terminal windows.
