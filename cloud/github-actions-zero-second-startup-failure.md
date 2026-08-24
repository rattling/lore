# A 0-second `startup_failure` on every workflow is a billing problem

## The thing

If **every** GitHub Actions run fails in **0 seconds** with `startup_failure`, across **all**
workflows and **all** trigger types — that's an org-level block. A spending limit hit, or
Actions disabled for the organisation. It is not your YAML.

Recognising the signature saves you from debugging a workflow file that's fine.

## Where I learned it

A repo where CI and deploys had both been dead for a month before anyone noticed — the deploy
pipelines were Actions workflows too, so the outage took releases with it.

## The signature

All four together mean org-level, not code-level:

| | |
|---|---|
| **0 seconds** | it never started; a real failure takes time to fail |
| **every workflow** | a broken file breaks one workflow, not twenty-two |
| **every trigger type** | `push`, `pull_request`, `schedule` *and* `workflow_dispatch` |
| **nothing changed** | the files are byte-identical to the last green run |

A useful tell: the run GitHub attributes the failure to has `path=BuildFailed` and
`state=deleted` — a placeholder, not one of your workflows.

## Ruling out what you control

Worth doing in this order, because it's quick and it makes the escalation credible:

```bash
# unchanged since the last green run?
git diff <last-good-sha> HEAD -- .github/workflows/

# repo-level Actions actually enabled?
gh api repos/OWNER/REPO/actions/permissions      # want enabled: true, allowed_actions: all

# is the workflow even registered?
gh workflow list                                  # want state: active

# does a manual trigger fail the same way?
gh workflow run ci.yml                            # 0s startup_failure → not event-related
```

Then parse every workflow file and check the structure — jobs have `runs-on` or `uses`, `needs`
resolve, no step has both `uses` and `run`, reusable-workflow references point at files that
exist. Twenty-two files takes a minute to check programmatically and removes the last doubt.

## The organisational trap

Here's the part that actually cost the month: **repo admin is not org owner.**

When a repo lives in an enterprise organisation someone else owns, org billing and enterprise
policy are neither visible nor changeable by you. You can verify every layer within your
control is healthy — and still be completely stuck, because the broken layer is one you can't
see.

Checking billing from the CLI needs the `admin:org` scope:

```bash
gh auth refresh -h github.com -s admin:org
```

Without it the billing endpoint just 404s, which reads like "not found" rather than "not
permitted" — so it's easy to misread as another dead end.

The practical move is to hand whoever *does* own the org a verbatim, unambiguous statement:
what stopped, when, that it's every workflow and every trigger, that the files are unchanged
and valid, and to please check the org's Actions spending limit. Precision matters because it's
going to a different team who will otherwise ask you to check your YAML.

Worth also noting: it was believed to have been raised and fixed once already. The evidence
said otherwise — if it had been fixed there'd be at least one successful run afterwards, and
there were none. **"It was sorted" is a claim; a green run is evidence.**

## Assess the blast radius before panicking

A month with no CI *sounds* like a month of unvalidated code. It wasn't: exactly one commit had
merged in that window without CI, and it was docs-only.

Work that out early. "A month of no releases" and "a month of unreviewed code shipped" call for
very different responses, and the difference is one `git log` away.

## Gotcha

Nobody noticed for a month. Scheduled jobs silently stopped, deploys silently stopped, and it
surfaced only when it blocked something urgent.

**A total CI outage is worth an alert of its own.** Every mechanism that would normally tell
you something is wrong is, in this failure, the thing that's broken.
