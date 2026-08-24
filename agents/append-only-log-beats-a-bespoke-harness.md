# For long-horizon agent work, encode the contract, not the procedure

## The thing

An append-only log plus a persistent workspace plus freedom beat purpose-built agent harnesses,
at a fraction of the tokens. Writing a workflow DSL for multi-hour agent work is usually the
expensive way to do worse.

## Where I learned it

Toolpost, August 2026, designing mission memory for task-horizon runs. The evidence is public:
**PRO-LONG** ([repo](https://github.com/alexisfox7/PRO-LONG) ·
[arXiv:2607.20064](https://arxiv.org/abs/2607.20064)), an ARC-AGI-3 harness. I read its run logs,
not just the paper.

## The measurements

PRO-LONG runs a stock coding agent (Claude Code or Codex CLI) with Read/Write/Bash/Grep, a
persistent workspace, and about thirty lines of system prompt. No subagents, no retrieval layer,
no orchestration.

- +18pp over the same agents without the log
- matches specialised harnesses at 4.2–5.8× fewer billed tokens
- 97.4% best@2

The thirty-line prompt works because the scaffolding moved into the data format.

## What transfers

**The harness owns an append-only log with a designed observation language.** Rigid markers
(`[POST-ACTION BOARD STATE]`, `[settled]`), not free prose. Design it as a language from day
one. Structure is what lets `grep` do the job a retrieval layer would otherwise do.

**The agent owns a workspace and decides what goes in it.** The notes file from one run is 67KB,
615 entries: named hypotheses with predictions written before acting, a world model corrected as
it learned, lessons from its own blunders, newest-first so a partial read lands on current state.

Nobody specified any of that. Worth knowing before you specify how an agent should keep notes.

**Derived state is fallible, the log is authoritative.** In one run the agent's map was correct,
its plan ignored it, it failed, then it re-derived from the log and recorded the lesson. That
recovery only works because the raw log is still there. A workspace can be wrong. The log must
not be deletable while the work is live.

**Memory is written forward.** Most notes are predictions and commitments ("submitted X, expect
score 7"), so the first job on resume is reconciling those against what happened.

## The anti-pattern

The temptation is to encode the procedure: a DSL, a state machine, a graph of steps. PRO-LONG
beat hand-built harnesses with log discipline, a workspace and latitude, at a fraction of the
cost.

Encode the contract. The procedure is what the agent is for.

## Caveats

The regime matters:

- ARC is a closed world of grids, computable and exact, so `grep` is genuinely the right
  retrieval language. Real missions live in prose and messy systems.
- ARC has a free scoring function. Verification is expensive everywhere else, which changes what
  a predict-and-check loop is worth.

The contract transfers. The retrieval economics don't, so make the observation language as
structured as your domain allows.

This is task-horizon memory: one job, hours to days, notebook dies with the job. Not the same
problem as cross-conversation product memory.
