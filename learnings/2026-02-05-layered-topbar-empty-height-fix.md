# Layered topBarStackView 空内容时高度异常的修复方案

Date: 2026-02-05

## Project context

- iOS 工程采用 `LayeredPageViewController`（TopBar 固定 + ContentCard）
- 需求：按 Figma 对齐运动模块（Catalogue/Activity type），其中 Catalogue 卡片 top 位置固定，列表区域可滚动且默认显示滚动条

## Problem

在某些页面（例如 Browse/Catalogue）`topBarStackView` 没有任何 arranged subviews 时，约束只绑定了：

- `top = safeAreaTop`
- `left/right = super`

由于垂直方向缺少“高度/底部”约束，系统可能把 `UIStackView` 纵向拉伸以满足布局（尤其当整体垂直链条存在可被满足的空间时），导致：

- `topBarStackView.frame.maxY` 远超屏幕高度
- `contentCardView.top = topBarStackView.bottom + offset` 被推到屏幕底部（Catalogue 看起来“贴底”）

## Fix (recommended)

给 `topBarStackView` 增加“空内容时高度为 0”的保护，同时不影响有内容时的正常高度：

1) 设置 vertical hugging / compression resistance 为 `.required`，避免 TopBar 被扩展吸收多余高度。
2) 增加低优先级约束：`topBarStackView.bottom == safeAreaTop`（priority 250）。当 TopBar 有内容需要高度时，该约束会被打破；当无内容时，TopBar 高度稳定为 0。

这能稳定锚点（TopBar.bottom）的位置，从根源避免 ContentCard 被错误下推。

## UI implementation notes (Figma: fixed header + scrollable list)

当设计稿要求 “Header 固定，内容框内滚动” 时：

- 结构：`ContainerView`（圆角/背景）中放 `HeaderView` + `UITableView`
- 约束：`tableView.top = header.bottom`，`tableView.left/right/bottom = container`
- 默认滚动条：`tableView.showsVerticalScrollIndicator = true`，并在进入页调用 `flashScrollIndicators()` 作为默认提示

## Ops / workflow notes

若项目要求每次改动必须：

- 使用 `git worktree`
- `xcodebuildmcp build_sim` 验证编译
- commit + push

建议把 UI 对齐验证与编译验证放在 commit 之前，减少返工。

另外，`./scripts/reflect` 可能会交互式询问 Summary；在 CI/自动化或无交互场景下，可以用管道一次性喂入多行回答：

```bash
printf "%s\n" "context..." "summary..." "prefs..." "what worked..." "follow-ups..." | ./scripts/reflect --task "..." --outcome success
```

