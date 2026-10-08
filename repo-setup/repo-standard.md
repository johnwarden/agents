# Repo standard: devbox + direnv + just + secrets.sh

**Policy (Jonathan, 2026-10-08).** Every new project uses this setup, with `social-protocols/context-bot` as the template. Anyone can do this:

```bash
git clone <repo> && cd <repo>
direnv allow      # devbox pins the tools; init_hook builds the language envs
just              # lists the commands: test, lint, check, dev, deploy, ...
```

No other manual setup step. Existing repos adopt it when next touched. The per-shell setup hook is devbox's `shell.init_hook` in `devbox.json`.

## 1. Toolchain: devbox + direnv

- `devbox.json` + committed `devbox.lock` pin **every** tool the repo needs: the language runtime, `just`, `jq`, `bitwarden-cli` when the repo has secrets, linters, and deploy CLIs. Don't rely on globally installed tools. context-bot pins Elixir/OTP, just, flyctl, bitwarden-cli, docker-client, shellcheck, shfmt, sqlite.
- `.envrc` **only** loads devbox, which is context-bot's file verbatim:
  ```bash
  eval "$(devbox generate direnv --print-envrc)"
  ```
  No exports, no secrets, no `dotenv` in `.envrc`. Non-secret local config goes in a gitignored `.env`, read by `just` via `set dotenv-load := true`.
- `shell.init_hook` creates or refreshes the language environment, so `direnv allow` is enough. It must be **idempotent and cheap** because it runs on every shell entry:
  - Elixir (context-bot): `mix local.hex --if-missing --force`, `mix local.rebar --if-missing --force`, `if [ -f mix.exs ]; then mix deps.get; fi`, plus `SSL_CERT_FILE=$NIX_SSL_CERT_FILE`.
  - Python: `uv` from devbox, then `uv sync` (or `[ -d .venv ] || uv venv && uv pip sync requirements.txt`), then `source .venv/bin/activate`. Existing Python repos may do this with venv+pip.
  - Node: `npm ci` **only when the lockfile changed**, for example `[ node_modules/.package-lock.json -nt package-lock.json ] || npm ci`. A bare `npm ci` on every `cd` wipes `node_modules`.
- Agents and non-interactive shells run `direnv exec . just <recipe>`. That is context-bot `AGENTS.md` § Environment.

## 2. Command surface: justfile

- The justfile is **the** command surface. Docs, AGENTS.md, CI and hooks call `just <recipe>`. `npm run`, `mix`, `pytest` and `npx` appear only *inside* recipes, never as the documented way to do something.
- The first recipe is `default: @just --list`, so a bare `just` lists commands. Give every recipe a `# comment` so the list explains itself. context-bot also has `help: @just --list --unsorted`.
- Standard short names, when they apply:
  | recipe | meaning |
  |---|---|
  | `setup` | explicit re-run of what init_hook does (deps, DB); safe to repeat |
  | `dev` | run the app locally (server or watcher) |
  | `test [path]` | unit tests; optional single path |
  | `lint` | linters (incl. shellcheck on repo shell scripts) |
  | `fmt` / `format-check` | write / verify formatting (context-bot names them `format` / `format-check`) |
  | `typecheck` | static types |
  | `check` | **every CI gate**: format-check, lint, typecheck, test |
  | `deploy` | ship; loads secrets; needs Jonathan's explicit yes |
  | `secrets` | validate that the allowlisted secrets load, without printing them |
  Project-specific recipes (`dry-run`, `live-run`, `db-migrate`, `fly-logs`, ...) are fine. Mark recipes that spend money or have external effects in their comment, for example `# PAID: ...` or `# May publish ...`.
