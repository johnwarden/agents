# Codex Cloud environments

Use this alongside `repo-standard.md`. The repository remains the source of
truth for tools, commands, instructions and dependencies. Codex supplies a
checkout and a reusable prepared environment. No Nix, Devbox or direnv in the
cloud: agents run `just <recipe>` directly.

## Connect the repository

1. Choose **Work in → Cloud → Select environment → Create environment**, or
   **Settings → Codex Cloud → Environments → Create environment**.
2. Select the project repository. If missing, choose **Missing a repository?
   Configure access**. Install/configure the **ChatGPT Codex Connector** GitHub
   App on the account or organization that owns the repo, select the intended
   repository, then refresh the picker.
3. Choose **Get started** to open environment setup.

A GitHub login alone is insufficient: the app needs repository access.
Reading a public repo through a connector does not prove that Codex can use it
as an environment repository. Prefer access to selected repositories.
Jonathan completes account connection and permission grants through the
secure sign-in/approval flow.

## Repository-owned setup

Commit shared `AGENTS.md` blocks with `bin/sync-agents-md`, following
`install-agents-md.md`. Never fetch or rewrite shared instructions at VM boot.
Commit skills the cloud task needs; local home-directory skills are not
automatically copied into the cloud.

Keep the bootstrap in `.codex/install.sh`. This is our convention, not an
automatically discovered Codex configuration file. Set **Install script** to:

```bash
bash .codex/install.sh
```

The committed script must:

- Resolve the repository root from its own location.
- Provide the runtime and every tool needed by `just check`, including
  `just`, `git`, and linters. Keep versions aligned with Devbox and CI.
- Follow the pinned-download and committed SHA-256 rules in
  `repo-standard.md`. Never use an unverified `curl | sh` installer.
- Make tools available in subsequent task shells. A temporary `export PATH`
  in the installation process is insufficient; verify from a fresh task.
- Set `git config core.hooksPath .githooks` when hooks exist.
- Run `just setup` to install locked dependencies idempotently.
- Require no model credentials, vault unlock or production tokens for setup
  and checks.

Use **Start skill** for task-start instructions. It should run the repo's
cheap, idempotent dependency setup after confirming the checkout, restore
hooks if necessary, and report failures before coding. Keep shell operations
in a committed script or `just` recipe instead of duplicating them in the UI.

Do not assume a new task refreshes Git refs. In the first Pi environment,
a fresh task restored the setup branch and stale origin/master from the
published filesystem, even though the PR had already been squash-merged.
Check the actual branch, HEAD, tree SHA, and worktree status. Before publishing,
explicitly fetch and switch/fast-forward a clean setup checkout to the approved
base commit when needed; then verify that commit in a new task. Do not reset,
rebase, or overwrite in-progress work. Do not copy Cursor's detached fetch/rebase
hook without checking the actual lifecycle. Normal PR handling still follows
the shared shipping rules.

## Prepare, publish and verify

1. Allow setup to reach the package registries and source/release hosts its
   scripts use. Use the package-manager preset or a narrow domain list.
   Verify task-time network access separately from installation access.
   Agent-proposed configuration changes remain drafts: review them in the
   Environment panel and click **Save draft** to apply them. In the initial
   Pi setup, `api.github.com` was needed for PR creation, and `api.jetify.com`
   plus `search.devbox.sh` for Devbox lockfile generation. Add hosts only when
   the actual setup requires them; do not assume the preset covers subdomains.
2. Run the install script, then `just check`. Check runtime, `just`, Git and
   linter versions in the setup environment.
3. Verify the setup checkout is clean and at the intended committed base.
   Review the Install script and Start skill, then **Publish**. This makes
   the prepared filesystem available to new tasks; it does not deploy the app.
4. Launch a fresh task. Verify repository, checkout, tools, hooks and
   `just check` again before claiming the environment is ready.

After toolchain or install-script changes, edit setup and **Republish**.
Updating Git refs does not rerun installation or rebuild the toolchain.
Existing tasks retain their own files; verify changes in a new task. Do not
delete an existing environment just to refresh it.

Load secrets only in recipes that need them, through the allowlisted
environment / `SECRETS_DIR` / Bitwarden convention in `repo-standard.md`.
Do not bake vault credentials or authenticated Pi session files into the
published filesystem. Unit checks and scripted Pi smoke tests need no live
model account.

## Minimal Pi experiment

- Pin Pi and `fdietze/pi-infinite-context` to exact versions or commits and
  commit the package lockfile. Use project dependencies, not floating globals.
- Expose setup, launch and checks through `just setup`, `just dev` and
  `just check`. Mark live model recipes as potentially paid.
- Start with a credential-free scripted-provider smoke test exercising real
  Pi extension loading and folding. Add model authentication when interactive
  use is needed.
- Vendor shipping instructions with `--shipping --no-deploy`. Open PRs for
  subsequent changes; merge only with Jonathan's approval for that PR.

Watching a Codex task and watching a Pi terminal are separate requirements.
Do not promise an ordinary ChatGPT Work chat exposes the shell's command
stream or an interactive Pi TUI. Verify the actual task interface first.

## Older environment UI

Some accounts expose **Setup script** / **Maintenance script** instead.
Call the committed install script from Setup and a cheap dependency/hook
refresh from Maintenance. Do not assume these fields coexist with Install
script / Start skill. Follow the UI presented and test both initial setup
and a fresh cached task.

References (checked 2026-10-08):

- [Current cloud environments](https://learn.chatgpt.com/docs/environments/cloud-environments)
- [Earlier environment workflow](https://learn.chatgpt.com/docs/environments/cloud-environment)
- [GitHub repository access](https://help.openai.com/en/articles/11145903-connecting-github-to-chatgpt)
