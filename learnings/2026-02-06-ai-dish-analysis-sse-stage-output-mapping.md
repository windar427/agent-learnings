# AI Dish Analysis SSE 阶段输出映射与交互稳定性

Date: 2026-02-06

## Project Context
- CalorWise iOS（TakePhoto / AI Dish Analysis 流式分析页）
- 目标：让“阶段卡片 + 流式输出 + 食材 icon”与后端 SSE 事件 1:1 对应，并避免阶段跳闪/未展示就跳转。

## Problem
1. **阶段 icon 不转**：卡片/Loading 的旋转动画在复用或多次进出页面后会丢失（`CABasicAnimation` 不一定一直存在）。
2. **最终阶段未显示就跳转**：SSE 结束后立即 push 下一页，导致 “Finalizing/最后一步” 没有稳定展示。
3. **阶段输出未对齐**：需要把不同阶段对应的输出拆开：
   - Initiating AI analysis → 当前的流式文字输出
   - Database Accessed → `search_fdc_id` 的输入（仅展示 `querys` JSON）+ `get_food_icons` 的图片输出
   - Goal Integration → 目标/热量计算阶段（例如 `calculate_calories_by_fdc_id`）

## What Worked
### 1) 用 SSE `step` + tool 事件驱动阶段（不要用“文本内容猜测阶段”）
- 只要 SSE payload 里出现 `step`，就以它为准（同时兼容 `Int`/`String`）。
- 对 tool 相关 event 解析 `tool_name`/`name`、`input`/`arguments`、`output`/`result`：
  - `search_fdc_id`：抓取 `querys`/`queries` 数组，生成 pretty JSON（只展示该 JSON）
  - `get_food_icons`：解析 icon URL，触发 UI 展示
  - `calculate_calories_by_fdc_id`：推进到 Goal Integration
- 其他类型（prompt/log 等）默认忽略，避免把系统提示/调试信息泄漏到 UI。

### 2) 阶段切换做“队列 + 最小展示时长”
- 后端可能从 step 0 直接跳 step 3；UI 需要按 0→1→2→3 顺序补齐。
- 用 `enqueuePhaseTransition(to:)` 把缺失的阶段补入队列。
- 用 `sseMinPhaseDisplayDuration`（例如 0.6s）限制每个阶段最少停留时间，避免闪跳。

### 3) “Finalizing 稳定展示 + 图标完成/超时兜底”后再 push
- push 触发条件：
  - 当前阶段为 `.finalizing`
  - 食材 icon 请求已完成（或超时允许无 icon 跳转）
  - 且 Finalizing 已展示满最小停留时间
- icon 获取是异步的：需要 completion 或标记位配合导航调度。

### 4) 动画丢失的修复策略
- `UICollectionViewCell` 复用时：在 `setCell(...)` 里重新 `startContinuousRotation()`。
- 入口处兜底：如果 `layer.animation(forKey:) == nil`，重新启动旋转。
- 对 loading 子视图不要用局部变量，改为 view 属性，确保可以在需要时重启动画。

### 5) 取消/退出要清理干净（防止旧回调污染新会话）
- `stopSSE()` 时清理：task、buffer、timer、completion handler、pending artifacts。
- 用 `sseSessionID`（自增）让 `DispatchQueue.main.asyncAfter` 的旧回调自动失效。

## Implementation Checklist (Reusable)
- [ ] SSE payload 解析：优先 `step`，并兼容 tool 事件
- [ ] 阶段切换：队列化 + 最小展示时长
- [ ] 文本渲染：分阶段拼接（tool JSON + 流式正文），避免混淆
- [ ] icon 渲染：结果到达即展示（且与 Database Accessed 阶段绑定）
- [ ] 导航：Finalizing 展示稳定 + icon 完成/超时兜底
- [ ] stop/cancel：清理定时器与 pending work，避免跨会话串台
