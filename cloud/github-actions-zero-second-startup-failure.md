# A 0-second `startup_failure` on every workflow is a billing problem

## The thing

Every GitHub Actions run failing in 0 seconds with `startup_failure`, across all workflows and
all trigger types, is an org-level block: a spending limit hit, or Actions disabled for the
organisation. Not your YAML.

## Where I learned it

A repo where CI and deploys had both been dead for a month before anyone noticed. The deploy
pipelines were Actions workflows too, so the outage took releases with it.

## The signature

All four together mean org-level:

| | |
|---|---|
| **0 seconds** | it never started; a real failure takes time to fail |
| **every workflow** | a broken file breaks one, not twenty-two |
| **every trigger type** | `push`, `pull_request`, `schedule` and `workflow_dispatch` |
| **nothing changed** | files byte-identical to the last green run |

A tell: the run GitHub blames has `path=BuildFailed` and `state=deleted`, a placeholder rather
than one of your workflows.

## Ruling out what you control

Quick, and it makes the escalation credible.

```bash
# unchanged since the last green run?
git diff <last-good-sha> HEAD -- .github/workflows/

# repo-level Actions enabled?
gh api repos/OWNER/REPO/actions/permissions      # want enabled: true, allowed_actions: all

# workflow registered?
gh workflow list                                  # want state: active

# manual trigger fail the same way?
gh workflow run ci.yml                            # 0s means it isn't event-related
```

Then parse every workflow file and check the structure: jobs have `runs-on` or `uses`, `needs`
resolve, no step has both `uses` and `run`, reusable-workflow references point at files that
exist.

## The organisational trap

Repo admin is not org owner. This is what cost the month.

When a repo lives in an enterprise organisation someone else owns, org billing and enterprise
policy are neither visible nor changeable by you. You can verify every layer within your control
and still be stuck.

Checking billing needs the `admin:org` scope:

```bash
gh auth refresh -h github.com -s admin:org
```

Without it the endpoint 404s, which reads like "not found" rather than "not permitted".

Hand whoever owns the org a verbatim statement: what stopped, when, that it's every workflow and
every trigger, that the files are unchanged and valid, and please check the Actions spending
limit. It's going to a different team who will otherwise tell you to check your YAML.

It was also believed to have been raised and fixed once already. If it had been, there'd be at
least one successful run afterwards. Ask for the green run, not the assurance.

## Work out the blast radius early

A month with no CI sounds like a month of unvalidated code. Exactly one commit had merged in
that window without CI, and it was docs-only. "No releases" and "unreviewed code shipped" need
very different responses, and the difference is one `git log` away.

## Gotcha

Nobody noticed for a month. A total CI outage is worth its own alert, because every mechanism
that would normally tell you something is wrong is the thing that's broken.
