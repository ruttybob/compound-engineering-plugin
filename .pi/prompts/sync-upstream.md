---
description: Sync upstream changes into the fork (fetch, rebase, push)
argument-hint: "[branch]"
---

Sync the fork with upstream.

## Remotes

- `origin` — personal fork (`ruttybob/compound-engineering-plugin`)
- `upstream` — source repo (`EveryInc/compound-engineering-plugin`)

## Steps

1. `git fetch upstream`
2. Checkout target branch (default: `$1` or `main`)
3. `git rebase upstream/<branch>` — resolve conflicts if any
4. `git push origin <branch>` — use `--force-with-lease` after rebase
5. Print short log of new commits: `git log --oneline ORIG_HEAD..HEAD`

## Rules

- NEVER push to `upstream`.
- If rebase has conflicts, stop and list conflicting files before proceeding.
- Abort rebase on unresolved doubts — do not guess resolutions.
