---
name: chameleon-esp-updater
description: End-to-end MECCHA CHAMELEON chameleonEsp update workflow from Dumper-7/SDK dump to staged tested DLLs. Use when the user asks to update, adapt, patch, rebuild, or diagnose chameleonEsp after a game update, SDK change, ESP crash, player name issue, role/enemy ESP issue, or wants a staged set of DLL builds to test.
---

# Chameleon ESP Updater

## Core Rule

Work from evidence, not guesses. Read the dumped SDK and current source at code level before editing. Cite exact local files/lines in notes when a field, class, function, or offset drives a change.

Resolve the user's real Desktop path at runtime. Do not hard-code `D:\Desktop` or `C:\Users\<name>\Desktop` unless the user explicitly gives that path.

This skill must be self-contained. Do not require, copy from, or compare against the creator's previous enhanced source folders, old DLLs, local final packages, or historical test workspaces. Only use a previous local build/source when the user explicitly asks for a regression comparison. In normal runs, reconstruct patches from the current upstream source, the current SDK evidence, and the patch rules in this skill.

Allowed normal-run inputs:
- current `phxgg/chameleonEsp` source downloaded from GitHub or supplied by the user
- current local MECCHA CHAMELEON game files and version evidence
- Visual Studio Build Tools/MSBuild detected on the user's machine
- Dumper-7/Xenos downloaded during the run or explicitly supplied by the user
- SDK generated during the run, or a fresh SDK dump explicitly supplied for the current game version
- crash logs, screenshots, and runtime feedback from the current test

Forbidden normal-run inputs:
- the creator's old enhanced source package
- old compiled DLLs
- old SDK dumps
- folders such as `D:\重要文件`, `D:\Desktop\chameleon-work`, `chameleonEsp-final-*`, or any previous local final/test package
- copying source files from a past successful adaptation just to reproduce fixes

If a useful fix was discovered in a previous session, encode the fix as a rule or code pattern in this skill, then apply it to the fresh upstream/source tree. Do not treat the old files themselves as part of the workflow.

## Desktop Path Discovery

Before creating work/output folders, determine `DESKTOP_ROOT`:
- Prefer an explicit user-provided output path.
- Otherwise query Windows Shell Desktop path, e.g. PowerShell:
  ```powershell
  [Environment]::GetFolderPath('Desktop')
  ```
- If the result is empty or invalid, check common redirected desktops such as `$env:USERPROFILE\Desktop` and `$env:OneDrive\Desktop`.
- Confirm the path exists or create it only when it is clearly the user's Desktop.
- Use `DESKTOP_ROOT` in all later paths and notes.

## Start From Zero Rule

Assume a fresh computer unless the user explicitly points to an existing working tree. Do not depend on prior folders, prior DLLs, prior dumps, or previous local build artifacts. Discover or create every required path during the run.

Default fresh-run workspace layout under `DESKTOP_ROOT`:
- `chameleon-work\upstream` for the original source
- `chameleon-work\dumper` for Dumper-7
- `chameleon-work\injector` for Xenos or another user-approved injector
- `chameleon-work\sdk-dump` for copied SDK output
- `chameleon-work\builds` for staged DLLs and hashes
- `chameleon-work\notes` for local evidence and test notes

## Version Gate / Route Selection

Before deciding whether to patch upstream or dump the SDK from zero, collect version evidence and choose the cheapest safe route.

Record three different version signals separately:
- **Game display version**: the player-facing version shown in the game UI, e.g. `2.5.0`. Prefer reading it from the title/settings screen if visible. If it is not exposed in local files, ask the user for the displayed version or ask for a screenshot.
- **Steam buildid**: read from `steamapps\appmanifest_4704690.acf`, e.g. `buildid`. This is the installed package build, not the game display version.
- **Engine/exe version**: read from `PenguinHotel-Win64-Shipping.exe` file version, e.g. `++UE5+Release-...`. Use this as supporting evidence only; do not compare it as the game version.

