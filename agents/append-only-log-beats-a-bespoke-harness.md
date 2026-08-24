# For long-horizon agent work, encode the contract, not the procedure

## The thing

An append-only log plus a persistent workspace plus freedom beat purpose-built agent harnesses,
at a fraction of the tokens. The instinct to write a workflow DSL for multi-hour agent work is
usually the expensive way to do worse.

## Where I learned it

Toolpost, August 2026, designing mission memory for task-horizon runs (hours to days,
resumable). The evidence is external and public: **PRO-LONG**
([repo](https://github.com/alexisfox7/PRO-LONG) · [arXiv:2607.20064](https://arxiv.org/abs/2607.20064)),
an ARC-AGI-3 harness. I read its published run logs, not just the paper.

## The observation

PRO-LONG runs a stock coding agent (Claude Code or Codex CLI) with Read/Write/Bash/Grep, a
persistent workspace, and a system prompt of about thirty lines. No subagents, no retrieval
layer, no bespoke orchestration.

- +18pp over the same agents without the log
- matches specialised harnesses at 4.2–5.8× fewer billed tokens
- 97.4% best@2

The part that's easy to miss: the thirty-line prompt works because the scaffolding moved into
the data format.

## What transfers

**1 · The harness owns an append-only log with a designed observation language.** Not free
prose. Rigid markers (`[POST-ACTION BOARD STATE]`, `[settled]`) that make the log
programmatically searchable. Design this as a language from day one. Structure buys you
greppability, and greppability is what replaces a retrieval layer.

**2 · The agent owns a workspace and decides what goes in it.** The published notes file from a
single run (67KB, 615 entries) is a lab notebook: named hypotheses with distinguishing
predictions written before acting, a world model corrected as it learned, lessons from its own
blunders, and newest-first ordering so a partial read lands on current state.

Nobody specified any of that. Given an immutable log and some latitude, the agent invented its
own curation.

**3 · Derived state is fallible, the log is authoritative.** In one run the agent's own map was
correct, its plan ignored it, it failed, then it re-derived from the log and recorded the
lesson. Recovering from confidently-wrong memory only works because the raw log is still there.
A workspace can be wrong. The log must not be deletable while the work is live.

**4 · Memory is written forward.** Most notes are predictions and commitments ("submitted X,
expect score 7"), so the first job on resume is reconciling those against what actually
happened, not re-reading everything.

## The anti-pattern

The temptation with long-running agent work is to encode the procedure: a workflow DSL, a state
machine, a graph of steps. PRO-LONG beat hand-built specialised harnesses with a simple
contract (log discipline, a workspace, latitude) at a fraction of the cost.

Encode the contract. The procedure is what the agent is for.

## Caveats

This is evidence, not proof, and the regime matters:

- ARC is a closed world of grids, computable and exact, so `grep` genuinely is the right
  retrieval language. Real missions live in prose and messy systems.
- ARC has a free scoring function. Verification costs nothing there and is expensive
  everywhere else, which changes how much a predict-and-check loop is worth.

The contract transfers. The retrieval economics don't, which is the argument for making the
observation language as structured as your domain allows.

Worth separating this from cross-conversation product memory. This is task-horizon memory: one
job, hours to days, and the notebook dies with the job. Different problem.
