# Deploy lock (every shipping repo)

2026-09-02: two squash+FF onto `main` raced. Fly Deploy #60 (#112) succeeded, then a retried Deploy #59 (#111) overwrote it with the older SHA.

Every workflow that deploys on push to `main` must serialize, and **latest `main` wins**:

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: true
```

Do not use a per-SHA or per-PR group on the `main` deploy path (`github.sha` makes every push its own lock). `cancel-in-progress: false` lets an older in-flight deploy finish after a newer one.

Linear history: the newer squash already contains the older one, so cancelling the older deploy is correct.

Optional extra on Fly (or any retry-prone CLI): skip if `HEAD` is no longer `origin/main` after fetch, so a zombie retry cannot roll back.

Install: add that `concurrency` block to the repo’s deploy-on-main workflow. Recipe pointer: `repo-setup/install-agents-md.md`. Bots open a PR; do not merge unless Jonathan says so.
