# Native iOS (SwiftUI and UIKit)

Apple ships first-party adaptive APIs. Prefer configuration and layout review over a rewrite.

Sources: [Preparing your app for iPhone Duo](https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo), [Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), Tech Talks 111461–111466.

## Audit greps

```bash
grep -rn "UIScreen.main" --include="*.swift" .
grep -rn "userInterfaceIdiom" --include="*.swift" .
grep -rn "UIInterfaceOrientation\|isLandscape\|supportedInterfaceOrientations" --include="*.swift" .
grep -rn "\.frame(width:" --include="*.swift" .
grep -rn "widthAnchor.constraint(equalToConstant:" --include="*.swift" .
grep -rn "safeAreaInsets.left \* 2\|insets.left \* 2" --include="*.swift" .
grep -rn "UIRequiresFullScreen" --include="*.plist" --include="*.swift" .
grep -rn "ignoresSafeArea" --include="*.swift" .
```

Ignore genuine component constants (44-pt controls). Flag anything that stands in for screen width.

## Common layout rules (both frameworks)

From Apple’s engineering guide:

- Prefer system containers: split views, tab bars, arrangement views, navigation stacks. They adjust for outer, open inner, folded inner, and camera occlusions.
- Size views relative to their **container**, not fixed iPhone dimensions.
- Use **scene or containing view bounds**, not screen dimensions.
- UIKit: Auto Layout + automatic trait tracking of `horizontalSizeClass` / `verticalSizeClass`. Do **not** use `userInterfaceIdiom` or `UIInterfaceOrientation` for layout.
- Vertical bars appear on the **outer display when closed**, and on some **leading/trailing** inner views when open. Inspectors stay horizontal. In a multi-column split view, sidebar/content bars are horizontal; **detail** bars are vertical.
- Sheets: outer display bars vertical by default (opt out with `toolbarVerticalBehavior` / `preferredVerticalBarBehavior`). Inner sheets: horizontal for centered/leading placement, vertical for trailing. Set `presentationPlacement` / `preferredPlacement`.
- Custom views: `toolbarVerticalEdge` (SwiftUI) or `verticalBarEdge` (UIKit) if you need to know the vertical bar edge.
- Hero/background under a vertical bar: `backgroundExtensionEffect()` / `UIBackgroundExtensionView`.
- Bar item order: Back/Close at top, then Done-like actions. Semantic placements; `topBarPinnedTrailing` / `pinnedTrailingGroup` for prominent trailing; `cancellationAction` / `leadingItemGroups` for custom Back/Close.
- Vertical inclusion: `axisBehavior`. Overflow order: `visibilityPriority`. Force overflow: `ToolbarOverflowMenu` / `additionalOverflowItems`.
- Item representations: vertical → icon; horizontal → icon or title (prefers icon); overflow → icon and title. Title-only or custom-view items **do not** present vertically.
- Arrangement: split (HStack/VStack-like) or overlay (ZStack-like). Do not nest inside NavigationSplitView, list, or scroll view.

## SwiftUI

What already adapts on the iOS 27.1 SDK: `NavigationSplitView`, `TabView`, sheets, popovers, context menus, alerts. Columns collapse closed and tile/overlay open. Standard toolbar items move to a side column on the closed outer display and inner landscape; inner portrait stays horizontal.

### Size classes, not orientation

```swift
@Environment(\.horizontalSizeClass) private var horizontalSizeClass

var body: some View {
    if horizontalSizeClass == .regular {
        WideLayout()
    } else {
        NarrowLayout()
    }
}
```

Inner: regular × regular either orientation. Outer portrait: compact × regular. Outer landscape: compact × compact.

### Frames

```swift
// Wrong — 402-pt iPhone constant
content.frame(width: 390)

// Right
content
    .frame(maxWidth: .infinity)
    .frame(maxWidth: 700)
```

Need live size:

```swift
GeometryReader { proxy in
    GalleryView(columns: proxy.size.width > 600 ? 3 : 1)
}
```

### Safe areas

SwiftUI respects the safe area by default. Failure mode: `.ignoresSafeArea()` plus symmetric manual padding.

```swift
GeometryReader { proxy in
    let insets = proxy.safeAreaInsets // leading and trailing differ
    ContentView()
        .padding(.leading, insets.leading)
        .padding(.trailing, insets.trailing)
}
```

Foreground stays in the safe area; background artwork may extend to the bounds (HIG / checklist).

### ArrangementView

