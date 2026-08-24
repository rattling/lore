# Azure Burstable tiers don't fail, they slow down until something times out

## The thing

Azure Database for PostgreSQL Flexible Server on a Burstable tier (the `B` series) runs on CPU
credits. Sustained load spends them, and when they run out you aren't throttled with an error,
you're throttled with latency. Everything still works, just slowly enough that whatever has a
timeout dies first.

A nightly data refresh is exactly the workload that exhausts them.

## Where I learned it

A platform running its ETL on `Standard_B1ms` (1 vCore), mid-2026. Weeks of intermittent
strangeness resolved into one cause.

## What it looked like

- The DB pegged at ~95% CPU with credits at zero
- A bulk `INSERT` in the nightly refresh crawled, about 27 minutes of active work
- The container job's own timeout fired and the run reported `Failed`
- The 02:00 job had been failing two days running before anyone noticed

The presenting symptom is a job failure, so you go and look at the job. Nothing is wrong with
the job. The floor moved under it.

## The pattern

Burstable is for spiky, mostly-idle workloads. Anything with a sustained nightly batch is the
wrong shape for it, however small the database is.

The fix that feels obvious is the wrong one. Moving up within Burstable (B1ms to B2ms) just
raises the credit ceiling, so you get the same failure later. Only General Purpose removes
credit throttling, because it has no credit mechanic at all.

The resize is in-place on Flexible Server with a brief restart, so it's cheap to do. Schedule
it rather than doing it mid-business-day.

## Why it took so long to see

Two things conspire:

1. Credit exhaustion is invisible from inside the application. Queries don't error and nothing
   in the logs says you're out of CPU credits. You see slow queries and a downstream timeout.
2. The metric that would tell you isn't the one you're watching. `cpu_percent` at 95% looks
   like ordinary load. `cpu_credits_remaining` trending to zero is the signal, and nobody
   graphs it until after the first incident.

## The half that isn't about Azure

An alert did fire. It was ignored, because it was unclear.

It didn't say what was wrong in plain language, what the impact was, or what to do about it,
and it went to one inbox with no escalation. So a real incident sailed past for two days, and
was found because it blocked a deploy rather than because anyone was told.

An unclear alert with no escalation is the same as no alert, except that it lets you believe
you have monitoring. An alert should say what happened, what it affects and what to do, and one
ignored email shouldn't be the end of the chain.

## Gotchas

- Check every environment, not the one that broke. Production was on `B2ms`: bigger, still
  Burstable, same latent failure at production load. The incident in dev was a free warning.
- Keep the SKU in IaC (Bicep `.bicepparam` here) so the tier is a reviewable line in a diff
  rather than something changed in the portal once.
- The same shape applies beyond databases. Any credit-based or burstable compute has it, and
  credit exhaustion is silent until it bites.
- Watch `cpu_credits_remaining` on a trend, not a threshold. By the time it's zero you're
  already in the incident.
