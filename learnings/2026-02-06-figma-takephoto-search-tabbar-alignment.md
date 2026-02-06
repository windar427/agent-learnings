# Figma 拍照页/搜索页/底部 Bar 对齐流程

Date: 2026-02-06

Project Context: CalorWise iOS（TakePhoto / SearchFood / RootTabBar）

## 背景
在一次集中对齐任务中，需要同时修复 3 个节点差异：
- 拍照页（15897:25757）
- 搜索页底部 Done 区域（15897:26044）
- 全局底部 Bar 顺序与样式（15930:19687）

## 关键经验
1. Figma 驱动改动必须先拿 `get_design_context` + `get_screenshot`，并且确保是目标 variant，再开始改约束。
2. 拍照页不是单层布局：顶部搜索相机条和下方相机容器是两套约束，常见“挤压/贴近”问题需要两层同时调整。
3. 搜索页底部 Done 区域要把 `safeArea` 计入高度并贴到底部容器，否则视觉上会“贴边不对齐”。
4. 底部 Tab 顺序不仅是图标顺序，还要同步 controller 映射与 selected 状态逻辑；抽 `setSelectedTab` 可以避免 icon 状态和 `selectedIndex` 失配。
5. `xcodebuildmcp build_sim` 超时时，先用 `xcodebuild ... -destination 'generic/platform=iOS Simulator' build` 预热，再跑 MCP 构建，成功率更高。

## 可复用检查清单
- 先 Figma：`design_context + screenshot`
- 再实现：局部组件约束 + 容器层级约束一起检查
- 再验证：`xcodebuildmcp build_sim`
- 最后提交：worktree 内 commit/push + 反思记录

## 风险与后续
- 视觉 1:1 仍建议在目标模拟器机型（如 iPhone 17 Pro）逐屏截图回归。
- Tab 语义调整后，需再走一次关键路径（Daily/Exercise/Recipe/Setting）确认入口行为与历史逻辑一致。
