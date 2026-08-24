# Denying a tool is not denying a capability

## The thing

An agent runner enforced "network denied" by adding `WebSearch` and `WebFetch` to its
disallowed-tools list. The worker then ran `npm install` and pulled packages over the network,
because `Bash` was still allowed and `npm`, `curl`, `git` and `pip` all reach the network
without going near a web tool.

The control blocked the named route and left every other one open.

## Where I learned it

Toolpost, August 2026. Found while dogfooding an unattended engineering loop, not by reasoning
about it. The run transcript showed `npm install --no-audit --no-fund` succeeding on a job whose
policy said `network_allowed: false`.

## The pattern

A tool allowlist describes an interface, not a boundary. It shapes what the model is invited to
do. It says nothing about what the process can reach.

If `Bash` or any other general execution primitive is available, every capability reachable
from a shell is available too, whatever the tool list says. Denying `WebFetch` while allowing
`Bash` is locking the front door and leaving the patio open.

Real enforcement is a sandbox: network namespace, firewall, or a container without egress.
That's the only thing that makes the claim true.

## Why it matters more than it looks

The problem isn't only that the control didn't work. It's that the policy said it did.

Someone reading `network_allowed: false` reasonably concludes that an unattended worker on an
untrusted job can't exfiltrate, can't pull an unpinned dependency, can't call production. None
of that was true.

A control that's documented but not enforced is worse than no control, because it gets relied
on.

If the real fix is far off, the interim move is to stop advertising the guarantee and surface
the gap instead. Uncomfortable, and correct.

## The mirror image

Another project got this right by construction: an agent Python sandbox with no network at all,
where the runtime does every piece of I/O and the sandbox only computes. Nothing inside can
reach anything, so no credential ever enters it, and running model-written code becomes
acceptable.

Same lesson from both ends. The boundary has to be structural. One project made it structural
and could then be relaxed about what ran inside. The other made it declarative and the
guarantee disappeared the first time somebody typed a shell command.

See [give an agent one sandbox, not twenty tools](code-channel-beats-tool-selection.md).

## Gotchas

- A `PreToolUse` hook inspecting Bash commands for network calls is a partial mitigation, not a
  boundary. It's a denylist against an infinite space and anything that shells out indirectly
  gets past it.
- This bites hardest exactly where you'd least want it: an unattended worker on an untrusted
  contract, where nobody's watching the transcript.
- Test for it rather than reviewing for it. The bug was invisible in the code, because the
  adapter did precisely what it said. It only showed up when someone looked at what a real run
  had actually done.