PowerShell evidence examples:
```powershell
$manifest = '<steam library>\steamapps\appmanifest_4704690.acf'
Select-String -LiteralPath $manifest -Pattern '"name"|"buildid"|"LastUpdated"|"installdir"'
$exe = '<game>\Chameleon\Binaries\Win64\PenguinHotel-Win64-Shipping.exe'
[System.Diagnostics.FileVersionInfo]::GetVersionInfo($exe) | Select-Object FileVersion,ProductVersion,ProductName
```

Resolve the author's current support level:
- Check the latest `phxgg/chameleonEsp` release/tag/README/commit notes or user-provided upstream package for a declared game version or Steam buildid.
- If internet access is available, verify the current GitHub state instead of relying on memory.
- If the author does not declare a game version, treat the upstream support level as unknown and use a compile/runtime probe before doing the full dump.

Route decision:
- If the installed game display version matches the author's declared supported version, and Steam buildid/date does not show a newer installed game than the author's update, use the lighter upstream patch route: apply the tested render crash, crowded-room name, stale-body/current-body ESP guard, English/Simplified Chinese, and CJK font patches to a fresh `phxgg/chameleonEsp` tree. The patch route must be implemented from the rules below, not by copying an old enhanced package.
- If the installed game display version is greater than the author's declared supported version, or the Steam buildid clearly changed after the author's latest update, use the full from-zero route: Dumper-7, fresh SDK, code-level SDK reading, and staged DLL builds.
- If the versions are equal but the patched upstream build still crashes, fails to compile against its SDK, or ESP fields behave like the SDK is stale, escalate to the full from-zero route.
- If author support is unknown, first try the upstream patch route only when it can be built quickly and safely. If it fails at SDK fields/classes, do not keep patching blindly; switch to the from-zero route.

When reporting this decision to the user, say exactly which route was chosen and include:
- installed game display version
- installed Steam buildid
- author declared supported version/buildid, or `unknown`
- reason for choosing upstream patch or full SDK dump

## Upstream Patch Route

Use this route only when the author's current source appears to target the installed game version. It produces a full-feature author build with local fixes, not the trimmed from-zero build.

1. Download or clone a fresh `https://github.com/phxgg/chameleonEsp` tree into `DESKTOP_ROOT\chameleon-work\upstream\chameleonEsp` or another clearly named fresh folder.
2. Do not ask the user for the creator's old enhanced source, old DLL, or final package. Treat the upstream tree as the only source input.
3. Read before editing:
   - `chameleonEsp\CheatManager.cpp`
   - `chameleonEsp\CheatManager.hpp`
   - `chameleonEsp\Main.cpp`
   - `chameleonEsp\Menu.cpp`
   - `chameleonEsp\Settings.cpp`
   - `chameleonEsp\Settings.hpp`
   - `chameleonEsp\includes.hpp`
   - `chameleonEsp\chameleonEsp.vcxproj`
4. Search for existing fixes and avoid duplicating them:
   ```powershell
   rg -n "snapshotMutex|snapshotLock|g_runtimeReady|TryCopyPlayerStateNameAndPawnWide|iLanguage|ImFontGlyphRangesBuilder|TryResolvePlayerStateActiveBody|activeBodyByState|GetGlyphRangesChinese" <repo>\chameleonEsp
   ```
5. Apply these self-contained patches, adapting only to names/fields that exist in the current source/SDK:
   - crash/render guard: protect `hkPresent`, `WndProc`, hotkeys, unload, and snapshot publishing; prefer an injection-safe native lock such as `SRWLOCK` over an object-owned `std::mutex` when `RenderEsp()` crashes in `msvcp140`
   - crowded-room player names: build a per-frame `PlayerState -> pawn/character/name` map from `GameState->PlayerArray`, PlayerState actors, `GetPawn()`, `PawnPrivate`, `TargetCharacter`, `OwnerCharacter_LINK`, `CustomPlayerName`, and `PlayerNamePrivate`
   - stale-body/current-body ESP guard: build `activeBodyByState` each frame and skip an actor only when the same PlayerState clearly points to another valid current body
   - English/Simplified Chinese toggle: add an internal translation helper and menu language option while preserving real player names
   - CJK font fix: compile as UTF-8 and load/merge CJK-capable Windows fonts so Chinese UI and player names do not render as boxes