```swift
NavigationStack {
    ArrangementView {
        PlayerView()
    } secondary: {
        UpNextView()
    }
    .arrangementViewStyle(.split) // or .overlay, or .split.axes(.horizontal)
}
```

Apple’s guide shows `.arrangementViewStyle(.split.axes(.horizontal))` when you want primary-only in vertical layout and side-by-side in horizontal.

### Reserved regions

```swift
GeometryReader { proxy in
    let divisions = proxy.reservedRegions(kind: .division)
    let occlusions = proxy.reservedRegions(kind: .occlusion)
    // Optional: options include inactive regions for planning
}
```

Division (hinge) is active only while partially folded; zero width when flat.

### Tab bar sidebar

```swift
TabView { /* … */ }
    .defaultTabBarPlacement(.sidebar)
```

(From Tech Talks / iphoneduosupport.com checklist.) Same destinations in the same order across placements.

### Idiom and UIScreen

Do not gate layout on `userInterfaceIdiom == .pad`. Do not use `UIScreen.main`. Use traits and the window scene:

```swift
let scale = traitCollection.displayScale
let screen = view.window?.windowScene?.screen // only if you truly need a UIScreen
```

## UIKit

Standard `UINavigationController`, `UITabBarController`, and `UISplitViewController` adapt. **Custom `UIToolbar` content is not considered for vertical bar placement** — migrate to navigation-controller toolbar items.

### Traits, not idiom or orientation

```swift
switch traitCollection.horizontalSizeClass {
case .regular: applyWideLayout()
default: applyNarrowLayout()
}
```

Use automatic trait tracking rather than manually registering where the API supports it.

### Constraints and insets

```swift
// Wrong
view.widthAnchor.constraint(equalToConstant: 390).isActive = true
let width = view.bounds.width - view.safeAreaInsets.left * 2

// Right
NSLayoutConstraint.activate([
    content.leadingAnchor.constraint(equalTo: view.layoutMarginsGuide.leadingAnchor),
    content.trailingAnchor.constraint(equalTo: view.layoutMarginsGuide.trailingAnchor),
    content.widthAnchor.constraint(lessThanOrEqualToConstant: 700),
])
let width = view.bounds.inset(by: view.safeAreaInsets).width
foreground.frame = view.bounds.inset(by: view.safeAreaInsets)
```

### Arrangement

```swift
let arrangementVC = UIArrangementViewController()
arrangementVC.setViewController(PrimaryViewController(), for: .primary)
arrangementVC.setViewController(SecondaryViewController(), for: .secondary)
arrangementVC.updateArrangement(.split.axes(.horizontal))
```

Put a navigation controller **around** the arrangement, not inside it.

### Reserved regions

```swift
let regions = view.reservedRegions(kind: .division)
let frames = regions.map(\.frame)
```

### Vertical bars (UIKit)

From Apple’s guide + Tech Talk checklists:

- `item.axisBehavior`
- `traitCollection.verticalBarEdge`
- `preferredVerticalBarBehavior` (opt out / prefer)
- `navigationItem.pinnedTrailingGroup`, `leadingItemGroups`, `additionalOverflowItems`
- `visibilityPriority`
- Tab sidebar (iphoneduosupport.com / Tech Talks): `tabBarController.sidebar.preferredPlacement = .sidebar`

### Lifecycle

A visible Split View app can resign active. Do not pause UI solely on `sceneWillResignActive` if the scene is still on screen.

### UIRequiresFullScreen

Honored, but the app still resizes and **scales** on the inner display, including Split View. Not an exemption.

## Camera (native only)

If the app uses AVKit/AVFoundation:

- Capture can come from outer front, inner front, and rear cameras.
- Opening/closing/rotating can change which display the app is on and which way the camera points. Use Apple’s direction coordinator: [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces).
- Fully open + rear capture can also show content on the outer display: [Registering a camera capture accessory on iPhone Duo](https://developer.apple.com/documentation/avfoundation/registering-a-camera-capture-accessory-on-iphone-duo).

## Xcode help

Xcode 27.1 App Resizability modernization skill can do a first mechanical pass. Review its output; do not accept it blindly.

## Multiple scenes

iPhone Duo is the first iPhone that can show multiple instances. Apps that already support multiple scenes on iPad get this. New windows open on the inner display; handle request errors. Optional scene accessories put supplementary UI on the outer display (teleprompter, subject preview) — camera/accessory docs, not a layout substitute.
