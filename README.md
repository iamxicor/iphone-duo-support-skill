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

## Clone this repository

This repository **is** the skill. Clone it first, then copy `skills/iphone-duo-support/` into the app you want to migrate (or into your user-level skills folder). Copy the **whole folder**, not only `SKILL.md`, so the agent can load `references/` on demand. The folder name must stay `iphone-duo-support` to match the skill `name`.

The repo is **private**. You need GitHub access (HTTPS with a PAT, or SSH with a key added to your GitHub account).

```bash
# HTTPS (prompts for GitHub credentials / token)
git clone https://github.com/himanshu-kushwah/iphone-duo-support-skill.git

# or SSH
git clone git@github.com:himanshu-kushwah/iphone-duo-support-skill.git

cd iphone-duo-support-skill
```

If you use the GitHub CLI:

```bash
gh repo clone himanshu-kushwah/iphone-duo-support-skill
cd iphone-duo-support-skill
```

Set `SKILL_SRC` to this clone so the copy commands below work from any directory:

```bash
export SKILL_SRC="$PWD/skills/iphone-duo-support"
```

Then `cd` into the **app repository** you want to make Duo-ready (not this skill repo, unless you are only installing for yourself).

## Install

Install into the app repo (project, shared with teammates) **or** your home directory (personal, all projects on this machine). Project copies should be committed.

### Cursor

Cursor loads skills from `.agents/skills/`, `.cursor/skills/`, `~/.agents/skills/`, and `~/.cursor/skills/`. It also reads Claude/Codex folders (`.claude/skills/`, `.codex/skills/`).

**Project** (commit this so Cloud Agents and teammates get it):

```bash
cd /path/to/your-app
mkdir -p .agents/skills
cp -R "$SKILL_SRC" .agents/skills/
git add .agents/skills/iphone-duo-support
git commit -m "Add iPhone Duo support skill"
```

**Personal** (this machine only):

```bash
mkdir -p ~/.cursor/skills
cp -R "$SKILL_SRC" ~/.cursor/skills/
```

Personal `~/.cursor/skills/` copies stay local unless you turn on **Settings → Agents → Sync Skills for Cloud Agents**.

In Agent chat, type `/iphone-duo-support` or ask to migrate the app to iPhone Duo. Open **Customize → Skills** if it does not appear; restart Agent after a first-time copy.

### Claude Code

Claude Code loads `.claude/skills/` in the repo (and parent dirs up to the repo root) and `~/.claude/skills/` for you personally. Cloud / Cowork sessions **do not** read `~/.claude/skills/`; commit a project copy for those.

**Project:**

```bash
cd /path/to/your-app
mkdir -p .claude/skills
cp -R "$SKILL_SRC" .claude/skills/
git add .claude/skills/iphone-duo-support
git commit -m "Add iPhone Duo support skill"
```

**Personal:**

```bash
mkdir -p ~/.claude/skills
cp -R "$SKILL_SRC" ~/.claude/skills/
```

Invoke with `/iphone-duo-support`. Claude Code watches the skills directory; a restart is usually unnecessary.

### Codex

Codex scans `.agents/skills` from the current directory up to the repo root, plus `~/.agents/skills`. This skill includes `agents/openai.yaml` for Codex/ChatGPT display metadata.

**Project:**

```bash
cd /path/to/your-app
mkdir -p .agents/skills
cp -R "$SKILL_SRC" .agents/skills/
git add .agents/skills/iphone-duo-support
git commit -m "Add iPhone Duo support skill"
```

**Personal:**

```bash
mkdir -p ~/.agents/skills
cp -R "$SKILL_SRC" ~/.agents/skills/
```

Invoke with `$iphone-duo-support` or `/skills`. Codex detects new skills automatically; restart Codex if one is missing.

You can also ask Codex `$skill-installer` to pull from this repo if you use that workflow; a manual copy is enough.

### One copy for Claude, Codex, and Cursor

If the same app repo is used with all three tools, install once under `.agents/skills/` and symlink for Claude:

```bash
cd /path/to/your-app
mkdir -p .agents/skills .claude/skills
cp -R "$SKILL_SRC" .agents/skills/
ln -s ../../.agents/skills/iphone-duo-support .claude/skills/iphone-duo-support
```

Cursor and Codex load `.agents/skills/` natively. Claude Code follows the symlink in `.claude/skills/`.

## Use

In the **app** you want to migrate, with the skill installed, ask:

> Migrate this app to iPhone Duo compatibility.

Or invoke it by name:

| Tool | Invoke |
| --- | --- |
| Cursor | `/iphone-duo-support` |
| Claude Code | `/iphone-duo-support` |
| Codex | `$iphone-duo-support` |

The agent should:

1. Detect the stack (SwiftUI, UIKit, React Native/Expo, Flutter, Unity, Capacitor/web, or other).
2. Report the iOS SDK / Xcode tier.
3. Grep for the four broken assumptions (fixed widths, orientation locks, symmetric insets, idiom/device-type gates).
4. Apply the matching reference under `references/` and verify poses in Xcode 27.1 Device Hub when available.

Example follow-ups:

- “Audit this React Native app for iPhone Duo; don’t bump the SDK yet.”
- “Adopt ArrangementView on the player screen.”
- “Fix asymmetric safe-area math in UIKit.”

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