6. Keep the author's existing features intact unless the user explicitly requests the trimmed from-zero feature set.
7. Build Release x64 and copy the output to a unique DLL name such as `chameleonEsp_author_enhanced_<date>.dll`.
8. Update the Xenos ESP profile to point to that exact DLL, then ask the user to test Enable, crowded-room names, Chinese text, role/enemy ESP, and dead-body filtering.

## Build Tools Prerequisite

Before downloading/building Dumper-7 or chameleonEsp, verify Visual Studio Build Tools/MSBuild is installed.

1. Search for MSBuild first:
   ```powershell
   Get-ChildItem "${env:ProgramFiles}\Microsoft Visual Studio\*\*\MSBuild\Current\Bin\MSBuild.exe" -ErrorAction SilentlyContinue
   Get-ChildItem "${env:ProgramFiles(x86)}\Microsoft Visual Studio\*\*\MSBuild\Current\Bin\MSBuild.exe" -ErrorAction SilentlyContinue
   ```
2. If no MSBuild is found, stop the workflow and tell the user to install Visual Studio Build Tools before continuing.
   - Preferred manual route: download `vs_BuildTools.exe` from Microsoft's Visual Studio Build Tools page or the Microsoft redirect `https://aka.ms/vs/17/release/vs_BuildTools.exe`.
   - Required workload: `Desktop development with C++`.
   - Required components: MSVC C++ toolset and Windows 10/11 SDK. Include recommended components unless disk space is a hard constraint.
   - After installation, ask the user to reopen Codex/PowerShell or continue after PATH/environment refresh.
3. If the user wants a command-line install and `winget` is available, suggest:
   ```powershell
   winget install --id Microsoft.VisualStudio.2022.BuildTools -e --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
   ```
4. Re-run MSBuild discovery after installation and record the chosen path in `DESKTOP_ROOT\chameleon-work\notes`.
5. Do not continue to Dumper-7 build, SDK replacement, or staged DLL builds until MSBuild is confirmed.

## Required Inputs

Locate or ask for:
- Upstream source or zip for `phxgg/chameleonEsp`; if missing, download current upstream source
- Dumper-7 source/release; if missing, download or use a user-provided archive
- Xenos injector source/release or another user-approved x64 DLL injector
- Game executable path, usually `...\MECCHA CHAMELEON\Chameleon\Binaries\Win64\PenguinHotel-Win64-Shipping.exe`
- Build tool path, usually Visual Studio Build Tools/MSBuild

Do not ask for a previous working DLL/source unless the task is explicitly regression comparison.

## Game Path Discovery

Find the game executable before Dumper-7 injection. Use evidence in this order:

1. If the user gives a path, verify it exists and ends with `PenguinHotel-Win64-Shipping.exe` or the current game shipping executable.
2. If the game is running, resolve the executable from the process:
   ```powershell
   Get-CimInstance Win32_Process -Filter "Name='PenguinHotel-Win64-Shipping.exe'" | Select-Object ProcessId,ExecutablePath
   ```
3. Query Steam install data and library folders:
   - read `HKCU:\Software\Valve\Steam` and `HKLM:\SOFTWARE\WOW6432Node\Valve\Steam` for `SteamPath` or `InstallPath`
   - read `steamapps\libraryfolders.vdf` under each Steam root
   - skip library paths whose drive/folder does not currently exist
   - search each library for `steamapps\common\MECCHA CHAMELEON\Chameleon\Binaries\Win64\PenguinHotel-Win64-Shipping.exe`
