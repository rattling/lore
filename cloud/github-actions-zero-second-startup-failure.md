# A 0-second `startup_failure` on every workflow is a billing problem

## The thing

If every GitHub Actions run fails in 0 seconds with `startup_failure`, across all workflows and
all trigger types, that's an org-level block: a spending limit hit, or Actions disabled for the
organisation. It isn't your YAML.

Recognising the signature saves you debugging a workflow file that's fine.

## Where I learned it

A repo where CI and deploys had both been dead for a month before anyone noticed. The deploy
pipelines were Actions workflows too, so the outage took releases with it.

## The signature

All four together mean org-level, not code-level:

| | |
|---|---|
| **0 seconds** | it never started; a real failure takes time to fail |
| **every workflow** | a broken file breaks one workflow, not twenty-two |
| **every trigger type** | `push`, `pull_request`, `schedule` and `workflow_dispatch` |
| **nothing changed** | the files are byte-identical to the last green run |

A useful tell: the run GitHub blames has `path=BuildFailed` and `state=deleted`, which is a
placeholder rather than one of your workflows.

## Ruling out what you control

Worth doing in this order. It's quick, and it makes the escalation credible.

```bash
# unchanged since the last green run?
git diff <last-good-sha> HEAD -- .github/workflows/

# repo-level Actions actually enabled?
gh api repos/OWNER/REPO/actions/permissions      # want enabled: true, allowed_actions: all

# is the workflow even registered?
gh workflow list                                  # want state: active

# does a manual trigger fail the same way?
gh workflow run ci.yml                            # 0s startup_failure means it isn't event-related
```

Then parse every workflow file and check the structure: jobs have `runs-on` or `uses`, `needs`
resolve, no step has both `uses` and `run`, reusable-workflow references point at files that
exist. Twenty-two files takes a minute to check programmatically and removes the last doubt.

## The organisational trap

This is the part that cost the month. Repo admin is not org owner.

When a repo lives in an enterprise organisation someone else owns, org billing and enterprise
policy are neither visible nor changeable by you. You can verify every layer within your
control is healthy and still be stuck, because the broken layer is one you can't see.

Checking billing from the CLI needs the `admin:org` scope:

```bash
gh auth refresh -h github.com -s admin:org
```

Without it the billing endpoint 404s, which reads like "not found" rather than "not permitted",
so it's easy to take as another dead end.

The practical move is to hand whoever does own the org a verbatim statement: what stopped, when,
that it's every workflow and every trigger, that the files are unchanged and valid, and to
please check the org's Actions spending limit. Be precise, because it's going to a different
team who will otherwise tell you to check your YAML.

It was also believed to have been raised and fixed once already. If it had been, there would be
at least one successful run afterwards, and there were none. Ask for the green run rather than
the assurance.

## Work out the blast radius early

A month with no CI sounds like a month of unvalidated code. It wasn't. Exactly one commit had
merged in that window without CI and it was docs-only.

"A month of no releases" and "a month of unreviewed code shipped" need very different responses,
and the difference is one `git log` away.

## Gotcha

Nobody noticed for a month. Scheduled jobs silently stopped, deploys silently stopped, and it
only surfaced when it blocked something urgent.

A total CI outage is worth its own alert, because every mechanism that would normally tell you
something is wrong is the thing that's broken.
