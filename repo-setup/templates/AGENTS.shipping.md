# AGENTS.md

Instructions for any coding agent (human-assisted or autonomous) working in this repository.

Keep this file **agent-general**. Tool-specific setup (Cursor Cloud `environment.json`, session-start hooks, IDE-only notes) belongs under `.cursor/`, not here.

## Trunk

Name the integration branch this repo fast-forwards onto (usually `main`). Everywhere this file says **trunk**, substitute that branch name. Per-repo exceptions belong in the install notes / `.cursor/trunk`, not in this template line.

## Merge

Squash the PR to **one commit**, then **fast-forward** onto trunk. That squash commit **is** HEAD of trunk.

- No merge commits
- Rebase-merge is **not** the path (it keeps N commits)
- GitHub’s “Squash and merge” is the button
- **Do not merge** unless Jonathan explicitly says so for that PR (`gh pr merge --squash` is still a merge)
- Never `gh pr merge --merge`. Do not `--rebase` unless he says so for that PR

The squash SHA differs from the PR head. Treat the **code** as identical. Do not write SHA-dependent tests.

## CI and deploy

Test on the PR (the code that becomes trunk). After squash+FF, **deploy immediately**. Do **not** re-run format/compile/test on push to trunk (that is how a post-merge red happens after deploy already shipped). Trunk workflows may deploy. Those deploy workflows need `concurrency: group: deploy-production` and `cancel-in-progress: true` so two pushes cannot race and land the older SHA last.

Branch protection must **require** those PR checks so untested code cannot merge.

When CI fails on a PR, notify or resume the agent that owns that branch. Do not poll. Do not merge to “fix” CI.

## GitHub settings (human, once per repo)

Settings → General → Pull Requests:

- Allow merge commits: **off**
- Allow squash merging: **on**
- Allow rebase merging: **off**

Settings → Branches → rule on trunk:

- Require linear history: **on**
- Require the PR checks before merge

Bots do not flip admin settings from a shared machine.

## Git hooks

If this repo has `.githooks`, environment setup must set `core.hooksPath=.githooks`. Do **not** `git commit` or `git push --no-verify` unless Jonathan says so. CI is the backstop, not the only gate.

## Incomplete work

The Bot that owns this repo owns open PRs, CI, merge conflicts, and drafts. Check at the weekday 8:56 Europe/Madrid run and whenever a signal arrives. Act without waiting to be nudged. Stay silent if nothing is new.

When trunk moves: rebase remaining **non-parked** feature/`cursor/*` PRs. Skip PRs Jonathan has parked (do not nag, do not rebase).

## Do not

- Put tokens, keys, or secrets in this repo, in docs, or in chat
- Merge, spend, publish, or send external mail unless Jonathan says so
- Enable a live bot or production flag unless he says so
