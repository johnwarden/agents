# agents

Canonical, harness-agnostic configuration for coding agents: shared
instructions and shared skills. One source, vendored or linked everywhere.

## Layout

- `AGENTS.md` — standing instructions shared by every harness and every repo.
- `skills/` — skills in the SKILL.md format read by Codex, Claude Code, and Cursor.
- `bin/sync-agents-md` — copies `AGENTS.md` into a project between marker comments.
- `repo-setup/` — shipping-repo install notes and templates (merge/CI/deploy rules, toolchain standard, session-start hook).

## Local machine

Each harness points at this repo rather than holding its own copy.

| Harness | How it reads this repo |
|---|---|
| Claude Code | `~/.claude/CLAUDE.md` imports `@~/.agents/AGENTS.md`. |
| Codex | `~/.codex/AGENTS.md` is a symlink to `~/.agents/AGENTS.md`. Codex also reads `~/.agents/skills` directly. |
| Cursor (editor) | No global file. Paste `AGENTS.md` into Settings → Rules → User Rules by hand. |

`~/.agents` is itself a symlink to this checkout.

## Projects and cloud agents

Cloud agents only see what is committed in the repo they are working in.
Nothing in a home directory reaches them. So every project carries a copy of
the shared instructions at the top of its own `AGENTS.md`, between these
markers:

```
<!-- shared-agents:start (do not edit; run bin/sync-agents-md) -->
...shared text...
<!-- shared-agents:end -->
```

Project-specific instructions go below the end marker. To apply or refresh
the shared block in a project:

```
~/.agents/bin/sync-agents-md /path/to/project
```

Then commit the result in that project. An agent working inside a project
that cannot reach this repo can do the same thing by hand: fetch the current
`AGENTS.md` from this repository, replace everything between the markers
with it (or insert the block at the top if the markers are absent), and
leave the rest of the file as is.

Do not fetch this file at VM boot from an install or start script. Install
output is baked into snapshots and goes stale, start scripts are detached,
and the fetched file appears as an uncommitted change.

## Shipping a new repo

`repo-setup/` is the checklist and templates for a repo that ships (merge,
CI, deploy). The shared block above is mental models only. Shipping rules
are a separate project section:

1. Sync the shared block into the project's `AGENTS.md` (`bin/sync-agents-md`).
2. Copy `repo-setup/templates/AGENTS.shipping.md` below the shared-agents
   end marker. Set Trunk. Add a short project section under that.
3. Install `repo-setup/templates/session-start.sh` as
   `.cursor/session-start.sh` (executable) and set `environment.json`
   `"start"` to that path.
4. Follow `repo-setup/repo-standard.md` (devbox + direnv + just +
   `secrets.sh`) and the checklist in `repo-setup/install-agents-md.md`.

## Changing the shared instructions

1. Edit `AGENTS.md` here and commit.
2. Re-run `bin/sync-agents-md` in each project and commit there.

Local harnesses pick the change up on their next session with no further
steps.
