---
name: iphone-duo-support
description: Migrate any app (iOS native, React Native, Flutter, Unity, Capacitor, or other) to iPhone Duo compatibility. Use when the user mentions iPhone Duo, foldable iPhone, iOS 27.1, ArrangementView, ReservedRegion, vertical bars, dual displays, or adapting layout for Apple's folding iPhone.
license: MIT
metadata:
  author: iphone-duo-support-skill
  version: "1.0"
  sources: "Apple HIG Designing for iPhone Duo; Apple Preparing your app for iPhone Duo; iphoneduosupport.com (Tech Talks 111461–111466)"
---

# iPhone Duo compatibility

Migrate the current app so it resizes correctly on iPhone Duo. Do not invent APIs, HIG rules, display sizes, or SDK behavior. If a fact is not in the sources below, say so and link the source you would need.

## Sources (authoritative)

1. [Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo) — HIG.
2. [Preparing your app for iPhone Duo](https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo) — Apple engineering guide.
3. [Get Ready for iPhone Duo](https://developer.apple.com/iphone-duo/) — Tech Talks 111461–111466.
4. [iPhone Duo Support](https://iphoneduosupport.com/) — practical checklist and cross-platform guidance sourced from those talks. Pages marked ENGINEERING ANALYSIS are inference about third-party frameworks (Meta, Google, Unity, web have published no iPhone Duo guidance). Preserve that distinction.

**Attribution:** Not affiliated with Apple. “iPhone” and “iPhone Duo” are trademarks of Apple Inc.

Load a reference only when the detected stack needs it:

- Native SwiftUI / UIKit → [references/native-ios.md](references/native-ios.md)
- React Native / Expo → [references/react-native.md](references/react-native.md)
- Flutter → [references/flutter.md](references/flutter.md)
- Unity, Capacitor, Cordova, web views, or unknown → [references/other-stacks.md](references/other-stacks.md)
- HIG design rules → [references/hig.md](references/hig.md)
- Source map → [references/sources.md](references/sources.md)

## Hardware facts (do not round)

Point sizes below are pixels ÷ 3 (`@3x`). Apple publishes pixels; points are derived.

| Display | Pixels | Points | Aspect | Notes |
| --- | --- | --- | --- | --- |
| Inner (unfolded) | 1878 × 2670 | 626 × 890 | 1.42 | 7.6", 430 ppi. Regular × regular size class in either orientation. Does **not** honor supported interface orientations. |
| Outer (closed) | 1398 × 2034 | 466 × 678 | 1.45 | 5.4", 460 ppi. Compact × regular (portrait), compact × compact (landscape). |
| iPhone 18 Pro (compare) | 1206 × 2622 | 402 × 874 | 2.17 | Previous “typical iPhone” shape. |

iPhone Duo reports the **phone** idiom with a **regular-width** inner display. iPad layouts gated on `.pad` / `Platform.isPad` never appear.

All apps participate in Split View multitasking. There is no opt-in. `UIRequiresFullScreen` is still honored but does **not** exempt the app: it still resizes and scales, including in Split View.

## SDK tiers

Apple’s written guide: build with the **latest Xcode** to use all available screen space. Apps built with **Xcode 26 and earlier** do not extend under the status bar and camera.

Tech Talk 111461 (via iphoneduosupport.com) describes three inner-display tiers:

| Built against | Inner display |
| --- | --- |
| Not the iOS 27 SDK | Familiar size and aspect ratio. Runs, does not use the display. |
| iOS 27 SDK | Extends left of the status bar area. |
| iOS 27.1 SDK | Reaches the screen edge. Standard bar buttons lay out vertically under the status bar. |

Rebuilding against 27.1 **without** the layout work can look worse than staying letterboxed. Treat the SDK bump as the start of the work.

Xcode 27.1 beta (18 September 2026) includes the iPhone Duo simulator and Device Hub pose controls. App Store submissions need the release Xcode.

## Workflow

### 1. Detect the stack

Inspect the repo. Pick one primary path:

| Signals | Path |
| --- | --- |
| `*.xcodeproj` / `Package.swift` + SwiftUI views | Native SwiftUI |
| UIKit view controllers, storyboards, `UIViewController` | Native UIKit |
| Mixed SwiftUI + UIKit | Native; apply both sections in `native-ios.md` |
| `react-native`, Expo, `app.json` / `app.config.*` | React Native |
| `pubspec.yaml`, `lib/`, Dart | Flutter |
| Unity `Assets/`, `*.unity` | Unity (other-stacks) |
| Capacitor, Cordova, Ionic, `WKWebView` | Web view (other-stacks) |
| Anything else shipping an iOS binary | General path in other-stacks |

State the stack and the SDK you found (`xcrun --sdk iphoneos --show-sdk-version`, Xcode version, EAS image, Flutter/Xcode selected) before editing.

### 2. Audit before rewriting

The four assumptions that cause most visible breakage (none need new APIs):

1. **“The screen is about 400 pt wide.”** Inner is 626. Fixed frames and launch-time cached widths break.
2. **“Portrait lock means portrait.”** Inner display ignores supported interface orientations.
3. **“Left and right insets match.”** Safe areas and layout margins are asymmetric. Never double one side.
4. **“Phone idiom means small screen.”** Idiom is `.phone`; width class on the inner display is regular.

Run the grep set for the stack (see each reference). Report hits with file:line. Do not “fix” 44-pt buttons or other component constants; only values that stand in for screen width/height.

### 3. Fix in this order

Do not skip ahead to ArrangementView / reserved regions until the baseline is clean.

1. **SDK** — build with Xcode 27.1 / iOS 27.1 SDK (or document why you cannot yet).
2. **Hard-coded sizes** — fill the container; cap line length with `maxWidth`, never pin to 390/402/428.
3. **Live geometry** — scene/view bounds, size classes, or the stack’s live dimension API. Never `UIScreen.main`.
4. **Asymmetric insets** — inset the rect, or pad leading and trailing separately.
5. **Size class / width, not idiom / orientation.**
6. **System navigation containers** so bars can go vertical.
7. **Toolbar overflow** — icon + title on every item; visibility priorities.
8. **ArrangementView / reserved regions** for primary+secondary or custom canvases.
9. **Camera coordinators** only if the app captures.
10. **Verify** every pose in Device Hub (matrix below).

### 4. HIG constraints while editing

Summarized from [Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo). Full notes: [references/hig.md](references/hig.md).

- Design for **iPhone**, not a new platform. Compact-width outer + regular-width inner cover every pose. Do not invent a layout per pose.
- Keep **functionality and state** the same across displays. Extra hierarchy on the inner display is allowed (Mail: list *or* message closed; both open).
- Use size classes, layout margins, and safe area insets. Avoid fixed widths and display-specific dependencies.
- Follow the system **vertical bars** (outer display, and inner landscape). Do not override default bar placement without a documented reason.
- **Asymmetry is expected.** Controls sit on one edge; Split View puts each app’s controls on its **outer** edge.
- Reserved regions: outer camera (always), inner camera (only when active), folding region (only when partially open). System alerts, sheets, menus, and split views adapt; custom views must query reserved regions.
- Adapt to the fold with **small moves**. Even column counts. Do not rearrange the whole screen.
- Arrangement views: split (side-by-side / stacked) or overlay (stacked, then one view per side of the fold). Keep navigation **outside** the arrangement.
- Games may lock portrait or landscape but must **fill the screen** as pose changes. Prefer changing aspect ratio over letterbox; if bars are unavoidable, fill them with artwork.
- Hinge angle is for **effects**, not layout.

### 5. Native APIs (iOS 27.1) — names only from Apple’s guide

Prefer system containers: split views, tab bars, arrangement views, navigation stacks.

**SwiftUI**

- `ArrangementView` + `.arrangementViewStyle(.split)` / `.overlay`; `.split.axes(.horizontal)` to limit axis.
- `GeometryProxy.reservedRegions(kind:options:layoutDirectionBehavior:)` with `ReservedRegion`. Kind: division (hinge) vs occlusion (camera). Fold division is inactive / zero width when flat; pass include-inactive when planning ahead.
- `@Environment(\.horizontalSizeClass)`, `toolbar`, `toolbarVerticalBehavior(_:)`, `toolbarVerticalEdge`, `axisBehavior(_:)`, `visibilityPriority(_:)`, `ToolbarOverflowMenu`, `ToolbarItemPlacement.topBarPinnedTrailing`, `.cancellationAction`, `presentationPlacement(_:)`, `backgroundExtensionEffect()`, `.defaultTabBarPlacement(.sidebar)`.

**UIKit**

- `UIArrangementViewController` + `setViewController(_:for:)` `.primary` / `.secondary`; `updateArrangement(.split.axes(.horizontal))`.
- `UIView.reservedRegions(kind:options:)` / `UIView.ReservedRegion`.
- Size classes via automatic trait tracking. Do **not** use `userInterfaceIdiom` or `UIInterfaceOrientation` for layout.
- `preferredVerticalBarBehavior`, `UITraitCollection.verticalBarEdge`, `UIBarButtonItem.axisBehavior`, `visibilityPriority`, `UINavigationItem.pinnedTrailingGroup`, `leadingItemGroups`, `additionalOverflowItems`, `UIBackgroundExtensionView`, `UISheetPresentationController.preferredPlacement`.

Do not place an arrangement inside a navigation split view, list, or scroll view.

Camera (only if relevant): `AVCaptureDeviceDirectionCoordinator`, `AVCaptureDeviceRotationCoordinator`, and Apple’s camera-capture-accessory APIs for outer-display preview while capturing on the rear camera.

### 6. Cross-platform rule

Apple ships adaptive APIs for SwiftUI and UIKit only. React Native, Flutter, Unity, and web stacks have **no first-party iPhone Duo support** as of the last verification dates on iphoneduosupport.com (mid-September 2026). For those stacks:

- Get layout **correct** with live size + asymmetric insets + width breakpoints (`>= 600` matches inner 626 vs outer 466).
- Do not claim hinge-awareness unless a native module/plugin wraps `reservedRegions`.
- Do not pause visible Split View content on `inactive` / `AppLifecycleState.inactive` / `OnApplicationPause` alone.

### 7. Verify

Device Hub pose matrix (Xcode 27.1 iPhone Duo simulator):

| State | Check |
| --- | --- |
| Closed, portrait | Compact-width outer layout |
| Closed, landscape | Toolbar overflow |
| Open flat, portrait | Regular width; sidebar if you adopted it |
| Open flat, landscape | Vertical bars |
| Partially folded | Nothing important on the division |
| Split View, drag divider | Continuous reflow |
| Second scene (if enabled) | Independent state |

Snapshot cheap checks: 626 × 890 pt (inner) and 466 × 678 pt (outer). Transitions matter more than static states.

Without Device Hub: iPad simulator Split View through the full divider range catches cached dimensions.

### 8. Stop conditions

- Do not add a database, auth, or unrelated refactor.
- Do not rewrite the app into SwiftUI “because Duo.”
- Do not lock orientation as a layout strategy (games: lock is allowed; filling the screen is required).
- Do not invent `displayFeatures` wiring on iOS; log it and treat empty as the default until proven on a real build.
- If the target SDK cannot yet be 27.1 (for example EAS still on Xcode 26), do the layout work first, then document the rebuild trap.

## Output to the user

1. Stack + current SDK tier.
2. Audit table (file:line, assumption broken, fix).
3. Code changes, framework-specific.
4. What was deferred (scenes, accessories, hinge module) and why.
5. How to test in Device Hub.
