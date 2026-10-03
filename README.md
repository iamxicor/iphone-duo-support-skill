# iPhone Duo Support skill

An [Agent Skill](https://agentskills.io/specification) that walks Claude, Codex, and Cursor through migrating **any** iOS-bound app — SwiftUI, UIKit, React Native, Flutter, Unity, Capacitor/web, or other — to [iPhone Duo](https://developer.apple.com/iphone-duo/) compatibility.

The skill does not rewrite your app into a new stack. It audits the four layout assumptions that break on Duo, applies Apple’s iOS 27.1 APIs when you are on SwiftUI/UIKit, and gives a correct-but-honest path for frameworks that have no first-party Duo support yet.

Not affiliated with, endorsed by, or sponsored by Apple Inc. “iPhone” and “iPhone Duo” are trademarks of Apple Inc.

## What the skill covers

- Dual displays (inner 626 × 890 pt, outer 466 × 678 pt) and SDK tiers
- Size classes instead of orientation locks or `.pad` idiom gates
- Asymmetric safe areas; never doubling one inset
- Vertical navigation/tool/tab bars and overflow
- `ArrangementView` / `UIArrangementViewController` and reserved regions (fold + cameras)
- React Native / Expo, Flutter, Unity, Capacitor, and a general path for other engines
- Device Hub pose testing

Instructions are sourced from Apple’s [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), [Preparing your app for iPhone Duo](https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo), and the practical checklist at [iphoneduosupport.com](https://iphoneduosupport.com/). The skill tells the agent not to invent APIs.

## Install

The skill is the folder `skills/iphone-duo-support/` (a `SKILL.md` plus `references/`). Copy that folder — not just `SKILL.md` — so progressive disclosure still works. The folder name must stay `iphone-duo-support` to match the `name` field.

### Cursor

Project (shared with the repo):

```bash
mkdir -p .agents/skills
cp -R /path/to/this-repo/skills/iphone-duo-support .agents/skills/
```

Cursor also loads `.cursor/skills/`. User-wide:

```bash
mkdir -p ~/.cursor/skills
cp -R /path/to/this-repo/skills/iphone-duo-support ~/.cursor/skills/
```

Cursor additionally reads `.claude/skills/` and `.codex/skills/`, so one copy in `.agents/skills/` is enough for mixed teams.

Invoke with `/iphone-duo-support` or ask to migrate the app to iPhone Duo. Restart Agent if a newly copied skill does not appear under Customize → Skills.

Personal `~/.cursor/skills/` copies stay on your machine unless you turn on **Sync Skills for Cloud Agents**.

### Claude Code

Project:

```bash
mkdir -p .claude/skills
cp -R /path/to/this-repo/skills/iphone-duo-support .claude/skills/
```

User-wide:

```bash
mkdir -p ~/.claude/skills
cp -R /path/to/this-repo/skills/iphone-duo-support ~/.claude/skills/
```

Commit the project copy so teammates and Claude cloud sessions get it. Invoke with `/iphone-duo-support`. Claude Code watches the skills directory; a restart is usually unnecessary.

### Codex

Codex loads `.agents/skills` from the current directory up to the repo root, and `~/.agents/skills` for the user:

```bash
mkdir -p .agents/skills
cp -R /path/to/this-repo/skills/iphone-duo-support .agents/skills/
```

or

```bash
mkdir -p ~/.agents/skills
cp -R /path/to/this-repo/skills/iphone-duo-support ~/.agents/skills/
```

Invoke with `$iphone-duo-support` (or `/skills`). Codex picks up new skills automatically; restart if one is missing. This skill includes `agents/openai.yaml` for Codex/ChatGPT display metadata.

### One copy for all three

If the repo is used with Cursor, Claude, and Codex:

```bash
mkdir -p .agents/skills
cp -R skills/iphone-duo-support .agents/skills/
```

Cursor and Codex load `.agents/skills/` natively. For Claude Code, either also copy to `.claude/skills/` or symlink:

```bash
mkdir -p .claude/skills
ln -s ../../.agents/skills/iphone-duo-support .claude/skills/iphone-duo-support
```

## Use

In the app you want to migrate, with the skill installed, ask:

> Migrate this app to iPhone Duo compatibility.

The agent should detect the stack, report the SDK tier, grep for the four broken assumptions, then apply the matching reference (`native-ios`, `react-native`, `flutter`, or `other-stacks`).

## Layout of this repository

```
skills/iphone-duo-support/
  SKILL.md                 # workflow the agent loads first
  LICENSE
  agents/openai.yaml       # Codex / ChatGPT UI metadata
  references/
    hig.md                 # HIG distilled
    native-ios.md          # SwiftUI + UIKit
    react-native.md        # React Native + Expo
    flutter.md
    other-stacks.md        # Unity, Capacitor, general
    sources.md             # URLs and API name provenance
```

## Requirements for actually shipping Duo layout

- **Xcode 27.1** or later for the iPhone Duo simulator, Device Hub poses, and the iOS 27.1 SDK (full-screen + vertical bars).
- Rebuilding against 27.1 without fixing layout can look worse than a letterboxed older binary.

## License

MIT. Technical facts derive from Apple’s public developer material; this repo is not an Apple product.
