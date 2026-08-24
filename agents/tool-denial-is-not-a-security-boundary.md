# Denying a tool is not denying a capability

## The thing

An agent runner enforced "network denied" by adding `WebSearch` and `WebFetch` to its
disallowed-tools list. The worker then ran `npm install` and pulled packages over the network
anyway — because `Bash` was still allowed, and `npm`, `curl`, `git` and `pip` all reach the
network without going near a web tool.

The control blocked the *named* route and left every other one open.

## Where I learned it

Toolpost, August 2026 — found while dogfooding an unattended engineering loop, not by
reasoning about it. The run transcript showed `npm install --no-audit --no-fund` succeeding on
a job whose policy said `network_allowed: false`.

## The pattern

**A tool allowlist describes an interface, not a boundary.** It shapes what the model is
*invited* to do. It says nothing about what the process can *reach*.

If `Bash` (or any general execution primitive) is available, then every capability reachable
from a shell is available, whatever the tool list says. Denying `WebFetch` while allowing
`Bash` is like locking the front door and leaving the patio open.

Real enforcement is a sandbox: network namespace, firewall, or container without egress. That
is the only thing that makes the claim true.

## Why it matters more than it looks

The failure isn't just that the control didn't work. It's that **the policy said it did.**
Somebody reading `network_allowed: false` reasonably concludes an unattended worker on an
untrusted job cannot exfiltrate, cannot pull an unpinned dependency, cannot call production.
None of that was true.

A control that's documented but not enforced is worse than no control, because it gets relied
on.

The interim fix, if the real one is far off, is to **stop advertising the guarantee** and
surface the gap. That's uncomfortable and correct.

## The mirror image

There's a counter-example from another project that got this right by construction: an agent
Python sandbox with **no network at all**, where the runtime does every piece of I/O and the
sandbox only computes. Because nothing inside can reach anything, no credential ever enters
it, and running model-written code becomes acceptable.

Same lesson from both ends: **the boundary has to be structural.** One project made it
structural and could then be relaxed about what ran inside; the other made it declarative and
the guarantee evaporated the first time somebody typed a shell command.

See [give an agent one sandbox, not twenty tools](code-channel-beats-tool-selection.md).

## Gotchas

- A `PreToolUse` hook inspecting Bash commands for network calls is a partial mitigation, not
  a boundary. It's a denylist against an infinite space, trivially bypassed by anything that
  shells out indirectly.
- This bites hardest in exactly the situation you'd least want it to: an *unattended* worker on
  an *untrusted* contract, where nobody is watching the transcript.
- Worth testing rather than reviewing. The bug was invisible in the code — the adapter did
  precisely what it said. It only showed up when someone looked at what a real run had actually
  done.
