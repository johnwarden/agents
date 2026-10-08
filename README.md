# agents

Canonical, harness-agnostic configuration for coding agents: shared
instructions, shared skills, and the repo-setup standard. One source,
vendored or linked everywhere.

## Layout

- `AGENTS.md` — standing instructions shared by every harness and every repo (mental models).
- `skills/` — skills in the SKILL.md format. Codex reads this directory directly; other harnesses need a copy or link.
- `bin/sync-agents-md` — vendors the shared blocks into a project's `AGENTS.md`.
- `repo-setup/` — the shipping-repo standard: install recipe, shipping-rules template, session-start hook, toolchain standard, merge/deploy rationale.

## Local machine

Each harness points at this repo rather than holding its own copy.

| Harness | How it reads this repo |
|---|---|
| Claude Code | `~/.claude/CLAUDE.md` imports `@~/.agents/AGENTS.md`. |
| Codex | `~/.codex/AGENTS.md` is a symlink to `~/.agents/AGENTS.md`. Codex also reads `~/.agents/skills`. |
| Cursor (editor) | No global file. Paste `AGENTS.md` into Settings → Rules → User Rules by hand. |

`~/.agents` is itself a symlink to this checkout.

## Projects and cloud agents

Cloud agents only see what is committed in the repo they are working in.
Nothing in a home directory reaches them. So every project carries copies
of the shared text at the top of its own `AGENTS.md`, each between marker
comments:

```
<!-- shared-agents:start (do not edit; run bin/sync-agents-md) -->
...mental models, from AGENTS.md here...
<!-- shared-agents:end -->

<!-- shared-shipping:start trunk=main deploy=yes (do not edit; run bin/sync-agents-md) -->
...shipping rules, from repo-setup/templates/AGENTS.shipping.md...
<!-- shared-shipping:end -->

...project-specific instructions...
```

The shared-agents block goes in every repo. The shared-shipping block goes
in repos that ship (merge, CI, deploy). Project-specific instructions go
below the blocks and are never touched by the script.

```
~/.agents/bin/sync-agents-md /path/to/project                  # shared block only
~/.agents/bin/sync-agents-md /path/to/project --shipping       # both blocks
~/.agents/bin/sync-agents-md /path/to/project --shipping --trunk master --no-deploy
```

The trunk name and deploy choice are recorded in the shipping start marker,
so a later plain re-run keeps them. Commit the result in the project.

A cloud agent working inside a project reaches this repo through the GitHub
App that grants it private-repo access. It can run the script from a clone,
or do the same by hand: fetch the current source file, replace everything
between the matching markers, and leave the rest of the file as is.

Do not fetch these files at VM boot from an install or start script. Install
output is baked into snapshots and goes stale, start scripts are detached,
and the fetched file appears as an uncommitted change.

## Setting up a repo that ships

Follow `repo-setup/install-agents-md.md`. It is the single procedure for
new repos, repos that already have an `AGENTS.md`, and repos that have none.

For Codex Cloud, also follow `repo-setup/codex-cloud-env.md`: connect the
repository, configure its install script and Start skill, publish the
prepared environment, and verify a fresh task.

## Changing the shared text

1. Edit `AGENTS.md` or `repo-setup/templates/AGENTS.shipping.md` here and commit.
2. Re-run `bin/sync-agents-md` in each project and commit there. The script
   replaces whichever blocks the project already carries.

Local harnesses pick up `AGENTS.md` changes on their next session with no
further steps.