- Use `set shell := ["bash", "-euo", "pipefail", "-c"]` and multi-line `#!/usr/bin/env bash` + `set -euo pipefail` bodies. Use bash, not zsh, so recipes run in CI and in cloud VMs.
- CI installs `just` (a pinned release binary, as in context-bot's `ci.yml`) and runs `just check`, or the individual recipes `check` is made of. Nothing in CI calls `npm`, `mix` or `pytest` directly.
- **Every pinned tool download is hash-verified.** This covers CI, `.cursor/Dockerfile`, and any script that fetches a release binary. Pin the exact version and its sha256 in the committed file, and run `sha256sum -c` before you extract or run the download. Take the expected hash from these sources, in order:
  1. **GitHub release-asset digest (first choice):** `gh api repos/OWNER/REPO/releases/tags/TAG --jq '.assets[] | "\(.name) \(.digest)"'` (drop the `sha256:` prefix).
  2. **Fallback, only when that asset has no digest:** the project's own published checksums file (`checksums.txt`, `SHA256SUMS`, ...) from **the same release**. Assets uploaded before about mid-2025 have no digest (Hugo 0.131 and 0.145, for example). Prefer a signed checksums file (cosign/sigstore, GPG, minisign) and check the signature when one is published. Add a comment saying which source the pinned hash came from.
  - If neither source exists, don't use that download. Pick another release, or install the tool another way (distro package, devbox). **Never allow an unverified download:** no `curl | tar` or `curl | sh` without a pinned hash. A checksums file fetched at build time from the same release does not count as verification on its own; the expected hash must be committed.
- Git hooks (`.githooks/`, `core.hooksPath`) call recipes too: pre-commit runs fast format/compile, pre-push runs `just test`.

## 3. Secrets: secrets.sh

Secrets live in Bitwarden. `secrets.sh` turns a short list of **allowlisted names** into env vars, only for the recipe that asks for them.

- **context-bot's `secrets.sh` is committed and contains no values.** It is an allowlisted loader (`FLY_API_TOKEN | SECRET_KEY_BASE | BOT_APP_PASSWORD | ANTHROPIC_API_KEY`). The only per-machine input is `BITWARDEN_ITEM_ID`, a non-secret pointer in gitignored `.env` (`.env.example` is committed). New repos copy this pattern. The older pattern, an untracked `secrets.sh` copied from a committed `secrets.example.sh`, is tolerated in existing repos. If a repo uses it, `secrets.sh` must be in `.gitignore`.
- **Loader behaviour, copied from context-bot:**
  - `source ./secrets.sh NAME [NAME...]` requires at least one name and refuses names outside the allowlist.
  - It disables `set -x` while values are in memory and restores it after.
  - It reads one `bw get item "$BITWARDEN_ITEM_ID"`. That needs `bw` unlocked, so `export BW_SESSION="$(bw unlock --raw)"` first.
  - It pulls custom fields by exact name and rejects empty or multi-line values.
  - It exports only the requested names, and only after all of them load.
  - It unsets the Bitwarden payload and temporary variables, and prints `secrets: loaded NAME` without the value.
  - Values never go in argv. To hand them to another tool, pipe them over stdin, for example `printf ... | fly secrets import`.
- Recipes source it **only where needed**, such as `deploy`, `secrets-sync`, and live or paid runs, each asking for the minimum names. Never source it from `.envrc`, `init_hook`, `test` or `check`. CI and unit tests need no secrets.
- Add a test like context-bot's `test/secrets_test.sh`. It stubs `bw` and asserts that only the requested names are exported and that no value appears in output. Run it from `just test`. Put `shopt -s inherit_errexit` right after `set -euo pipefail`: without it, checks inside `$( ( ... ) 2>&1 )` capture blocks don't inherit `set -e` and silently pass.
- No partial-secret echo recipes.

### Machines without Bitwarden

Some machines have no Bitwarden (do not put Jonathan's vault on them). On those machines, a secret that is already exported stays exported, or else is one file per env var under `$SECRETS_DIR` (0700 dir, 0600 files). context-bot's loader has no such fallback: it unsets the requested names and requires `bw`. This is the convention for new loaders:

For each requested allowlisted `NAME`, in order:
1. If `NAME` is already exported and non-empty, keep it.
2. Else, if `SECRETS_DIR` is set and `"$SECRETS_DIR/NAME"` is a readable file, read it with trailing CR/LF stripped.
3. Else, use Bitwarden as above.

Never write a machine-local path into the repo. The in-repo loader only knows the generic `SECRETS_DIR`. Same rules apply: allowlist, no echo, export only requested names. Laptop users who want strict Bitwarden-only behaviour just leave `SECRETS_DIR` unset and don't pre-export.

## 4. Cursor cloud VMs

Rules below; `repo-setup/cursor-cloud-env.md` is a worked example from context-bot (Elixir). **No Nix, devbox or direnv on cloud VMs.** Too heavy on cold start, and direnv exports don't survive a Build snapshot.

- `.cursor/Dockerfile` (via `environment.json` `build`) provides the toolchain from a language base image, **including `just`, `git`** (slim base images lack it, so `.cursor/session-start.sh` fetch/rebase and agent commits silently fail) and the non-runtime tools in `devbox.json` (e.g. `shellcheck`, `jq`), so `just lint` works there. context-bot keeps them aligned with a test, `test/cursor_dockerfile_devbox_align_test.sh`.
- After changing `.cursor/environment.json` or `.cursor/Dockerfile`, trigger a **new Cloud Agents build** ("Update Stale Builds" pulls code but does not rebuild the image) and verify `node`/`just`/`shellcheck` versions **and one GitHub-Verified commit** in a fresh agent.
- **Missing Dockerfile tools in a fresh agent usually means a stale active Build**, made before the `environment.json`/Dockerfile change merged. The fix is **Trigger New Build** on the environment's Builds tab; "Update Stale Builds" only pulls code and keeps the old image. Then check the tools in a fresh agent (`which just git shellcheck` plus the language runtime). The repo's `.cursor/environment.json` takes priority over personal and team saved environments, so no switch-over is normally needed. Before asking anyone to change it, check the environment page's Config/Source field. Only if it does not already say Repository file does the repo owner need to switch it and then trigger a new build.
- Cloud base images must have **glibc ≥ 2.38**: Cursor's commit signing runs `/exec-daemon/ssh-keygen`, which fails on Debian bookworm (2.36) and leaves agent commits unsigned. Use a `trixie` (2.41) or Ubuntu 24.04 base, e.g. `node:22.20-trixie-slim`; context-bot's hexpm `debian-trixie` image is fine. Tests that make throwaway git repos should run git with `GIT_CONFIG_GLOBAL=/dev/null GIT_CONFIG_NOSYSTEM=1` so machine signing config can't break them.
- `environment.json` `install` installs deps idempotently, does what init_hook does, and runs `git config core.hooksPath .githooks`. `start` is `.cursor/session-start.sh` (`repo-setup/install-agents-md.md`).
- In the cloud, agents run `just <recipe>` directly, without `direnv exec`. Recipes therefore must not depend on direnv-only exports.

## 5. New-repo checklist

- [ ] `devbox.json` (runtime, `just`, `jq`, `bitwarden-cli` if secrets, linters) + `devbox.lock` committed
- [ ] `.envrc` = devbox loader only
- [ ] `justfile` with `default: @just --list`, the standard recipes, and `check` = all CI gates
- [ ] `secrets.sh` (value-free allowlisted loader with the `$SECRETS_DIR` fallback) + `.env.example` with `BITWARDEN_ITEM_ID=` (or legacy `secrets.example.sh` + gitignored `secrets.sh`)
- [ ] `.gitignore`: `/.devbox/`, `/.direnv/`, `/.env`, `/.env.*`, `!/.env.example`, the language env dir (`.venv/`, `node_modules/`, `deps/` `_build/`), plus `secrets.sh` only for the legacy pattern
- [ ] `.githooks/pre-commit` + `pre-push` calling recipes; `core.hooksPath` set in Cursor `install`
- [ ] CI installs pinned, sha256-verified `just` (release-asset digest; same-release checksums file only if there is no digest) and runs `just check`
- [ ] `.cursor/Dockerfile` with `just`, `git`, `shellcheck` (and other devbox dev tools) + `environment.json` (`build`, `install`, `start`)
- [ ] README **Getting started**: `direnv allow`, then `just`, then `just dev` / `just test`, with prerequisites "Devbox, direnv hooked into your shell"
- [ ] Root `AGENTS.md` (`repo-setup/install-agents-md.md`) project section: "Commands are `just` recipes. Run `just check` before claiming done (on a laptop, `direnv exec . just check`). Secrets only via `secrets.sh` in the recipes that need them." Never make `direnv exec` the only documented way; cloud VMs have no direnv.

### Skeletons (generic, trimmed from context-bot)

`devbox.json`:
```json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/main/.schema/devbox.schema.json",
  "packages": { "nodejs": "22", "just": "latest", "jq": "latest", "bitwarden-cli": "latest", "shellcheck": "latest" },
  "shell": {
    "init_hook": [
      "[ node_modules/.package-lock.json -nt package-lock.json ] || npm ci"
    ]
  }
}
```

`justfile`:
```just
set shell := ["bash", "-euo", "pipefail", "-c"]
set dotenv-load := true

# List recipes
default:
    @just --list

# Install locked deps (init_hook does this on direnv allow)
setup:
    npm ci

# Run the app locally
dev:
    npm run dev

# Unit tests
test:
    npm test
    bash test/secrets_test.sh

# Linters
lint:
    npm run lint
    shellcheck secrets.sh

# Static types
typecheck:
    npm run typecheck

# Every CI gate
check: lint typecheck test

# Validate secrets load (prints names only)
secrets:
    #!/usr/bin/env bash
    set -euo pipefail
    source ./secrets.sh DEPLOY_TOKEN
    echo "secrets ok"

# Deploy. Needs Jonathan's explicit yes.
deploy:
    #!/usr/bin/env bash
    set -euo pipefail
    set +x
    source ./secrets.sh DEPLOY_TOKEN
    <deploy-cli> deploy
```

`secrets.sh` (outline; copy context-bot's file and edit the allowlist):
```bash
#!/usr/bin/env bash
# source ./secrets.sh NAME [NAME...]  — exports only requested allowlisted names; never prints values
# 1) keep NAME if already exported  2) else read "$SECRETS_DIR/NAME" if SECRETS_DIR set
# 3) else Bitwarden: bw get item "$BITWARDEN_ITEM_ID" (needs BW_SESSION) -> custom field NAME
case "$NAME" in OPENROUTER_API_KEY | DEPLOY_TOKEN) ;; *) fail "unsupported secret name";; esac
```

CI step:
```yaml
- name: Install just
  run: |
    # sha256 = GitHub release-asset digest of just-1.58.0-x86_64-unknown-linux-musl.tar.gz
    curl -fsSL -o /tmp/just.tgz https://github.com/casey/just/releases/download/1.58.0/just-1.58.0-x86_64-unknown-linux-musl.tar.gz
    echo "4a5cc2f53e6f0f8c59092a6cc38291eb729d46a7dd95d3ae582008881b84931d  /tmp/just.tgz" | sha256sum -c -
    sudo tar -xzf /tmp/just.tgz -C /usr/local/bin just
- run: just check
```

## Template differences worth knowing (as of 2026-10-08)

- context-bot's README Getting-started still has extra steps (`cp .env.example .env`, `set -a; source .env`, `just setup`, `just db-migrate`) before `just dev`. The target is the three-command flow above.
- context-bot's CI calls the `check` pieces one by one, plus one raw `MIX_ENV=test mix compile --warnings-as-errors`. New repos call `just check`.
- Do not copy a `default` recipe that builds instead of listing, zsh recipe bodies, or a legacy untracked `secrets.sh`.
- Do not run a full lockfile install on every `init_hook` entry. Do not hardcode secret values (including test keys) in recipes.
