# iOS Floating Glass Tab Bar Implementation

**Date:** 2026-02-04  
**Project:** CalorWise (iOS, Swift/UIKit)  
**Tags:** UIKit, tab bar, glass effect, blur, Figma, iOS 26

---

## Context

When implementing a floating frosted glass tab bar (iOS 26 style) in the CalorWise project, several techniques are required to achieve the correct visual result matching Figma specs.

## Key Learnings

### 1. Blur Effect — Use the Right Material

Use `UIBlurEffect(style: .systemUltraThinMaterial)` instead of `.light` with colored overlays. **Never** add an opaque `backgroundColor` to `UIVisualEffectView` — it defeats the glass transparency entirely.

### 2. Glass Border for Definition

Add a subtle white border to give the floating bar edge definition:

```swift
layer.borderWidth = 0.5
layer.borderColor = UIColor.white.withAlphaComponent(0.7).cgColor
```

### 3. Inner Shadow for Depth (CALayer Trick)

To replicate Figma's inset shadow, use a CALayer workaround:

- Create an outer bezier path larger than the view's bounds.
- Append the reversed inner path (the bar's rounded rect).
- Set the combined path as `shadowPath` on a layer with `masksToBounds = true`.

This produces the inset shadow effect that standard `CALayer` shadow properties cannot achieve directly.

### 4. Content-to-Bar Gap (Floating Effect)

Add extra offset (e.g., +16pt) to `contentBottomConstraint` beyond `kCWTabBarOverlayHeight` so the content card doesn't touch the tab bar. This reveals the gradient background through the glass, reinforcing the floating effect.

### 5. Scroll View Bottom Padding

All scroll views in tab pages must have bottom padding (`kCWScrollBottomPadding`). For example, `BrowseViewController` originally had 0pt which caused content cutoff at the bottom — fixed with:

```swift
scrollView.updateContentHeight(0, minimum: kCWScrollBottomPadding)
```

### 6. Figma Reference Spec

The Figma node "Focus 为悬浮导航栏" specifies:

| Property | Value |
|---|---|
| Backdrop blur | 40px |
| Background | `rgba(255, 255, 255, 0.01)` |
| Border | `rgba(255, 255, 255, 0.7)` |
| Inner shadow | `inset 6px 6px 24px rgba(0, 0, 0, 0.16)` |

## Summary

The glass tab bar effect requires careful layering: a near-transparent blur view, subtle white border, inner shadow via path inversion, and proper spacing so background content shows through the glass. Scroll views on all tab pages need matching bottom padding to avoid content being hidden behind the floating bar.
