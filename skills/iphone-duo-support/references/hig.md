# Designing for iPhone Duo (HIG)

Source: [Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo) (Apple, new 9 September 2026).

An app designed for iPhone Duo adapts to both displays and stays continuous as the device opens and closes. You are still designing for **iPhone**. Standard iPhone patterns apply.

## Anatomy

- **Outer display:** used when closed. System places toolbars and tab bars on the **side** to preserve vertical space.
- **Inner display:** used when open. Controls stay on the side in **landscape** so height matches the closed experience. Portrait inner display keeps horizontal bars.
- **Hinge:** enables poses (book, tent, laptop, flat, standing on edges) and occupies space when folded.
- **Outer camera:** corner, always visible, vertically aligned with side controls.
- **Inner camera:** under the display; hidden until the camera is active.

Do not design a custom layout per pose. Compact width (outer) + regular width (inner) cover the fundamentals. Let the existing layout expand.

## Best practices

- **Build to resize.** Two displays, many poses, Split View. Use size classes, layout margins, and safe area insets. Avoid fixed widths and display-specific dependencies.
- **Consistent experience across displays.** Same functionality and element state. You may show an extra level of hierarchy on the inner display (Mail: list or message when closed; both when open).
- **Same functionality across poses.** Controls may overflow and content may move; people must still reach the same actions.
- **Follow vertical bars.** Outer display is wider and shorter than other iPhones; side controls preserve content height and match hardware asymmetry. Standard components get this automatically.
- **Games:** may lock portrait or landscape, but must fill the screen as pose changes. Keep text and control sizes consistent while resizing. Prefer changing aspect ratio over letterboxing/pillarboxing; if bars are unavoidable, put artwork in the padding.

## Reserved regions

Areas content avoids, or that components adapt around (similar to iPad window controls).

1. **Outer front-facing camera** — always present; expands into Dynamic Island for Live Activities. Side controls account for it.
2. **Inner front-facing camera** — only when the camera is active. UI moves aside when it appears.
3. **Folding region** — when partially open, divides the inner display into usable regions and excludes the center.

System alerts, context menus, and sheets move for the fold. Split views adjust column width and margins. Custom components use reserved-region APIs (see native-ios.md).

Adapt to the fold with small moves. Prefer even grid columns. Do not make extreme layout changes while folding.

## Split views and arrangements

- Split view: expands on the inner display, collapses to one pane on the outer — same as compact vs regular elsewhere. Standard split views adapt to reserved regions.
- **Arrangement view:** primary + secondary, reorganized by size, orientation, and reserved regions.
  - **Split:** divides the area. Horizontal when wider than tall; vertical when taller than wide.
  - **Overlay:** primary on top of secondary when no active division; when partially folded, one view per side of the fold.
- You can limit split axes and collapse the overlay secondary.
- Use an arrangement when the layout already looks like one (HStack/VStack → split; ZStack → overlay).
- **Keep navigation outside** arrangement views. Arrangements do not handle navigation.

## Vertical controls

On the outer display and inner landscape, toolbars, tab bars, and navigation move to the side. Inner portrait keeps horizontal bars.

Side column, top to bottom: Dynamic Island, status bar, toolbar (including navigation), tab bar.

Split View: each app’s controls sit on its **outer** edge (left app → left).

Vertical controls stay aligned with hardware (same side relative to the outer camera; same side in RTL).

- Account for **asymmetric** content space. Use safe areas, including the opposite edge in Split View.
- Keep control **relative positions** similar across poses.
- Top of the vertical axis: Back/Close, then prominent actions (Done). Keep original groupings; the system spaces top-bar vs bottom-bar items.
- Overflow is **bottom to top** by default. Set visibility priority on groups, then items. Keep frequent actions and badged status visible.
- Do not override default bar placement without a strong reason.
- Full-width layouts are OK for non-scrolling immersive UI that avoids the Dynamic Island and status bar (Calculator). Background/header may span full width while scrollable content stays inset.
- Group toolbar items with system grouping APIs; do not add fixed spacing.
- Keep controls near the content they affect (Mail list actions stay with the list, not the side column of the detail).
- Provide **title and symbol** for each non-text-only toolbar item. Text-only items stay in a horizontal bar. Custom views in items do not present vertically.
- When space is tight: navigation-focused UIs overflow the toolbar so tabs remain; task-oriented UIs minimize the tab bar to keep toolbar actions (default vs minimized-tab behavior).
- Use the **system overflow menu**. Reserve the ellipsis for overflow.

## What this page does not contain

Apple’s HIG page does not list point sizes, SDK tier tables, or cross-platform APIs. Those live in Preparing your app, Tech Talks, and iphoneduosupport.com.