4. If Steam data is missing, search likely library roots on fixed drives only, avoiding full-drive recursion unless needed:
   - `*\SteamLibrary\steamapps\common\MECCHA CHAMELEON\...`
   - `*\Steam\steamapps\common\MECCHA CHAMELEON\...`
   - `*\Program Files (x86)\Steam\steamapps\common\MECCHA CHAMELEON\...`
5. If multiple candidates exist, prefer the one matching the running process path or newest file timestamp, and tell the user which one was selected.
6. If no candidate is found, ask the user to open Steam, right-click the game, choose Manage/Browse local files, then paste the opened folder path.

Record the resolved executable path and process name in `DESKTOP_ROOT\chameleon-work\notes`.

Use the executable path for verification and notes, not for launching by default. For online testing, tell the user to launch MECCHA CHAMELEON from Steam so Steam authentication/session setup is preserved.

## Dumper-7 / SDK Acquisition

This phase must happen before SDK reading when no fresh SDK dump is provided.

Prerequisite: complete **Build Tools Prerequisite** first. If MSBuild is missing, do not start this phase.

1. Locate the game executable and confirm the running process name, usually `PenguinHotel-Win64-Shipping.exe`.
2. Obtain Dumper-7:
   - use a user-provided `Dumper-7*.zip` if present
   - otherwise download from `https://github.com/Encryqed/Dumper-7`
3. Build or prepare Dumper-7 for x64 injection according to the Dumper-7 project files present on disk.
   - If `Dumper-7.sln` fails because optional sample projects such as `SDKTest.vcxproj` are missing, build `Dumper\Dumper.vcxproj` directly.
   - If the project requests an unavailable older MSVC toolset, pass the installed toolset explicitly, e.g. `/p:PlatformToolset=v145`.
4. Obtain an injector:
   - use a user-provided known-working injector if present
   - otherwise prefer the official Xenos release package from `https://github.com/DarthTon/Xenos/releases`, especially `https://github.com/DarthTon/Xenos/releases/download/2.3.2/Xenos_2.3.2.7z`
   - extract the release package into `DESKTOP_ROOT\chameleon-work\injector\Xenos-release`; Windows `tar -xf Xenos_2.3.2.7z -C <dest>` may work even when 7-Zip is not installed
   - use `Xenos64.exe` for the 64-bit game process
   - create a Xenos x64 profile at `DESKTOP_ROOT\chameleon-work\injector\chameleon-dumper.xpr64` after `Dumper-7.dll` is built
   - write the Xenos profile as plain ASCII/UTF-8 text without a UTF-16 BOM; do not use PowerShell `-Encoding Unicode`, because this can make Xenos load an empty profile
   - use Xenos-style empty XML tags such as `<procCmdLine/>`, `<initRoutine/>`, and `<initArgs/>`
   - set the profile to Existing process mode, Native inject mode, target process `PenguinHotel-Win64-Shipping.exe`, and image path pointing to the built `Dumper-7.dll`
   - tell the user to start the game first, then launch `Xenos64.exe --load <profile>` so DLL and process fields are prefilled; use `--run <profile>` only when the user wants immediate injection without GUI confirmation
   - if release download fails, download Xenos source from `https://github.com/DarthTon/Xenos` into `DESKTOP_ROOT\chameleon-work\injector`
   - initialize source dependencies before building source: `git submodule update --init --recursive`
   - if building from source, inspect project files and build x64 Release with the installed MSVC toolset
   - expect older Xenos/Blackbone source to fail on newer MSVC because of strict C++/const-string/compiler compatibility; do not spend the main update run rewriting third-party injector code unless the user explicitly asks
   - if Xenos cannot be built or a binary is not available, pause and ask the user to provide a trusted x64 DLL injector path
5. Ask the user to start the game normally through Steam and reach a stable state where UE objects are loaded.
   - Do not directly launch `PenguinHotel-Win64-Shipping.exe` unless the user explicitly asks, because direct executable launch can bypass Steam/session setup and fail to connect to servers.
   - After the user starts the game, verify the running process and executable path with `Get-CimInstance Win32_Process`.
