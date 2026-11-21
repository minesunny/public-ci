# Private repository CI on public Actions

The public workflow runs when a matching public branch is pushed or when manually started. It resolves a private branch or PR to one commit SHA, then checks out that SHA in each job.

## Setup

1. In the public repository, set the Actions variable `PRIVATE_REPOSITORY` to the private repository's full name, for example `minesunny/mine-query`.
2. Create `PRIVATE_REPO_READ_TOKEN` as an Actions secret in the public repository. Its fine-grained PAT needs access to the private repository with **Contents: read-only** and **Pull requests: read-only** and **Issues: read-only** permissions. Grant organization authorization if required.
3. Configure `MAVEN_REPOSITORY_USERNAME`, `MAVEN_REPOSITORY_PASSWORD`, `MAVEN_RELEASE_REPOSITORY_URL`, and `MAVEN_SNAPSHOT_REPOSITORY_URL` as Actions secrets. Optionally set the `RUNNER_TYPE` Actions variable.
4. Commit the workflow on the default branch and include it in each public branch used to trigger a push. GitHub runs the workflow version from the pushed branch. Only trusted users should have permission to push or change workflow files.

## Push trigger

If the configured private repository is `minesunny/mine-query`:

- Push public branch `mine-query/test_ci` to test the current head of the private branch `test_ci`.
- Push public branch `mine-query/feature/test` to test private branch `feature/test`.
- Push public branch `mine-query/pr/123` to test the current head of private PR 123. The `pr/` prefix is reserved for PR numbers; use manual dispatch to test a private branch with that prefix.

The prefix must exactly match the configured private repository name, including case. A push to another slash-containing public branch starts a run but fails validation. Creating or updating a private PR alone does not trigger this workflow: a public branch must be pushed again. The resolved private SHA is taken at public push processing time, not the time of a private commit.

## Manual runs

Select **Actions → Private PR CI → Run workflow**. Enter at most one of a full 40-character commit SHA, PR number, or branch. Leaving all inputs blank selects the private repository's `main` branch. PRs and branches are resolved once before the jobs start.

## Migration limitations

For PR-number runs, the policy job supplies a synthesized PR event to the private repository script and validates live PR metadata; branch and commit runs skip PR metadata validation. Public workflow results are not automatically reported as checks on the private repository; implement status reporting and update the private branch ruleset before relying on this migration. Keep the original private CI enabled until these integrations have been tested.

The migrated jobs upload test and failure artifacts, which may expose private source or test data. Public Actions logs are visible to others. PR code can read Maven credentials and checked-out source, so only run trusted revisions and use restricted credentials. Review artifacts, logs, caches and runner access before enabling this workflow.

## External pull requests

In the repository Settings, under Features → Pull requests, select Collaborators only to prevent outside users from creating pull requests. This repository setting must be configured by a repository administrator.
