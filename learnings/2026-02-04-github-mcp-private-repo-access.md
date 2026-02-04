# GitHub MCP Token 无法访问私有仓库时的 PR 创建回退方案

- **Date**: 2026-02-04
- **Project**: CalorWise (FroskaAI/CalorWise-iso)

## 背景

在 CalorWise 项目中，通过 SSH key 可以成功 `git push` 到 `FroskaAI/CalorWise-iso` 私有仓库，但 GitHub MCP 工具（认证用户 `windar427`）调用 `create_pull_request` 时返回 404，因为 MCP token 没有该私有仓库的 API 访问权限。

## 经验

1. **SSH push 权限 ≠ GitHub API 权限**: SSH key 和 GitHub MCP token 是独立的认证机制。SSH key 绑定在仓库的 deploy key 或用户账号上，而 MCP 使用的是 Personal Access Token（PAT），两者的权限范围不同。

2. **诊断步骤**:
   - 先用 `mcp__github__get_me` 确认当前认证用户
   - 再用 `mcp__github__list_branches` 测试对目标仓库的 API 访问权限
   - 如果返回 404，说明 token 无权访问该仓库

3. **回退方案**: 当 MCP 和 `gh` CLI 都不可用时：
   - 提供 GitHub 的 PR 创建 URL: `https://github.com/{owner}/{repo}/pull/new/{branch}`
   - 附上建议的 PR title 和 body 内容，方便用户手动创建

4. **`gh` CLI 注意事项**: 该项目环境中 `gh` CLI 未安装（`/opt/homebrew/opt/gh` 存在但 binary 不在 PATH 中），需要用 MCP 工具或手动方式替代。

## 建议

- 如果需要频繁通过 Claude Code 创建 PR，应确保 GitHub MCP token 有 `repo` scope 并且对目标组织/仓库有访问权限
- 或者安装 `gh` CLI 并用 `gh auth login` 完成认证
