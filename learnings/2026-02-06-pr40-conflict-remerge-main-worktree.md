# PR40 Conflict Re-Merge Main in Worktree

Date: 2026-02-06

## Project Context
- Repository: `FroskaAI/CalorWise-iso`
- PR: `#40`
- Branch: `wt/fix-figma-photo-tabbar-align`
- Scenario: PR had conflicts again after `main` advanced with new merge/revert commits.

## Learning
When a PR branch becomes conflicted again after `main` moves, resolve it directly in the feature worktree branch by re-merging latest `origin/main`, not by editing from the primary repo root.

## Reusable Workflow
1. Fetch and prune:
   - `git fetch origin --prune`
2. Work in the correct worktree branch:
   - `git -C <worktree> merge origin/main`
3. Resolve all conflict files and remove conflict markers.
4. Verify compile before finishing merge commit:
   - `xcodebuildmcp build_sim`
5. Commit merge and push branch:
   - `git commit`
   - `git push -u origin <branch>`
6. Sanity-check branch state:
   - `git status`
   - `git merge-base --is-ancestor origin/main HEAD`

## Why This Matters
- Avoids drifting fixes or accidental edits in the wrong directory.
- Keeps PR mergeable with current `main`.
- Catches runtime/compile regressions early by enforcing simulator build verification.

## Pitfalls to Avoid
- Assuming a previously resolved conflict stays resolved after `main` changes.
- Skipping compile validation after manual conflict resolution.
- Forgetting to push the updated feature branch after local merge commit.
