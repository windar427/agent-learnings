# CalorWise Review Environment Fallbacks (Build Command + Origin Remote)

Date: 2026-02-06

## Project Context
- Project: CalorWise (iOS)
- Situation: A review report claimed `xcodebuildmcp build_sim` was unavailable and `origin` remote was missing.

## What Happened
- The runtime environment may not provide a local `xcodebuildmcp` CLI binary, even when MCP tools are available in the agent runtime.
- Review environments can differ from development environments, causing false-negative operational checks.
- `xcodebuildmcp build_sim` can intermittently fail with `build.db locked` if concurrent builds run against the same DerivedData path.

## Reusable Fix Pattern
1. Add a build wrapper that is environment-tolerant:
- Prefer `xcodebuildmcp build_sim` when available.
- Fallback to `xcodebuild` simulator build when MCP CLI is unavailable.

2. Add a remote guard script:
- If `origin` exists, no-op.
- If `origin` is missing, recover from the only existing remote or from an explicit `ORIGIN_URL`.

3. Update docs so reviewers can run the same entrypoint:
- `./scripts/build_sim.sh`
- `./scripts/ensure_origin.sh`

## Commands / Artifacts
- Build wrapper: `scripts/build_sim.sh`
- Remote guard: `scripts/ensure_origin.sh`
- Verification:
  - `./scripts/ensure_origin.sh`
  - `./scripts/build_sim.sh`
  - `xcodebuildmcp build_sim` (rerun if `build.db locked` due to concurrency)

## Practical Guidance
- Treat `build.db locked` as a concurrency issue first, not a code regression.
- Re-run build serially before escalating.
- Prefer script-based entrypoints in PR/review instructions to reduce environment drift.
