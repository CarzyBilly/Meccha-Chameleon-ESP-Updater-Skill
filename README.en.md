# Chameleon ESP Updater Skill

[简体中文](README.md) | English

This is a custom Codex skill for quickly fixing and adapting `phxgg/chameleonEsp` after a MECCHA CHAMELEON update. It has two routes: if the upstream project already supports the installed game version, Codex patches the author's full source tree, keeps the original feature set, fixes the verified bugs, and adds enhancements such as English/Simplified Chinese UI and CJK font support. If the installed game version is newer than upstream, Codex switches to the from-zero workflow: find the game, prepare tools, dump the SDK, read the SDK, build staged DLLs, test them, read crash logs, and continue patching.

This workflow is designed for two cases: first, the upstream project matches the current game version but still needs local fixes for injection crashes, missing player names, CJK square-box text, or similar issues; second, the game has just updated and upstream has not caught up yet, so a usable build is needed quickly to restore core features. The from-zero quick-adaptation build is not a full replacement for the upstream project. It usually covers only part of the original feature set, with ESP as the main focus and only a few practical features kept.

## Purpose

Use this skill when:

- The upstream source matches the installed game version, but injection, Enable, player names, or CJK text still need patches on top of the full version.
- The game updated and the existing `chameleonEsp` build crashes or no longer injects correctly.
- The SDK may have changed and needs to be regenerated with Dumper-7.
- ESP names show as `Unknown`, `Player`, or disappear in crowded rooms.
- hunter/survivor/enemy ESP detection is broken.
- Chinese UI text or player names render as square boxes.
- Multiple staged DLLs are needed so the user can test them one by one.

There are two output modes:

- Author-source patch build: keeps the upstream feature set and applies only the needed bug fixes plus English/Simplified Chinese and font enhancements.
- From-zero quick-adaptation build: focuses on quickly restoring these features by default.

- ESP
- Teleport
- Rename / Name Changer
- Prevent Server Kick

In the from-zero quick-adaptation build, other features are hidden, disabled, or treated as out of scope.

## Installation

The recommended installation path is GitHub-based. After this repository is uploaded to GitHub, other users can ask their own Codex:

```text
Install the chameleon-esp-updater skill from https://github.com/CarzyBilly/Chameleon-ESP-Updater-Skill
```

Or, more explicitly:

```text
Install skills/chameleon-esp-updater from https://github.com/CarzyBilly/Chameleon-ESP-Updater-Skill into my Codex skills directory.
```

Codex should install it to a path similar to:

```text
C:\Users\<your-user-name>\.codex\skills\chameleon-esp-updater
```

Manual installation is also possible: copy `skills\chameleon-esp-updater` from this repository into the Codex skills directory:

```text
C:\Users\<your-user-name>\.codex\skills\chameleon-esp-updater
```

Expected structure:

```text
.codex
└── skills
    └── chameleon-esp-updater
        └── SKILL.md
```

Then trigger it in Codex with natural language, for example:

```text
The game updated. Use chameleon-esp-updater and adapt it from zero.
```

or:

```text
Dump the SDK again and build six staged DLLs for testing.
```

## Workflow

The skill follows this order:

1. Detect the user's real Desktop path.
2. Read the game display version, Steam buildid, and exe/UE version.
3. Compare them with the version currently declared or released by the upstream author.
4. If the installed game version matches upstream, use the lighter route: prepare fresh upstream source and apply the tested crash, player-name, English/Simplified Chinese, and CJK font patches.
5. If the installed game version is newer than upstream, or Steam buildid clearly moved after upstream last updated, use the from-zero route and dump a fresh SDK.
6. Create the `chameleon-work` workspace.
7. Check Visual Studio Build Tools / MSBuild.
8. Locate the game install path and running process.
9. Prepare or download Dumper-7.
10. Prepare or download Xenos.
11. Guide the user to launch the game from Steam and manually complete Dumper injection plus the F8 dump step.
12. Detect the generated SDK output.
13. Read important SDK classes, fields, and functions at code level.
14. Build staged DLLs.
15. Update the Xenos profile after every build.
16. Continue patching based on screenshots, behavior, or crash logs from user testing.
17. On crashes, read Unreal Crash Reporter files automatically before asking the user to copy text from the crash window.

The workflow keeps these version signals separate: `2.5.0` style values are player-facing game versions, Steam `buildid` is the installed package build, and `++UE5+Release-...` is the engine/exe version. Route selection primarily uses the game display version and upstream declaration; Steam buildid is used to catch same-version package updates.

## Build Stages

The workflow builds from shallow to deep:

```text
v1_baseline_compile      prove upstream source + new SDK can compile
v2_crash_safety          fix startup/render crashes
v3_player_names          fix Unknown / Player / missing crowded-room names
v4_roles_enemy           fix hunter/survivor/enemy ESP
v4.5_cjk_font            fix CJK square-box text
v5_feature_trim          keep only target features
v6_final                 final test build
```

## Crash Logs

When the game crashes, the skill first checks:

```text
%LOCALAPPDATA%\Chameleon\Saved\Crashes
%LOCALAPPDATA%\CrashReportClient\Saved\Logs
```

It extracts:

```text
CrashGUID
LoginId
ErrorMessage
EngineVersion
CallStack
PCallStack
crashing DLL name
source file and line number
```

`LoginId` is not treated as unique, because it can repeat across multiple crashes. The workflow combines it with timestamp, CrashGUID, crashing module, and exception text.

## Referenced Projects

- Visual Studio Build Tools: https://visualstudio.microsoft.com/visual-cpp-build-tools/
- chameleonEsp: https://github.com/phxgg/chameleonEsp
- Dumper-7: https://github.com/Encryqed/Dumper-7
- Xenos: https://github.com/DarthTon/Xenos

This repository does not include third-party source code, binaries, SDK dumps, release packages, or compiled DLLs. It only contains the Codex skill workflow.

## Positioning

This skill is a fast-update workflow, not a long-term fork of `chameleonEsp`. The upstream project is more complete. This workflow is intentionally practical: when the game updates, it focuses on quickly restoring a usable ESP-oriented build. The generated output usually covers only around 60-70% of the original project's feature surface, centered on ESP, Teleport, Rename / Name Changer, and Prevent Server Kick.

## Disclaimer

This repository contains a Codex skill document for documenting an adaptation, debugging, and build workflow.

Use it only in environments where you have permission. You are responsible for complying with game rules, platform rules, open-source licenses, third-party tool licenses, terms of service, and local law. This repository is not affiliated with MECCHA CHAMELEON, phxgg/chameleonEsp, Dumper-7, or Xenos, and does not represent their maintainers.

Any build output, testing behavior, or result produced through this skill is the user's own responsibility.
