# Sources

Do not invent APIs or HIG rules. If a claim is missing here, look it up or mark it unverified.

## Apple (primary)

| Document | URL |
| --- | --- |
| Designing for iPhone Duo (HIG) | https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo |
| Preparing your app for iPhone Duo | https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo |
| Get Ready for iPhone Duo | https://developer.apple.com/iphone-duo/ |
| Design for iPhone Duo | https://developer.apple.com/videos/play/tech-talks/111466/ |
| Prepare your app for iPhone Duo | https://developer.apple.com/videos/play/tech-talks/111461/ |
| Raise the bar with iPhone Duo | https://developer.apple.com/videos/play/tech-talks/111462/ |
| Strike a pose with adaptive layouts | https://developer.apple.com/videos/play/tech-talks/111463/ |
| Leverage multiple displays and scenes | https://developer.apple.com/videos/play/tech-talks/111464/ |
| Build a great camera experience | https://developer.apple.com/videos/play/tech-talks/111465/ |
| Choosing a camera by the direction it faces | https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces |
| Registering a camera capture accessory | https://developer.apple.com/documentation/avfoundation/registering-a-camera-capture-accessory-on-iphone-duo |
| Consumer specs | https://www.apple.com/iphone-duo/specs/ |

Apple JSON used while authoring this skill:

- `https://developer.apple.com/tutorials/data/documentation/technologyoverviews/preparing-your-app-for-iphone-duo.json`
- `https://developer.apple.com/tutorials/data/design/human-interface-guidelines/designing-for-iphone-duo.json`

## Practical checklist (secondary)

[https://iphoneduosupport.com/](https://iphoneduosupport.com/) — derived from the Tech Talks. Machine-readable: `/llms.txt`, `/llms-full.txt`, and `.md` on every path.

Pages marked **ENGINEERING ANALYSIS** (React Native, Flutter, Unity, Capacitor, Expo) mix Apple facts with inference because those vendors had not published iPhone Duo guidance as of the page dates (typically 2026-09-14/15). Preserve that distinction in user-facing output.

Point dimensions on that site are pixels ÷ 3. Apple publishes pixels, not points.

## API names taken from Apple’s written guide

SwiftUI: `ArrangementView`, `ReservedRegion`, `GeometryProxy.reservedRegions(kind:options:layoutDirectionBehavior:)`, `toolbarVerticalBehavior`, `toolbarVerticalEdge`, `axisBehavior`, `visibilityPriority`, `ToolbarOverflowMenu`, `topBarPinnedTrailing`, `cancellationAction`, `presentationPlacement`, `backgroundExtensionEffect`.

UIKit: `UIArrangementViewController`, `UIView.ReservedRegion`, `UIView.reservedRegions(kind:options:)`, `preferredVerticalBarBehavior`, `verticalBarEdge`, `UIBarButtonItem.axisBehavior`, `visibilityPriority`, `pinnedTrailingGroup`, `leadingItemGroups`, `additionalOverflowItems`, `UIBackgroundExtensionView`, `preferredPlacement`.

Do not use `userInterfaceIdiom` or `UIInterfaceOrientation` for layout (Apple).

## Not independently extracted from Apple JSON

These appear on iphoneduosupport.com citing Tech Talks. Use them, but do not embellish signatures:

- Three SDK inner-display tiers (Talk 111461)
- `.defaultTabBarPlacement(.sidebar)` / UIKit sidebar preferred placement
- `UIScreen.main` deprecation discussion (Talk 111461: avoid main-screen references)
- Hinge-angle APIs for effects, not layout (Talk 111464)
- Device Hub pose controls in Xcode 27.1
