# Cursor cloud agent toolchain: worked example (context-bot, Elixir)

This is a dated log of how `social-protocols/context-bot` got its cloud environment right. It is an example, not the standard. The general rules for cloud VMs are in `repo-setup/repo-standard.md` § 4; follow those and use this file for the reasoning and the pitfalls.

Agents on default Ubuntu VMs will not have `mix` / `devbox` / `direnv`. Observed 2026-08-25: `which mix` missing even though `.cursor/environment.json` existed.

Do **not** install Nix/devbox/direnv on every VM from scratch. Too heavy on cold start, and direnv only exports vars that **do not survive** a Cursor Build snapshot (disk only; shell exports are dropped).

## What the repo actually pins

- Elixir **1.20.2**, OTP **28.5.0.3** (`mix.exs` `~> 1.20`; production `hexpm/elixir:1.20.2-erlang-28.5.0.3-…`)
- Toolchain on the laptop/CI is Devbox + direnv (`.envrc` only loads Devbox). No `.tool-versions`.
- DB is **SQLite**, not Postgres.
- Format: `just format` → `mix format` + `shfmt`. `mix format` is enough for `.ex`/`.exs`.
- Existing `.cursor/environment.json` `install` was `direnv allow` then `just setup` (a no-op without those binaries). Extra keys `repos` / `setupExplanation` are outside the official schema.
- Do not reuse the production Dockerfile as the Cloud Agent Dockerfile (it `COPY`s the app).

## Do this instead

Per-environment, not account-wide.

1. Base image has Erlang + Elixir on PATH (`FROM hexpm/elixir:1.20.2-erlang-28.5.0.3-…` or agent-driven setup). https://cursor.com/dashboard/cloud-agents#environments
2. Schema-valid `.cursor/environment.json` `install`: `mix local.hex --force && mix local.rebar --force && mix deps.get` (idempotent). Optionally compile. Not Postgres.
3. Enable **Builds** so later agents boot from a prepared snapshot. Docs: https://cursor.com/docs/cloud-agent/setup and https://cursor.com/docs/cloud-agent/builds
4. Skip direnv. Skip Nix unless laptop parity is the actual goal.

`mix format` needs Elixir only. Tests that need SQLite compile (`ecto_sqlite3`) belong after deps + headers, not a full Devbox.

Do not store vault tokens on machines that should not hold them, and do not log into Cursor as Jonathan from those machines. Context Bot PRs `environment.json`; Jonathan (or a cloud setup agent) applies it on that environment.

## Merging environment.json does not rebuild the snapshot

Launching with an existing Personal environment uses that **Personal snapshot**. Merging `.cursor/environment.json` does not rebuild it. Observed: new VM, no `mix` on PATH.

Jonathan: environment page → **Builds → Trigger build**. If install is still direnv/just, **Update with Agent** to the hexpm Dockerfile. Then a new cloud VM.

Do not drop a cloud environment that already has deploy tokens configured. Do not install Nix on the VM.

## Missing tools after a Dockerfile merge: stale Build (2026-10-08)

The repo's `.cursor/environment.json` takes priority over personal and team saved environments (Cursor docs, "Environment resolution order"). New agents still boot from the environment's **active Build**. If that Build was made before the `environment.json`/`.cursor/Dockerfile` change merged, a fresh agent comes up without the Dockerfile's tools.

1. Environment page → Builds → **Trigger New Build**. "Update Stale Builds" only pulls code and does not rebuild the image.
2. Once the new Build is active, check the tools in a fresh agent (`which just git shellcheck` plus the language runtime) and make one GitHub-Verified commit.
3. Only if tools are still missing: check the environment page's Config/Source field. If it does not already say Repository file, the repo owner switches it and triggers a new build. Don't ask for a switch-over before checking that field.

Don't write environment IDs into repos or docs.

## Quality gate (2026-08-25)

Cursor cloud agents on `social-protocols/context-bot` must not `git commit --no-verify` or `git push --no-verify` unless Jonathan says so for that commit. Put that in root `AGENTS.md` (not launch-prompt paste).

Local hooks (after https://github.com/social-protocols/context-bot/pull/36): `git config core.hooksPath .githooks` in env install, then rebuild/activate the environment. Prefer hooksPath over copying into `.git/hooks`. Pre-commit: format + compile. Pre-push: `mix test`. CI is the backstop, not the only gate.

Cannot physically block `--no-verify` on the VM.

## Fetch before work (2026-09-02)

Jonathan: cloud agents must `git fetch origin` and update to current `origin/main` **before starting work**, not only before opening a PR. Rebase the task branch onto `origin/main` if already on a feature branch. Do not assume the VM checkout or Cursor Build snapshot is current (Builds freeze `main`). Keep the existing environment. Do not drop it. Do not install Nix.

## Session start (2026-09-02)

Jonathan: the agent should not fetch/pull/rebase. Put that in `.cursor/environment.json` `start` (every agent boot): hooksPath, fetch, ff-only trunk to `origin/<trunk>`, rebase feature branches (abort on conflict). Optional `.cursor/trunk` names the integration branch (default `main`). A repo with no single trunk may ship a fetch-only start (no ff/rebase) when `.cursor/trunk` is absent, if it documents why; do not change the template's default. `install` still sets hooksPath at Build time. General standing rules live in repo `AGENTS.md`; Cursor-only notes under `.cursor/` / `repo-setup/cursor-repo-notes.md` — not CloudAgent launch prompts.
