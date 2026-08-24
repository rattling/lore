# Two agent sessions, one working tree, one bad commit

## The thing

Two coding-agent sessions sharing a working tree. One ran `git add -A` and swept the other's
in-progress files into its commit. The content survived. The history is now a lie about who did
what.

The hazard is the shared working tree, not the model.

## Where I learned it

Toolpost, August 2026. It had already happened while planning the loop that was meant to run
agents in parallel.

## The fix

Git worktrees, one per session:

```bash
git worktree add ../repo-feature-x feature/x
```

Cheap, native, removes the class. Any runner dispatching agents needs it anyway, and anyone
running two Claude Code windows on one repo needs it today.

## The part worth generalising

Three things get lumped together as "concurrency" and cost very different amounts:

1. **Filesystem and git isolation.** Worktrees. Cheap. Do it.
2. **Logical work isolation.** Two agents on the same ticket, or one rewriting a shared config
   while another reads it. This is what leases are for, and they're expensive: acquisition,
   expiry, renewal, deadlock, recovery from a crashed holder.
3. **Genuine parallelism.** Actually wanting two things at once.

Sequencing is the cheap substitute for leases. Run the loops in order and there's no contention
to manage. Leases only pay off once you want concurrent execution, and usually you don't yet.

Before building coordination machinery, check whether you can remove the concurrency instead.
Ordering is nearly free. Correct locking isn't.

## Gotchas

- The damage is to history, not content, so nothing alarms you. You find out weeks later reading
  a diff that makes no sense.
- `git add -A` is the trigger and agents reach for it constantly. Worth banning in favour of
  explicit paths even with worktrees.
- Worktrees share one `.git`, so branches, stashes and hooks are common. Two sessions still
  can't sit on the same branch.
- Applies to humans running parallel sessions too. The first victim was somebody's own two
  terminal windows.
