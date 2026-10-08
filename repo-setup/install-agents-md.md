# Install AGENTS.md in every shipping repo

There is **no** account-wide master `AGENTS.md`. Agents only reliably read the copy in **that** repo. The canonical sources live in this repository and are vendored into each project by `bin/sync-agents-md`:

- `AGENTS.md` → the `shared-agents` block (mental models, every repo)
- `repo-setup/templates/AGENTS.shipping.md` → the `shared-shipping` block (repos that ship)

Do **not** paste machine-local paths into the in-repo file.

Rationale and detail: merge/CI in `repo-setup/github-merge.md`, deploy lock in `repo-setup/github-deploy.md`, Cursor-only notes in `repo-setup/cursor-repo-notes.md`, incomplete work in `repo-setup/incomplete-work.md`, toolchain in `repo-setup/repo-standard.md`. The bot that owns the repo installs GitHub merge/CI wake listeners, required for every shipping repo.

## Procedure

1. **Sync the blocks.** From a clone of this repo:

   ```
   bin/sync-agents-md /path/to/project --shipping [--trunk NAME] [--no-deploy]
   ```

   `--trunk` defaults to `.cursor/trunk` if present, else `main`. Use `--no-deploy` for a repo with no deploys; the block then carries the CI-only variant. Both choices are recorded in the shipping start marker and kept on later re-runs.

2. **Remove hand-copied duplicates.** If the project's `AGENTS.md` already contained the shipping rules outside the markers (older repos do), delete those sections so the rules exist only inside the block. Keep everything project-specific.

3. **Add or keep the project section** below the blocks: toolchain, how to run tests, deploy. Follow `repo-setup/repo-standard.md` (devbox + direnv on laptops, `just` as the only command surface, `secrets.sh`). The project section must say: "Commands are `just` recipes. Run `just check` before claiming done (on a laptop, `direnv exec . just check`). Secrets only via `secrets.sh` in the recipes that need them." Cloud VMs have no direnv, so never make `direnv exec` the only documented way.

4. **Install harness-specific environment files.** For Codex Cloud, follow
   `repo-setup/codex-cloud-env.md` for the committed bootstrap and environment
   configuration. For Cursor, install the following:
   - `repo-setup/templates/session-start.sh` → `.cursor/session-start.sh` (executable)
   - `.cursor/environment.json` with `"start": ".cursor/session-start.sh"` (create a minimal file if the repo has none)
   - `.cursor/trunk`: one line naming the integration branch. Omit for `main`. A repo with a multi-line branch model documents its targets in `AGENTS.md` and may omit it.
   - Cursor-only guidance from `repo-setup/cursor-repo-notes.md` goes under `.cursor/`, not in root `AGENTS.md`.

   `session-start.sh` sets `core.hooksPath` when `.githooks` exists, fetches, fast-forwards trunk (`merge --ff-only`), and rebases a feature branch onto `origin/<trunk>` (aborts on conflict). Agents should not have to update git. A repo with no single trunk may ship a start that only fetches when `.cursor/trunk` is absent, as long as it documents why. Do not change the template's default behaviour.

5. **Deploy workflows.** On any workflow that deploys on push to trunk, set `concurrency: group: deploy-production` and `cancel-in-progress: true` (`repo-setup/github-deploy.md`). Not a per-SHA group on that path.

6. **GitHub settings.** Jonathan sets squash+FF and required PR checks once per repo (listed in the shipping block). Bots do not flip admin settings.

7. **Wake listeners.** The bot that owns the repo installs after-merge and, when it uses Cursor cloud agents, CI-failure listeners (`repo-setup/incomplete-work.md`).

8. **Open a PR.** Do not merge unless Jonathan says so.

## Re-syncing after the shared text changes

Run step 1 again with no extra flags and commit. The script replaces the blocks and keeps the recorded options.

## Naming

Do **not** put the LLC name in shipping PRs, `AGENTS.md`, or other in-repo markdown. If a personal name is needed, use Jonathan’s. The shipping block is general-purpose agent policy: no Cursor product surface, no machine-local paths.
