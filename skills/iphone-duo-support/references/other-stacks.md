# Other stacks (Unity, Capacitor/web, everything else)

Apple’s adaptive APIs are SwiftUI/UIKit only. For any other engine or wrapper: make layout **correct** at every size, then decide if a native plugin for reserved regions is worth it.

Third-party sections are ENGINEERING ANALYSIS from [iphoneduosupport.com](https://iphoneduosupport.com/) unless noted. Vendors had published no iPhone Duo guidance as of mid-September 2026.

## Universal path (any iOS binary)

1. Identify how the app is compiled for iOS (Xcode version / SDK). Apply the SDK tiers in SKILL.md.
2. Find every place size is cached at launch (`UIScreen`, `window.innerWidth` once, `Screen.width` in `Start()`, engine “design resolution”).
3. Find orientation locks and idiom/UA/device-type layout branches.
4. Find symmetric horizontal inset math.
5. Prefer the host’s **live** size (view bounds, CSS, engine safe-area rect) over a device table.
6. Width breakpoint `>= 600` logical points distinguishes outer 466 vs inner 626 when you lack size classes.
7. Split View: all apps participate; continuous resize; do not freeze a visible inactive window.
8. Keep important chrome off the **center** of the inner display (fold) until you can query division regions.
9. Games: HIG allows orientation lock but requires filling the screen as pose changes; prefer aspect change over letterbox.

## Unity

No first-party support. Resolution changes **mid-play** on fold, unfold, display move, and Split View.

```csharp
void Update() {
    if (Screen.width != _lastW || Screen.height != _lastH) {
        _lastW = Screen.width; _lastH = Screen.height;
        RebuildLayout(); // viewports, canvas scalers, render textures, input zones
    }
}
```

- Do not cache `Screen.width` / `Screen.height` in `Start()`.
- `Screen.safeArea` is the right API. Use the **rect**, not one horizontal margin applied to both edges.
- Orientation lock is allowed for games if the game still fills the screen.
- Folding can pause without true backgrounding. `OnApplicationPause` that stops time can look frozen — test fold during play.
- Joystick/tap zones in cached pixels go stale.
- HUD in the center of the inner display sits on the fold when partly open. Even column counts.
- `reservedRegions` needs a native plugin.

HIG for games: keep text/control sizes consistent; prefer changing aspect ratio over letterboxing; artwork in unavoidable bars.

## Capacitor, Cordova, Ionic, WKWebView

CSS was built for variable viewports. Most hybrid apps adapt if they treat the viewport as live and insets as per-edge.

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

Without `viewport-fit=cover`, `env(safe-area-inset-*)` is zero.

```css
/* Wrong — one side applied to both */
padding: 0 env(safe-area-inset-left);

/* Right */
padding-left: env(safe-area-inset-left);
padding-right: env(safe-area-inset-right);

@media (min-width: 600px) {
  .layout { grid-template-columns: 260px 1fr; }
}
```

- Do not branch layout on a phone user-agent (serves the phone layout to a 626-pt regular-width display). Use container/media queries.
- `100vh` misbehaves as the viewport changes; use `100dvh`.
- Cache of `window.innerWidth` is stale after fold/Split View; use `ResizeObserver` / `visualViewport`.
- HTML tab bars stay where CSS puts them; native bars move to the side.
- Safari has **no** Viewport Segments API for the fold (unlike Chrome on Android foldables). Web content cannot see the hinge without a native plugin.
- Capacitor status bar/keyboard plugins behave normally per that analysis.

## Kotlin Multiplatform / .NET MAUI / Xamarin / other

Same four assumptions. Map:

| Assumption | Typical bug | Fix |
| --- | --- | --- |
| Fixed width | Design-size constants, `UIScreen` | Bind to container / window size |
| Orientation lock | Info.plist / framework orientation API | Size classes or width |
| Symmetric insets | `horizontal inset * 2` | Per-edge insets |
| Phone idiom | `idiom == phone` → compact layout | Regular width on inner display |

If the toolkit embeds UIKit system bars, they may go vertical on 27.1. If it draws its own chrome, design that chrome for 466×678 and 626×890.

## What needs native code

Reserved regions, hinge angle, ArrangementView, scene accessories, and vertical-bar item APIs are UIKit/SwiftUI. Any non-native stack that must avoid the fold or drive a primary/secondary pose layout needs a plugin wrapping those APIs. Until then, keep critical controls off the inner-display center and use even columns.
