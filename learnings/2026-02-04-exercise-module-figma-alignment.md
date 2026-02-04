# Exercise Module Figma Alignment — Implementation Patterns

**Date:** 2026-02-04
**Project:** CalorWise (iOS, Swift/UIKit)

## Context

Aligned the Exercise module (7 files) with Figma designs covering 11 page states. Key patterns and decisions documented below for future reference.

## UITableView A-Z Section Index

When adding an alphabetical sidebar index to a UITableView:

- Use native `sectionIndexTitles(for:)` and `sectionForSectionIndexTitle(_:at:)` — no custom view needed
- Group data in a `[String: [Model]]` dictionary keyed by uppercase first letter, with `#` as fallback for non-alpha
- Sort keys alphabetically, pushing `#` to the end
- Style with `tableView.sectionIndexColor` (green `kbase4BC473`) and `tableView.sectionIndexBackgroundColor = .clear`
- Use a computed property like `isBrowseAllMode` to conditionally return sections vs flat list, keeping both modes in the same data source
- Call `tableView.flashScrollIndicators()` in `viewDidAppear` to hint scrollability

## Dynamic Chart Y-Axis Ticks

Instead of hardcoding Y-axis values like `[0, 200, 400, ..., 2000]`:

- Calculate from data max using a predefined interval list: `[50, 100, 200, 250, 500, 1000]`
- Pick the first interval that yields 2-4 tick marks
- Always include 0 as the first tick
- This matches Figma's sparse Y-axis layout (e.g., 0, 500, 700 for typical exercise kcal data)

## X-Axis Date Formatting by Time Range

- 7-day mode: Use `"EEE"` format → Mon, Tue, Wed...
- 15/30-day mode: Use `"MMM-d"` format → Dec-5, Jan-12...
- Read the current `selDaysView.selDays` value to determine which format to apply

## Cell Selection State Pattern

For toggle-style selection (e.g., intensity level cells):

- Add `isItemSelected: Bool` property with `didSet` on the **cell** to swap images
- Set the property in `cellForRowAt` by comparing model IDs: `cell.isItemSelected = (model?.id == selectedModel?.id)`
- Don't rely on UITableView's built-in selection highlight — use explicit image swapping for custom radio-button UX

## Figma-to-iOS Search Bar Style

- Figma background `#F8F5FF` (light purple) replaces white + border approach
- Remove `borderWidth`/`borderColor` when Figma uses background color differentiation instead
- Corner radius values matter: 25 → 30 per Figma spec

## Typo Fix Strategy

- When fixing display text typos (e.g., "Excecise" → "Exercise"), only change user-visible strings (enum `rawValue`, button titles)
- Keep internal variable names and enum case names unchanged to avoid cascading refactor across multiple files
- Keep image asset names unchanged (`addPage-excecise`) since renaming assets requires Xcode project file changes

## gh CLI vs GitHub MCP for Private Repos

- GitHub MCP token (`windar427`) may not have access to organization repos (e.g., `FroskaAI`)
- For private org repos, `gh` CLI with proper SSH key auth is more reliable
- Fallback: provide the direct GitHub PR creation URL for manual browser-based creation
