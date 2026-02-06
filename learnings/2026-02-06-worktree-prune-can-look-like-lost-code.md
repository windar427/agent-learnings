# Worktree Prune Can Look Like Lost Code

Date: 2026-02-06

Project: CalorWise iOS (git worktree workflow)

## Context

- A feature branch was merged to `origin/main` and the remote branch got deleted.
- A local `git worktree` directory for that branch was removed/pruned.
- Opening the deleted worktree path (or a stale folder) made it look like the code "went back" or recent changes disappeared.

## Diagnose Quickly

- Confirm the truth in `origin/main`:
  - `git log -1 --oneline origin/main`
  - `git show -s --format='%H %d %s' HEAD`
- List local worktrees and their paths:
  - `git worktree list --porcelain`

## Fix

- Recreate the worktree from `main` (or a known good commit):

```bash
git worktree add -b wt/<task> ../CalorWise-wt-<task> main
```

## Notes

- Remote branch deletion after merge is normal. The source of truth is the commit history on `origin/main`.

## Build Verification Tip (xcodebuildmcp 60s Timeout)

If `xcodebuildmcp build_sim` times out waiting for the tool call:

1. Prewarm once with `xcodebuild` using a stable `-derivedDataPath`.
2. Rerun `xcodebuildmcp build_sim` (incremental builds usually finish within the timeout).

## Layout Robustness (New Devices / iOS)

- Avoid hard-depending on global `kStatusBarH` / `kBottomSafeH` for critical layout.
- Prefer constraints relative to `view.safeAreaLayoutGuide` and/or existing header views (e.g. `backBtn.bottom`) so spacing stays correct across devices.
