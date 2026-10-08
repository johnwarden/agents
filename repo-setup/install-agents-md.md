# Install AGENTS.md in every shipping repo

There is **no** account-wide master shipping `AGENTS.md`. Agents only reliably read the copy in **that** repo. The canonical shipping template is `repo-setup/templates/AGENTS.shipping.md` in this repository. Copy it into each project's `AGENTS.md` **below** the shared-agents block (the mental models vendored by `bin/sync-agents-md`). Do **not** paste machine-local paths into the in-repo file.

Merge/CI detail: `repo-setup/github-merge.md`. Deploy lock: `repo-setup/github-deploy.md`. Cursor-only notes: `repo-setup/cursor-repo-notes.md`. Incomplete work: `repo-setup/incomplete-work.md`. The bot that owns the repo installs GitHub merge/CI wake listeners — required for every shipping repo.

Also install:

- `repo-setup/templates/session-start.sh` → `.cursor/session-start.sh` (executable)
- Wire `.cursor/environment.json` `"start": ".cursor/session-start.sh"`
- Optional `.cursor/trunk`: one line, integration branch name (omit for `main`; repos with a multi-line branch model document their targets in `AGENTS.md` and may omit `.cursor/trunk`)
- Cursor-only guidance from `repo-setup/cursor-repo-notes.md` under `.cursor/` (not in root `AGENTS.md`)

`session-start.sh` sets `core.hooksPath` when `.githooks` exists, `git fetch`es, fast-forwards trunk (`merge --ff-only`), and rebases a feature branch onto `origin/<trunk>` (aborts on conflict). The agent should not have to update git. A repo with no single trunk may ship a session-start that only fetches (no ff/rebase) when `.cursor/trunk` is absent, as long as it documents why. Do not change the template's default behaviour.

## Naming

Do **not** put the LLC name in shipping PRs, `AGENTS.md`, or other in-repo markdown. If a personal name is needed, use Jonathan’s. The shipping block in a project's `AGENTS.md` is general-purpose agent policy — no Cursor product surface, no machine-local paths.

## New repo (or new Bot that ships GitHub)

1. Sync the shared-agents block into the repo root `AGENTS.md` (`bin/sync-agents-md`). Copy `repo-setup/templates/AGENTS.shipping.md` **below** the end marker. Set the **Trunk** section to this repo’s integration branch.
2. Install `.cursor/session-start.sh`, `.cursor/environment.json` `start`, and `.cursor/trunk` when trunk ≠ `main`.
3. Add a short **project** section below the shipping rules (toolchain, how to run tests, deploy) — do not delete the shipping block. Move any Cursor Cloud-only notes under `.cursor/`.
   Toolchain and commands follow `repo-setup/repo-standard.md` (devbox + direnv, `just` as the only command surface, `secrets.sh`).
4. Jonathan sets GitHub squash+FF and required PR checks once (see `AGENTS.md`).
5. On any workflow that deploys on push to trunk, set `concurrency: group: deploy-production` and `cancel-in-progress: true` (`repo-setup/github-deploy.md`). Do not use a per-SHA group on that path.
6. The bot that owns the repo installs after-merge (and CI factory if Cursor cloud) wake listeners.
7. Open a PR. Do not merge unless he says so.

## Existing repo that already has AGENTS.md

Do **not** overwrite. Keep project-specific toolchain. Ensure the shared-agents block is at the top. Add or replace a clearly marked **shipping** section so it matches `repo-setup/templates/AGENTS.shipping.md` (merge, CI on PR only, deploy on FF, hooks, no `--no-verify`, no merge without his yes). Move Cursor-only content under `.cursor/`. Open a PR. Do not merge unless he says so.

## Existing repo with no AGENTS.md

Sync the shared-agents block, copy `repo-setup/templates/AGENTS.shipping.md` below it, set Trunk, then add project notes. PR, do not merge unless he says so.

## Who installs

The Bot that owns the repo. Canonical files and this recipe live in this repository.