6. Injection/F8 checkpoint:
   - Tell the user to start the game first, then open the generated Xenos profile with `Xenos64.exe --load <profile>`, confirm the game process and `Dumper-7.dll` are populated, click Inject, then press `F8`, if that is how the local Dumper-7 build is configured.
   - If computer-control tools are available and the user explicitly authorizes control, Codex may help focus the game/injector and press `F8`; otherwise treat this as a manual user step.
   - Do not assume dump success. Wait for the user to say the SDK was generated or verify output files exist.
7. Auto-detect generated output by searching recent folders/files for SDK indicators:
   - folders named `SDK`, `CppSDK`, `Dumper-7`, `Output`, `Generated`, or game/process names
   - files such as `SDK.hpp`, `Basic.hpp`, `Engine_classes.hpp`, `*_classes.hpp`, `*_functions.cpp`
8. Copy the detected generated SDK into `DESKTOP_ROOT\chameleon-work\sdk-dump` before editing the target project.
9. If no SDK output is found, ask the user for the Dumper output path and do not continue with guessed SDK data.

## SDK Reading Workflow

Before editing:
1. Enumerate candidate SDK files with `rg --files`, not manual browsing.
2. Read class inheritance and fields for:
   - main character classes
   - hunter/survivor classes or runtime class names
   - player state classes
   - game state/player array
   - enemy AI classes
   - skeletal mesh/bone functions
   - ProcessEvent wrappers used by risky features
3. Verify names and fields directly in generated headers/functions:
   - `CustomPlayerName`
   - `PlayerNamePrivate`
   - `PawnPrivate`
   - `GetPawn()`
   - `TargetCharacter`
   - `OwnerCharacter_LINK`
   - current-body links such as `TargetCharacter`, `OwnerCharacter_LINK`, `GetPawn()`, and `PawnPrivate`
   - `World->GameState->PlayerArray`
   - role indicators or runtime class-name alternatives
   - death flags, health fields, ragdoll/simulation signals
   - `GetNumBones()`
4. Build a short "evidence table" before editing for name and role logic: field/function, declaring class, file/line, type, and fallback priority.
5. Record short evidence notes before editing: class, file, line, and why it matters.
6. If a field is missing or renamed, stop relying on it. Prefer robust fallback chains and runtime checks.

## Six Build Stages

Always build six staged DLLs from shallow to deep when adapting after a game update. Give each DLL a unique filename and keep a short changelog next to it. The user tests them in order and reports the first working/broken stage.

### Build 1: Baseline Compile

Goal: prove the upstream source plus newly dumped SDK can compile.

Allowed changes:
- copy/merge generated SDK into the project
- include/build fixes only
- project file fixes only if required
- no behavior changes

Output name pattern: `chameleonEsp_v1_baseline_compile.dll`

### Build 2: Crash/Render Safety

Goal: prevent render-thread/UObject and transition crashes.

Apply only when not already present upstream:
- gather live UObject data on the game thread
- render from a snapshot only
- publish empty snapshot when world/player context is invalid
- prefer a Windows `SRWLOCK` or other injection-safe native lock for the render snapshot instead of a heap-object `std::mutex` when `RenderEsp()` crashes inside `msvcp140`
- gate render calls with a runtime-ready flag that is set only after globals are constructed and hooks are enabled, and cleared before unload teardown
- guard null and stale UObjects
- avoid stale cached `UFunction*` when calling dynamic Blueprint functions
- check `GetNumBones()` before skeleton/box bone reads

Output name pattern: `chameleonEsp_v2_crash_safety.dll`

### Build 3: Player Name Fix

Goal: fix `Unknown`, `Player`, or missing names, especially in crowded rooms.

