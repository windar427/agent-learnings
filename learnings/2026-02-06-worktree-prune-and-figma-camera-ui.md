# Worktree Cleanup + Figma Camera Placeholder Gotcha

Date: 2026-02-06

Project context: CalorWise iOS app (git worktree workflow, Figma-driven UI alignment)

## Worktree cleanup behavior
- `git worktree remove` (or a prune script) deletes the worktree directory, but merged commits remain in `main`, and pushed branches/commits remain on the remote.
- “Updated code disappeared” is often either:
  - the worktree directory was removed after the branch was merged, or
  - a revert PR landed on `main` (so changes are still in history but no longer in the working tree).
- Quick recovery checklist:
  - verify `main` vs remote: `git fetch --prune` + `git log --oneline --decorate -n 30`
  - confirm whether a branch is merged: `git merge-base --is-ancestor <branch> origin/HEAD`
  - recreate a deleted worktree when needed: `git worktree add ../<path> <branch-or-commit>`

## Figma alignment workflow for large nodes
- `get_design_context` for a full screen can be huge/truncated; use `get_metadata` to locate child node IDs (search bar, bottom bar, tab bar), then call `get_design_context` on those smaller nodes to extract exact sizes/spacing.
- Use the node’s measured values (e.g. 343/345 widths, 58/50 heights, offsets) to set SnapKit constants precisely.

## Simulator camera placeholder overlap
- If the “camera unavailable” placeholder image already contains UI chrome (prompt text, focus frame, shutter, gallery icon), you must hide the real overlay controls (or remove the chrome from the placeholder asset) to avoid double-rendering.
