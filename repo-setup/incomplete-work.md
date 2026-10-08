# Incomplete work (all Bots)

Jonathan, 2026-08-26: every Bot owns unfinished work. He should not have to nudge.

- At the weekday 8:56 Europe/Madrid check, and whenever a signal arrives, look at open tasks: PRs, CI, merge conflicts, drafts, anything waiting on you.
- Act on what you own. Do not wait for him to poke you.
- Stay silent if nothing is new. Do not re-nag an already-stated blocker.
- No email, spend, publish, or merge without his yes.

## Signals (shipping GitHub)

Weekday cron alone is not enough. For every shipping repo, the bot that owns the repo installs GitHub listeners that ping the owning Bot:

1. **After merge** (`pr-merged`) — owning Bot continues incomplete work the merge unblocked (rebases, release assemble, next step).
2. **CI attach factory** (`pr-opened` → PR-scoped `ci-failed` watcher) when that Bot uses Cursor cloud agents on the repo.

The bot that owns the repo installs these listeners. They ping the owning Bot; they do not launch VMs or merge.