First auto-discover the name/pawn links from the dumped SDK instead of hard-coding yesterday's class names:
- search SDK headers for `CustomPlayerName`, `PlayerNamePrivate`, `PawnPrivate`, `GetPawn`, `TargetCharacter`, `OwnerCharacter`, `OwnerCharacter_LINK`, `MyPlayerState`, `MyPlayerState_LINK`, `PlayerArray`, and `GameState`
- read the exact declaring class and field type before editing
- if the new SDK has both normal and game-mode-specific PlayerState classes, support both
- if a generated parameters file fails to compile because of bad Dumper-7 assertions, avoid wrapper calls and use direct fields or dynamic `GetFunction` where safe

Use a layered name resolver:
- prefer valid `CustomPlayerName`
- fall back to `PlayerNamePrivate`
- cache last valid name per actor
- map PlayerState to pawn with `GetPawn()`, then `PawnPrivate`
- use game-specific links such as `TargetCharacter` and `OwnerCharacter_LINK`
- scan `World->GameState->PlayerArray`
- optionally scan `APlayerState` actors when GameState is incomplete
- keep a per-frame map from pawn/character actor to discovered name, then let actor ESP lookup use that map before returning `Unknown`
- protect FString reads against null/invalid data; if a copied name is empty, continue the fallback chain instead of stopping early

Output name pattern: `chameleonEsp_v3_player_names.dll`

### Build 4: Role/Enemy ESP

Goal: restore hunter/survivor/enemy display without trusting stale SDK role fields.

First auto-discover role evidence from the dumped SDK:
- search for old and new role indicators such as `IsHunter`, `Hunter`, `Survivor`, `KingCharacter`, `LinkCharacter`, `PlayerClass`, role enums, game-mode class names, and game-state fields
- search for current-body evidence such as `GetPawn()`, `PawnPrivate`, `TargetCharacter`, `OwnerCharacter_LINK`, `MyPlayerState`, `MyPlayerState_LINK`, `LastMyPlayerState`, `PlayerArray`, and body/class fields such as `CurrentBodyClass`
- inspect class inheritance and runtime Blueprint class names for player characters
- prefer explicit replicated fields when present, then game-state links such as `KingCharacter`, then runtime class-name fallback

Use:
- runtime class-name fallback, e.g. names containing `Hunter`, `Survivor`, `King`, or game-specific role markers
- conservative `IsEnemy`: if roles are unknown, do not hide ESP under Enemy Only
- dead/ragdoll filtering for players
- stale-body/current-body filtering: each frame build `activeBodyByState` from PlayerState evidence (`GetPawn()`, `PawnPrivate`, `TargetCharacter`, `OwnerCharacter_LINK`) and skip an actor only when the same PlayerState clearly points to another valid actor; this prevents a dead survivor's old body from staying labeled as `Survivor` after the real player has switched to a hunter body
- keep the current-body filter conservative: if PlayerState is missing, the active body is null, or the mapped actor is invalid, do not hide the ESP entry
- enemy AI scan only if SDK evidence confirms classes and fields
- if role display is uncertain, surface `Player`/unknown only as a temporary staged build and continue SDK reading before final

Output name pattern: `chameleonEsp_v4_roles_enemy.dll`

### Build 4.5: CJK/Unicode Text Fix

Goal: prevent Chinese/Japanese/Korean names or UI strings from rendering as square boxes.

Apply before final packaging when the menu or ESP may display non-ASCII text:
- inspect ImGui initialization in `Main.cpp` or the local renderer setup
- load a default Latin font first, then merge CJK-capable Windows fonts when present
- prefer Windows font paths in this order for Simplified Chinese: `C:\Windows\Fonts\msyh.ttc`, `C:\Windows\Fonts\simhei.ttf`, `C:\Windows\Fonts\simsun.ttc`
- include `io.Fonts->GetGlyphRangesChineseSimplifiedCommon()` for Chinese, plus Japanese/Korean ranges when the matching fonts exist
- use `ImFontConfig::MergeMode = true` for merged fonts and set `ImFontFlags_NoLoadError` or equivalent if supported by the local ImGui version
- call `io.Fonts->Build()` after adding fonts
- keep all player-name strings UTF-8 when passing them to ImGui; when changing names through `FString`, convert UTF-8 to wide string before constructing `SDK::FString`
- if the codebase already has CJK font merging, verify the font file names still exist on the target Windows install and add fallbacks if only one font is listed

