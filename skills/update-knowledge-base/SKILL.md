---
name: update-knowledge-base
description: Use after research, exploration, or non-trivial debugging that produced a reusable lesson worth keeping for future work in this repo. Captures the detail in a dated report under `knowledge-base/reports/` and a one-sentence summary in `knowledge-base/learnings.md` (which is loaded into agent context, so it must stay terse).
---

# Updating the knowledge base

The knowledge base has two parts:

1. **`knowledge-base/reports/YYYY-MM-DD-slug.md`** — long-form record of an investigation: what was asked, what was tried, what was found, what was concluded. Read on demand.
2. **`knowledge-base/learnings.md`** — a single bullet list of one-sentence facts/constraints, each linking to its source report. This file is loaded into every agent's context, so it must stay tight.

## When to invoke

Invoke this skill when *any* of the following hold:

- An investigation revealed something non-obvious about the code, the data, or an external tool/library.
- A debugging session uncovered a constraint, gotcha, or root cause worth remembering for future work.
- An exploration *ruled out* an approach (negative results matter — they save future time).
- A user explicitly asks to record a finding or "remember this".

Do **not** invoke for:

- Routine fixes whose diff is self-explanatory.
- Implementation work where the design is already documented in code or commits.
- Information that belongs in `git log`, `AGENTS.md`, or `AGENTS.md`.

## How to add an entry

**Resolve the repo root first.** Run `git rev-parse --show-toplevel` and use that path as the base for the knowledge base. If it fails (not a git repo), use the current working directory and warn the user once that the knowledge base will live there. All paths below are relative to that root.

### 1. Write the report

Path: `knowledge-base/reports/YYYY-MM-DD-short-slug.md` (today's date in ISO; slug = 2-5 words).

`knowledge-base/` and `knowledge-base/reports/` may not exist yet — create them on first use without prompting.

Suggested structure (no rigid template — write what's useful):

```markdown
# {Title}

**Date:** YYYY-MM-DD
**TL;DR:** One or two sentences that someone could read instead of the rest.

## Context
What prompted this investigation? What were we trying to decide or fix?

## Investigation
What was tried, in what order. Include commands, file paths with line numbers,
corpus counts, URLs — embed evidence directly so the report is self-contained.

## Findings
The actual answer. Be specific.

## Implications
What this means for future work. Things to do, things to avoid, code to trust
or distrust. This is the section that feeds the learnings file.
```

### 2. Update `knowledge-base/learnings.md`

If the file does not exist yet, create it with this header:

```markdown
# Learnings

One-sentence facts and constraints distilled from investigations in this repo. Loaded into every agent's context — keep terse. Each line links to a longer report under `reports/` for full detail.
```

Then add a bullet for each new learning:

```markdown
- {one-sentence learning, self-contained, ≤150 chars} ([reports/YYYY-MM-DD-slug.md](reports/YYYY-MM-DD-slug.md))
```

Rules for the bullet:

- State the **constraint or fact**, not the activity. ✅ "podr- pudrir forms are absent from the 547M-token corpus." ❌ "Investigated podrir frequency."
- Self-contained: readable without opening the report.
- Under ~150 characters. If it won't fit, the learning is probably two learnings.
- Group related lines under a `## Heading` only once you have 3+ on a topic.
- If a new learning **supersedes or contradicts** an existing entry, edit the existing entry — don't append a second.

### 3. Make sure it gets loaded into agent context

`knowledge-base/learnings.md` is only useful if agents see it. Pick the auto-load file with this priority:

1. If `AGENTS.md` exists at the repo root, use it.
2. Else if `AGENTS.md` exists, use it.
3. Else create a minimal `AGENTS.md` stub at the repo root with a short title (e.g. `# {repo-name}`) and the wiring line below.

Append (or insert near the top of) this exact line, if not already present:

```
Always-on context: see `knowledge-base/learnings.md` for distilled facts and constraints from prior investigations.
```

Be idempotent — check first with something like `grep -q 'knowledge-base/learnings.md' AGENTS.md AGENTS.md 2>/dev/null` so re-runs don't duplicate the line.

## Output

After writing, briefly tell the user: which report file was created, the one-line learning(s) that were added, and which doc (if any) was touched to wire `learnings.md` into agent context. Don't dump the report contents back at them — they just wrote them.
