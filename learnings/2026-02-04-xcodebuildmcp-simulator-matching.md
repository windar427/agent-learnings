# xcodebuildmcp Simulator Matching: Use simulatorId Instead of simulatorName

## Date
2026-02-04

## Project
CalorWise (iOS, Swift/UIKit)

## Context
When using `xcodebuildmcp` MCP tool's `build_sim` to verify iOS builds, the `session-set-defaults` tool accepts both `simulatorName` and `simulatorId` for specifying the target simulator.

## Learning
When multiple iOS SDK versions are installed (e.g., iOS 18.3.1 and iOS 26.1), using `simulatorName: "iPhone 16"` with `OS: latest` can fail because `latest` resolves to the newest SDK (26.1), but "iPhone 16" may only exist under the older SDK (18.3.1). The build fails with "Unable to find a device matching the provided destination specifier."

### Solution
Always prefer `simulatorId` over `simulatorName` when setting xcodebuildmcp session defaults. Get the UUID from `xcodebuild -showdestinations` or `xcrun simctl list devices` first, then pass it directly:

```swift
// ❌ Fragile — OS:latest may not match the simulator name
session-set-defaults(simulatorName: "iPhone 16")

// ✅ Robust — UUID is unambiguous
session-set-defaults(simulatorId: "1E035448-2DB1-47EB-ACF0-618E7AA921B5")
```

### Additional Tip
Pre-warm `derivedData` with a direct `xcodebuild` command before running `xcodebuildmcp build_sim` to avoid the 60-second MCP timeout on cold builds.

## Tags
xcodebuildmcp, iOS, simulator, build-verification
