# BPE SSE 分阶段动画实现经验

**Date:** 2026-02-04
**Project:** CalorWise (iOS)

## 概要

为食物识别的 SSE 流式输出环节实现了分阶段入场动画、SSE step 驱动的卡片切换、以及食物图标加载功能。

## 关键经验

### 1. SSE step 字段驱动卡片切换

SSE 事件 JSON 中包含 `step` 字段（Int 类型），可直接映射到分析阶段枚举：

- `step=0` → analyzing（蓝色卡片）
- `step=1` → databaseAccess（绿色卡片）
- `step=2` → goalIntegration（橙色卡片）
- `step=3` → finalizing（紫色卡片）

**双策略检测**：优先解析 `step` 字段，兜底使用文本关键词匹配（如 `"2. Ingredients"` / `"5. Portion"` / `"7. Plating"`）。阶段只能前进不能后退。

### 2. ExcerciseModuleView.switchToIndex 替代 Timer

删除 `BPECollectionCardView` 中的 `Timer` 自动滚动逻辑，改为外部 SSE 阶段驱动调用 `exerciseView.switchToIndex(index:)`。该方法在主线程异步执行滚动，需确保 collectionView 已加载且可见（alpha=1）后再触发。

### 3. 分阶段入场动画时序

对齐 Figma 9 帧设计的动画序列：

```
F1→F2 (0~0.3s):  bpeView 整体 alpha 0→1，约束从 kScreenH 滑到 0
F2→F3 (0.3~0.65s): backView 弹性滑入（spring damping=0.85）
F3→F4 (0.65~1.0s): 内容逐个渐显（齿轮→标题→虚线→卡片→Loading）
延迟 0.5s:        展开 SSE 文本区
```

退场动画分两步：backView 缩小到 0.95 + 半透明 → 整体下滑退出。非动画退场后需重置 `transform = .identity`。

### 4. 食物图标获取模式

复用 `SearchFoodModel.getFdcImgURL()` 的 API 调用模式：

```swift
let param: [String: Any] = ["token": token, "fdc_id_list": fdcIds]
NetworkManager.request(.getFoodIcons, parameter: param) { value, ... in
    // value.dictionary 以 fdc_id 为 key，包含 real_icon_url 和 cartoon_icon_url
}
```

优先显示 `real_icon_url`，降级到 `cartoon_icon_url`，使用 Kingfisher `kf.setImage(with:)` 加载。

### 5. xcodebuildmcp 超时问题

在 worktree 中首次构建时，xcodebuildmcp 的 `build_sim` 可能因编译时间过长而超时。解决方案：先用 `xcodebuild` CLI 直接构建预热 derivedData，再用 xcodebuildmcp 做增量验证。设置 session defaults 时需将 `workspacePath` 指向 worktree 中的 `.xcworkspace`。

### 6. backViewFullH 计算

插入 `foodIconsStackView`（44pt 高度 + 10pt 间距）后，`backViewFullH` 从 490 增至 544。布局链：cardCollectionView → foodIconsStackView → textView → loadingView。