Do not use bitmap-only or narrow ASCII fonts for the final DLL.

### Build 5: Feature Trim

Goal: keep only the features the user wants.

Keep:
- ESP
- Teleport
- Rename
- Prevent Server Kick

Remove, disable, or hide:
- kill features
- magnet/item features
- no cooldown
- infinite ammo
- forced visibility
- unrelated debug controls unless needed for current diagnosis

Do not leave dead UI buttons that call removed logic.

Output name pattern: `chameleonEsp_v5_trimmed_features.dll`

### Build 6: Final Package

Goal: deliver the final tested DLL and the minimal local source state needed to rebuild it.

Produce:
- final DLL
- final source folder if code edits were made
- short local build/test notes with build command, DLL hash, tested stage, and remaining unknowns
- final notes stating which player-name resolver paths, role resolver paths, and CJK font paths are active

Output name pattern: `chameleonEsp_v6_final.dll`

## Build Verification

Use MSBuild Release x64. Example:

```powershell
& 'C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe' '<repo>\chameleonEsp.slnx' /p:Configuration=Release /p:Platform=x64 /p:PlatformToolset=v145 /m /nologo
```

If the repo uses another toolset, read the project file and adapt the command.

After every build:
- copy DLL to a stable output folder under `DESKTOP_ROOT`
- compute SHA256
- update the Xenos test profile so its `imagePath` points to the newly built staged DLL
- write stage number, source path, build command, warnings/errors, and test purpose
- for builds at or after player-name/role stages, record whether name resolver, role resolver, and CJK font fallback are included
- pause for user injection/runtime testing before moving to the next behavioral stage unless the user explicitly asks to continue without testing
- do not overwrite older stage DLLs

## Runtime Test Feedback Loop

After producing each staged DLL, ask the user to test that exact DLL before deeper changes.

Tell the user:
- launch MECCHA CHAMELEON normally from Steam
- open the current Xenos ESP profile, not the Dumper profile
- verify Xenos `imagePath` points to the staged DLL just built
- inject the DLL
- enter a room state relevant to the current stage, such as lobby, crowded room, in-game spectator, or active game

If the game crashes or the DLL fails:
- first try to read the Unreal crash files automatically before asking the user to copy text from the window
- collect the newest crash package from `%LOCALAPPDATA%\Chameleon\Saved\Crashes`, especially `CrashContext.runtime-xml`, `UEMinidump.dmp`, and `CrashReportClient.ini`
- also read `%LOCALAPPDATA%\CrashReportClient\Saved\Logs\CrashReportClient.log` and recent backup logs
- if the user provides or screenshots a `LoginId`, search recent crash packages for that exact ID, but do not treat `LoginId` as unique; combine it with newest `LastWriteTime`, `CrashGUID`, exception text, and crashing module/DLL name
- extract `CrashGUID`, `LoginId`, `ErrorMessage`, `SecondsSinceStart`, `ExecutableName`, `EngineVersion`, `CallStack`, `PCallStack`, module names, and any source file/line
- only ask the user to copy the full crash text/log if the crash files cannot be found, are incomplete, or belong to a different crash
- treat source file and line numbers in the crash as the next primary evidence
- do not guess the next fix from symptoms alone when a crash log is available
- map the crash frame to the current source line before editing
- build a new uniquely named DLL for the fix, update Xenos profile to that DLL, and ask the user to retest

