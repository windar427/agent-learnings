# CalorWise Home Page Figma Pixel-Perfect Alignment

**Date:** 2026-02-04
**Project:** CalorWise iOS App (Swift + UIKit, SnapKit layout, CocoaPods)
**Task:** Align Home page UI to Figma design spec (Node `15897:15885`, File `nin1JX956wFtcKRsHcRMBV`)

---

## 8 Changes Made

### 1. NutritionalParamView Fold State
Reduced fold height from 268→94pt. Hide `tImgView`/`titleLabel`/`fiberView`/`sugarView`/`saltView` in fold. Added `proteinViewTopFold=16` for collapsed state. Changed `valueLabel` font from 15pt medium to 13pt regular.

### 2. Eaten/Burned Card Containerization
Added `eatenCardView` and `burnedCardView` (white bg, `cornerRadius=20`, 110×113) in `HomeCalView`. Labels moved inside cards. Cards positioned overlapping with `circleView` edges.

### 3. Week Calendar Selected Style
`selectBackView` color `#6591FC`→`#AC92F7` (purple), `cornerRadius` 20→30, `ratingDotView` 10→9pt, `dateLabel` font bold→medium, `selectBackView` width 40→41.

### 4. Date Picker Arrow Buttons
Added `backgroundColor=kBaseE1F0FE` and `cornerRadius=16` to `lastWeakBtn`/`nextWeekBtn`. `todayLabel` font bold→medium.

### 5. Meal Row Chevron
Wrapped `checkDetailImgView` in a `checkDetailContainerView` (`kBaseE1F0FE` bg, `cornerRadius=16`, 40×40). Inner icon sized to 12×12 centered. Updated `dashedLine` constraint to reference container.

### 6. Greeting Area
Unhid `settingBtn`. Adjusted `helloLabel` top 40→38, left `kSnp_left_30`→26. `settingBtn` right `-kSnp_left_30`→`-26`.

### 7. Section Spacing
`nutritionalView.top` relative to `ratingView.bottom`: `offset(15)`→`offset(0)`.

### 8. CircularCircleView Text
`"Kcal Left"`→`"Cal Left"`.

---

## Lessons Learned

- **Constraint reference updates:** When wrapping a `UIImageView` in a container view, update all SnapKit constraint references that previously pointed to the image view (e.g., `dashedLine.right` constraint).
- **Fold/expand animation:** For `NutritionalParamView` fold/expand, use `snp.updateConstraints` for the `proteinView` top offset rather than `remakeConstraints` since only one constraint value changes.
- **Reuse existing design tokens:** All needed colors (`kBaseAC92F7`, `kBaseE1F0FE`, etc.) were already defined in `ColorExtension.swift` — always check existing tokens before adding new ones.
- **git worktree workflow:** Always create a worktree before making changes; build in the worktree directory.
- **Build timeout prevention:** Pre-warming `derivedData` with `xcodebuild` generic build before running `xcodebuildmcp build_sim` avoids 60s timeout issues.

---

## Files Modified

- `Modules/Home/Views/NutritionalParamView.swift`
- `Modules/Home/Views/HeaderInfo/HomeCalView.swift`
- `Modules/Home/Views/Calendar/HomeWeekCalendarView.swift`
- `Modules/Home/Views/Calendar/FoodRecordView.swift`
- `Modules/Home/HomeViewController.swift`
- `Modules/Home/Views/HeaderInfo/CircularCircleView.swift`
