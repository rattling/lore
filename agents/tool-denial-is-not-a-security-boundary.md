# Denying a tool is not denying a capability

## The thing

An agent runner enforced "network denied" by adding `WebSearch` and `WebFetch` to its
disallowed-tools list. The worker ran `npm install` and pulled packages anyway, because `Bash`
was still allowed and `npm`, `curl`, `git` and `pip` all reach the network without a web tool.

It blocked the named route and left every other one open.

## Where I learned it

Toolpost, August 2026, dogfooding an unattended engineering loop. The run transcript showed
`npm install --no-audit --no-fund` succeeding on a job whose policy said
`network_allowed: false`.

## The pattern

A tool allowlist describes an interface, not a boundary. It shapes what the model is invited to
do, not what the process can reach.

If `Bash` or any general execution primitive is available, so is everything reachable from a
shell. Denying `WebFetch` while allowing `Bash` is locking the front door and leaving the patio
open.

Real enforcement is a sandbox: network namespace, firewall, or a container without egress.

## Why it matters more than it looks

The control didn't work, and the policy said it did.

Someone reading `network_allowed: false` reasonably concludes an unattended worker on an
untrusted job can't exfiltrate, can't pull an unpinned dependency, can't call production. None
of it was true.

A control that's documented but not enforced is worse than none, because it gets relied on. If
the real fix is far off, stop advertising the guarantee and surface the gap.

## The mirror image

Another project got this right by construction: an agent Python sandbox with no network, where
the runtime does the I/O and the sandbox only computes. Nothing inside can reach anything, so no
credential enters it, and running model-written code becomes acceptable.

The boundary has to be structural. One project made it structural and could relax about what ran
inside. The other made it declarative and lost the guarantee at the first shell command.

See [give an agent one sandbox, not twenty tools](code-channel-beats-tool-selection.md).

## Gotchas

- A `PreToolUse` hook inspecting Bash for network calls is a mitigation, not a boundary. It's a
  denylist against an infinite space, and anything that shells out indirectly gets past it.
- It bites where you'd least want it: an unattended worker on an untrusted contract, with nobody
  watching the transcript.
- Test for it, don't review for it. The adapter did exactly what it said. It only showed up when
  someone looked at what a real run had done.
