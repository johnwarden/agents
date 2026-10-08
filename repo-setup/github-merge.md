Agent-facing copy for every shipping repo: `repo-setup/templates/AGENTS.shipping.md`, vendored into each project's `AGENTS.md` by `bin/sync-agents-md --shipping` (Cursor bits under `.cursor/`). Install: `repo-setup/install-agents-md.md`.

# PR merge rule

Jonathan: **squash to one commit, fast-forward onto trunk** (usually `main`). That squash commit is the new HEAD of trunk. No merge commits. Rebase-merge is not the path (it keeps N commits). All repos we ship (including `social-protocols/context-bot`). Bots do not merge unless he says so.

GitHub’s “Squash and merge” is the button. The squash SHA differs from the PR head; Jonathan treats the **code** as identical and does not want a second test suite on `main` for that. Ignore SHA-dependent tests.

Flow: test on the PR (what becomes `main`) → squash+FF → deploy as soon as that lands. Deploy-on-main workflows must use a single concurrency group with `cancel-in-progress: true` (`repo-setup/github-deploy.md`) so a slower older deploy cannot overwrite a newer one. Do **not** re-run format/compile/test on push to `main` (that is how a post-merge red happens after Fly already shipped). `main` workflows may deploy. Branch protection: require the PR checks so untested code cannot merge.

Merge commits are still banned (untested merge node, extra history).

## GitHub UI (each repo)

Settings → General → Pull Requests:

- Allow merge commits: **off**
- Allow squash merging: **on**
- Allow rebase merging: **off**

Settings → Branches → rule on `main` (or default):

- Require linear history: **on** (FF-only)

Bots: `gh pr merge --squash` is still a merge. Do not merge without his yes. Never `--merge`. Do not `--rebase` unless he says so for that PR.

## Cursor / gh

Do not put GitHub admin on a shared or unattended machine to flip this; Jonathan sets it once per repo.
