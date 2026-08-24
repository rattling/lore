# Azure Burstable tiers don't fail, they slow down until something times out

## The thing

Azure Database for PostgreSQL Flexible Server on a Burstable tier runs on CPU credits. Sustained
load spends them, and when they run out you're throttled with latency rather than an error.
Everything still works, just slowly enough that whatever has a timeout dies first.

A nightly refresh is exactly the workload that exhausts them.

## Where I learned it

A platform running its ETL on `Standard_B1ms`, mid-2026. Weeks of intermittent strangeness, one
cause.

## What it looked like

- DB pegged at ~95% CPU, credits at zero
- a bulk `INSERT` in the nightly refresh crawling, ~27 minutes of active work
- the container job's own timeout firing, run reported `Failed`
- the 02:00 job failing two days running before anyone noticed

The symptom is a job failure, so you go and look at the job. Nothing's wrong with the job.

## The pattern

Burstable is for spiky, mostly-idle workloads. A sustained nightly batch is the wrong shape for
it however small the database is.

The obvious fix is wrong. Moving up within Burstable (B1ms to B2ms) raises the credit ceiling
and you get the same failure later. Only General Purpose removes credit throttling, because it
has no credit mechanic.

The resize is in-place with a brief restart. Schedule it rather than doing it mid-business-day.

## Why it took so long to see

Credit exhaustion is invisible from inside the application. Queries don't error, nothing in the
logs mentions credits. You see slow queries and a downstream timeout.

And the metric that would tell you isn't the one you're watching. `cpu_percent` at 95% looks
like load. `cpu_credits_remaining` trending to zero is the signal, and nobody graphs it until
after the first incident.

## The half that isn't about Azure

An alert fired. It was ignored, because it was unclear: no plain statement of what was wrong,
what it affected, or what to do, sent to one inbox with no escalation. A real incident sailed
past for two days and was found because it blocked a deploy.

An unclear alert with no escalation is the same as no alert, except it lets you believe you have
monitoring.

## Gotchas

- Check every environment, not the one that broke. Production was on `B2ms`: bigger, still
  Burstable, same latent failure at production load.
- Keep the SKU in IaC so the tier is a reviewable line in a diff rather than something changed
  in the portal once.
- Applies beyond databases. Any credit-based compute has it, and exhaustion is silent until it
  bites.
- Watch `cpu_credits_remaining` on a trend. By the time it's zero you're in the incident.
