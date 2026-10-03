# Flutter on iPhone Duo

**Status:** Partial. Flutter has `MediaQuery.displayFeatures` / `TwoPane` from Android foldables. **Whether the iOS embedder populates them for iPhone Duo is unconfirmed.** Google has published no iPhone Duo guidance. Apple-side facts are from Apple; Flutter conclusions are from [iphoneduosupport.com](https://iphoneduosupport.com/frameworks/flutter/) (ENGINEERING ANALYSIS, last verified 2026-09-15).

Assume `displayFeatures` is **empty** until a real iOS 27.1 build logs otherwise.

## Audit greps

```bash
grep -rn "setPreferredOrientations\|OrientationBuilder" --include="*.dart" .
grep -rn "EdgeInsets.symmetric" --include="*.dart" .
grep -rn "MediaQuery.of(context).size" --include="*.dart" .
grep -rn "AppLifecycleState.inactive" --include="*.dart" .
grep -rn "defaultTargetPlatform\|isPhone\|isTablet" --include="*.dart" .
```

## Checklist

1. `flutter build ios` uses the selected Xcode. Use **Xcode 27.1** to reach the screen edge.
2. Read size in `build` with `MediaQuery.sizeOf(context)` — never cache in `State` / `initState`.
3. Do not use `SystemChrome.setPreferredOrientations` as layout. Inner display ignores it. Do not use `OrientationBuilder` as the layout signal (it reports shape, not room). Branch on width.
4. `MediaQuery.paddingOf(context)` — use `left` and `right` separately. `EdgeInsets.symmetric(horizontal: padding.left)` and `padding.horizontal / 2` are wrong.
5. `width >= 600` (existing tablet breakpoint) fires on the 626-pt inner display. Drive it by size, not platform.
6. Pause on `AppLifecycleState.paused`, not `inactive` (visible Split View can be inactive).
7. Log `displayFeatures` on a Duo/simulator build.
8. Flutter-drawn `CupertinoTabBar` / `NavigationBar` stay where you put them; system bars move to the side. Consider `NavigationRail` when wide.
9. Test the `camera` plugin across a fold.
10. `ArrangementView` and vertical bar APIs have no Dart equivalent — platform channel if you need them.

## Size in build

```dart
// Wrong
late final double _width;
@override
void initState() {
  super.initState();
  _width = MediaQuery.of(context).size.width;
}

// Right
@override
Widget build(BuildContext context) {
  final width = MediaQuery.sizeOf(context).width;
  return width > 600 ? const TwoColumn() : const SingleColumn();
}
```

`sizeOf` rebuilds only when size changes. Use `LayoutBuilder` for a subtree’s constraints (Split View window ≠ display).

## Insets

```dart
final padding = MediaQuery.paddingOf(context);
// padding.left and padding.right differ on Duo
```

## displayFeatures (unverified on iOS)

```dart
final features = MediaQuery.of(context).displayFeatures;
debugPrint('displayFeatures on this device: $features');
final hinges = features.where((f) => f.type == DisplayFeatureType.hinge);
```

Conceptual map (only if populated): hinge → Apple **division** region; cutout → **occlusion**; `DisplayFeatureState` ≈ active/inactive. `TwoPane` then matches Android foldable patterns.

If empty: same options as React Native — platform channel around `reservedRegions(kind:)`, or fold-unaware but correctly resizing layout.

## Media

Inner aspect ~1.42. `AspectRatio(16 / 9)` filling the screen letterboxes heavily. Prefer a `Column` with a fixed-aspect player and related content beneath (the layout ArrangementView would give you natively).

## Tab bars and camera

Design Flutter chrome for a short wide outer display on purpose. Camera preview: start a session, fold and unfold, watch for black/frozen/wrong lens.

## Testing

- iPad simulator Split View through the divider range.
- Snapshots at 466 × 678 and 626 × 890 points.
- Device Hub for fold and side bars.
