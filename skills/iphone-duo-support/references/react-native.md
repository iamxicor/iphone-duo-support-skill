# React Native and Expo on iPhone Duo

**Status:** Meta has shipped no iPhone Duo APIs. Apple-side facts below are from Apple; RN/Expo conclusions are from [iphoneduosupport.com](https://iphoneduosupport.com/frameworks/react-native/) (ENGINEERING ANALYSIS, last verified 2026-09-14/15). Do not claim first-party RN support.

Flexbox/Yoga reflows when UIKit resizes the root view. Most screens built from flex ratios survive. Cached dimensions, orientation locks, idiom checks, and symmetric inset math do not.

## Audit greps

```bash
grep -rn "Dimensions.get" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" .
grep -rn "Platform.isPad\|Device.deviceType\|userInterfaceIdiom" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" .
grep -rn "orientation.*portrait\|orientationLock\|lockToPortrait" .
grep -rn "paddingHorizontal\|Math.max(insets" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" .
grep -rn "AppState" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" .
```

Every `Dimensions.get` **outside a component render** is suspect. `Dimensions.addEventListener` does not rebuild a module-scope `StyleSheet.create`.

## Checklist

1. Build with **Xcode 27.1 / iOS 27.1 SDK**. Older binaries run letterboxed (SDK tiers in SKILL.md).
2. Replace module-scope `Dimensions.get('window')` with `useWindowDimensions()`.
3. Remove portrait locks used as layout. Inner display ignores supported orientations.
4. Pad `insets.left` and `insets.right` separately (`react-native-safe-area-context`). Do not `Math.max` them into one horizontal padding.
5. Branch on **live width**, not `Platform.isPad` (false on Duo). `width >= 600` → inner 626 (two columns) vs outer 466 (one column). Split View can go back below 600.
6. Pause media/timers on `AppState` `background`, not `inactive` (visible Split View can be inactive).
7. Prefer a **native** tab bar so iOS can move it to the side column. JS-drawn tabs stay at the bottom.
8. Test camera libraries across a fold (depends on the library adopting Apple’s coordinators).
9. Hinge / ArrangementView / vertical bar APIs: **no JS bindings**. Canvas/editor/game boards need a native module wrapping `reservedRegions`. Content apps can be correct without being hinge-aware.
10. Full-screen 16:9 letterboxes heavily on 1.42. Put related content under the player.

## Dimensions

```jsx
// Wrong — once at module load
const { width } = Dimensions.get('window');
const styles = StyleSheet.create({ card: { width: width - 32 } });

// Right
function Card({ children }) {
  const { width } = useWindowDimensions();
  return <View style={{ width: width - 32 }}>{children}</View>;
}
```

Width-driven layout:

```jsx
const { width } = useWindowDimensions();
const isWide = width >= 600;
return isWide ? <TwoColumn /> : <SingleColumn />;
```

`useWindowDimensions()` re-renders on unfold, fold, rotation, and Split View.

## Safe areas

`SafeAreaView` / `react-native-safe-area-context` apply edges separately. Hand-rolled math is the bug:

```jsx
const insets = useSafeAreaInsets();
// Wrong
<View style={{ paddingHorizontal: Math.max(insets.left, insets.right) }} />
// Right
<View style={{ paddingLeft: insets.left, paddingRight: insets.right }} />
```

RN’s own `SafeAreaView` is deprecated in Expo guidance; prefer `react-native-safe-area-context`.

## Navigation

On Duo, system toolbars/tab bars/nav move to the side on the closed outer display and inner landscape; inner portrait stays horizontal.

`createBottomTabNavigator` draws JS views — the system has nothing to move.

React Navigation 7 native bottom tabs (unstable; needs RN 0.79+ and `react-native-screens` 4.25+):

```jsx
import { createNativeBottomTabNavigator } from '@react-navigation/bottom-tabs/unstable';
const Tabs = createNativeBottomTabNavigator();
```

Expected to get `UITabBarController` side placement on iOS 27.1; **unverified on device** — test in the simulator before committing.

## Expo-specific

An Expo app is a React Native app plus three extra decisions:

1. **EAS image = SDK tier.** As of 15 September 2026, Expo’s published `latest` / `sdk-57` iOS images pointed at Xcode 26.6 (`macos-tahoe-26.5-xcode-26.6`). Cloud builds on that image stay in the “familiar size” tier. Confirm [Expo build infrastructure](https://docs.expo.dev/build-reference/infrastructure/) before assuming 27.1. Do layout work **before** the rebuild that opts into the new SDK.
2. **`app.json`:** change `"orientation": "portrait"` (Expo template default) to `"default"`. Leave `ios.requireFullScreen` off — Duo honors it by **scaling**, not exempting.
3. **Expo Router:** switch to `NativeTabs` so iOS can place the tab bar in the side column.
4. Do not choose layouts with `expo-device` `Device.deviceType` — Duo is a phone with regular width.
5. eas-cli **24.4.2+** before building on Xcode 27 (per that Expo article).

## What you cannot reach from JS

No JavaScript bindings for `ReservedRegion`, `ArrangementView`, hinge angle, scene accessories, or vertical toolbar `axisBehavior` / `visibilityPriority`. Bridging `reservedRegions(kind:)` is the native-module path.

Until then: keep critical controls off the **center** of the inner display (where the fold falls) and use even column counts.

## Testing without hardware

- iPad simulator Split View, drag the divider through its range.
- Snapshots at 466 × 678 pt (outer) and 626 × 890 pt (inner).
- Device Hub in Xcode 27.1 for fold, vertical bars, and camera-across-fold.