Useful PowerShell for crash collection:
```powershell
$crashRoot = Join-Path $env:LOCALAPPDATA 'Chameleon\Saved\Crashes'
$latest = Get-ChildItem -LiteralPath $crashRoot -Directory |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1
Get-ChildItem -LiteralPath $latest.FullName -Force
Select-String -LiteralPath (Join-Path $latest.FullName 'CrashContext.runtime-xml') `
  -Pattern 'CrashGUID|LoginId|ErrorMessage|SecondsSinceStart|ExecutableName|EngineVersion|CallStack|PCallStack|chameleonEsp|CheatManager|Main.cpp|msvcp140'
Get-Content -LiteralPath (Join-Path $env:LOCALAPPDATA 'CrashReportClient\Saved\Logs\CrashReportClient.log') -Tail 160
```

If the failure is visual or behavioral rather than a crash:
- ask for screenshots and the room/game state: waiting lobby, crowded lobby, observer/spectator, active game, after round transition, or after reinjection
- record which toggles were enabled
- prefer focused one-issue patches over broad rewrites

## Xenos Profile Updates After Builds

After each staged chameleonEsp DLL build, create or update a matching Xenos profile for testing that exact DLL.

Rules:
- Keep the Dumper-7 profile separate, e.g. `DESKTOP_ROOT\chameleon-work\injector\chameleon-dumper.xpr`, pointing to `Dumper-7_x64_Release.dll`.
- For ESP test builds, create a separate profile such as `DESKTOP_ROOT\chameleon-work\injector\chameleon-stage-current.xpr` or `chameleon-final.xpr`.
- Also copy the active ESP profile to `DESKTOP_ROOT\chameleon-work\injector\Xenos-release\XenosCurrentProfile.xpr` when using the official Xenos release folder, so opening `Xenos64.exe` directly loads the latest test DLL.
- Write profiles as plain ASCII/UTF-8 text without UTF-16 BOM. Do not use PowerShell `-Encoding Unicode`.
- Preserve Existing process mode and Native inject mode:
  - `<processMode>0</processMode>`
  - `<injectMode>0</injectMode>`
  - `<procName>PenguinHotel-Win64-Shipping.exe</procName>`
- Set `<imagePath>` to the exact staged DLL the user should test next. If a DLL cannot be overwritten because the game/injector still holds it open, write a new uniquely named DLL and point the Xenos profile to that new filename.
- Tell the user which profile to open and which DLL it points to before testing.

Profile template:
```xml
<XenosConfig>
	<imagePath>DESKTOP_ROOT\chameleon-work\builds\chameleonEsp_v6_final.dll</imagePath>
	<manualMapFlags>0</manualMapFlags>
	<procName>PenguinHotel-Win64-Shipping.exe</procName>
	<hijack>0</hijack>
	<unlink>0</unlink>
	<erasePE>0</erasePE>
	<close>0</close>
	<krnHandle>0</krnHandle>
	<injIndef>0</injIndef>
	<processMode>0</processMode>
	<injectMode>0</injectMode>
	<delay>0</delay>
	<period>0</period>
	<skip>0</skip>
	<procCmdLine/>
	<initRoutine/>
	<initArgs/>
</XenosConfig>
```

## Packaging Rules

For normal skill runs:
- Do not create extra publishing packages or public-facing documents unless the user explicitly asks.
- Keep outputs focused on staged DLLs, the working source folder, hashes, and local test notes.
- Exclude `.vs`, `x64`, `.git`, `.obj`, `.pdb`, `.ilk`, `.iobj`, `.ipdb`, `.dll`, `.exe`, `.lib`, `.exp`, logs, and tlogs from any copied source folder unless the user explicitly asks for those artifacts.

## Communication

Tell the user:
- which stage DLL they should test next
- what changed compared with the previous stage
- what to screenshot/report if it fails
- if injection/runtime crashes, keep the Unreal crash reporter window open; Codex should first read the generated crash files automatically, and only ask for copied text if the files cannot be matched
- what remains untested
- when Dumper-7 injection is needed, start the game from Steam manually first, then let Codex detect the process and open Xenos

When writing public notes, be direct:
- "I tested ESP only"
- "Other non-ESP features were not fully tested because I do not currently use them"
