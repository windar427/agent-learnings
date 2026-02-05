# CalorWise：透明悬浮 TabBar 的“内容穿透”根因修复（用 Scroll Insets）

**Date:** 2026-02-05  
**Project:** CalorWise (iOS, UIKit)  
**Tags:** UIKit, TabBar, ScrollView, Figma, LayeredPage

---

## Context

CalorWise 使用自定义“悬浮玻璃 TabBar”。产品要求：

- TabBar 永远在最上层（视觉与点击层级）。
- TabBar 材质半透明/玻璃效果时，页面内容应该能在其下方继续滚动并从 TabBar 下“透出来”。
- 这是全局共性问题，不能靠单页打补丁（例如抬高 content 容器）来“看起来不遮挡”。

## Root Cause

常见的错误做法是把页面主内容（ContentCard/ScrollContainer）整体 **向上抬高**（比如通过 bottom constraint 加上 `overlayHeight`），这样会导致：

- 内容在 TabBar 上方被“截断”，无法实现“透明材质下的内容穿透”。
- 最后一屏内容的可达性依赖于页面额外 padding，容易在不同页面/机型上漏掉。

## Fix Pattern（推荐）

目标是模拟系统 `UITabBar.isTranslucent = true` 的行为：**TabBar 覆盖在内容之上，但内容可以在其下方滚动。**

1) **保持 TabBar 永远最上层**
- 在 `viewDidLayoutSubviews` 或每次 layout 后，对 TabBar 容器做 `bringSubviewToFront`（以及必要的浮动按钮/日历容器等）。

2) **允许页面内容延伸到屏幕底部**
- 不要通过约束把内容容器整体顶上去。

3) **对“页面内部的 ScrollView”加 bottom inset（关键）**
- 给嵌入的 `UIScrollView/UITableView/UICollectionView` 增加：
  - `contentInset.bottom += tabBarOverlayHeight`
  - `scrollIndicatorInsets.bottom += tabBarOverlayHeight`
- 需要保存/叠加 base inset（避免重复叠加，或被外部代码覆盖）。

4) **统一入口处理，避免逐页补丁**
- 在通用页面基类（例如 `LayeredPageViewController`）中，对“被 embed 的滚动视图”集中设置 inset。

## Implementation Notes（落地要点）

- 如果页面使用 “ContentCard + 内部滚动” 的结构，优先在 embed 发生时记录 scrollView 的原始 inset（base），后续只在 base 之上叠加 overlay。
- 如果某些页面需要额外的底部间距（例如卡片与 TabBar 之间留 16pt 浮动感），应优先通过 **scrollView 的 inset/内容末尾 padding** 实现，而不是抬高整个 content 容器。

## Related Fixes（同一轮改动中验证过的关联问题）

### Week calendar 翻周/翻月 param error
- 选择日期被推进到未来（例如 nextWeek 直接落在未来周六）会导致后端参数错误。
- 修复要点：
  - 翻周时把 candidate 日期 clamp 到“本周最后一个非未来日期”。
  - 发请求前过滤未来日期（未来不请求）。

### 运动消耗（Burned）不显示
- 首页 burned 不应依赖 `DailyLog.exercise` 的字段推导；应使用运动日详情接口返回的 `total_kcal`（或同等权威字段）。

### 运动记录页 confirm 交互
- 合理交互：先选择 duration，之后再出现 confirm。
- 关键点：
  - confirm 的显示条件应绑定 `selDuration != nil`。
  - intensity 选项拉取完成后可以默认选中第一个，避免“必须点 intensity 才能继续”的反直觉路径。

### Workout Checkout 图表（网格/语言）
- 网格线样式要按 Figma（例如只保留横向虚线网格）。
- 周期标签与日期格式按设计稿语言（英文 weekday、必要时 `Thu` -> `Thur`）。

## Ops / Workflow Reminders

- 仓库若强制要求：`git worktree` → 修改 → `xcodebuildmcp build_sim` → `git commit` → `git push`。
- `./scripts/reflect` 若交互式，可用管道一次性喂入多行回答以自动化：

```bash
printf "%s\n" "context..." "summary..." "prefs..." "what worked..." "follow-ups..." | ./scripts/reflect --task "..." --outcome success
```
