# Codex 技能触发方式（$remember 而非 /remember）
Date: 2026-02-04

## Context
- Codex CLI 的 skill 是通过 `$skillName` 触发的（例如 `$remember ...`），不是通过 `/command`。
- `remember` skill 默认写入 `~/code/projects/agent-learnings/learnings/`。

## Learning
- 在 Codex 里输入 `/remember ...` 不会触发 skill（那是内置命令风格）；应使用 `$remember topic: 内容`。
- 如果本机没有 learnings 仓库，需要先准备好 `~/code/projects/agent-learnings`，并确保只写入现有 `learnings/` 目录。
- 保存后按仓库规范提交并推送到 `origin main`，便于后续通过 raw URL 直接访问。
