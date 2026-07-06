# Chameleon ESP Updater Skill

简体中文 | [English](README.en.md)

这是一个给 Codex 使用的自定义 skill，用来在 MECCHA CHAMELEON 更新后，按固定流程快速修复和重新适配 `phxgg/chameleonEsp`。它有两条路线：如果原作者项目已经适配当前游戏版本，就在作者的完整源码上打补丁，保留原项目功能，同时修复已验证的问题并加入中英双语、中文字体等增强；如果游戏版本已经超过原作者当前版本，则从 0 开始完成找游戏、准备工具、Dump SDK、阅读 SDK、分阶段构建、测试、读取崩溃日志和继续修复。

这个工作流主要是为两种情况准备的：一是原项目可用但存在本地注入崩溃、玩家名字缺失、中文方块等问题，需要在完整版基础上打补丁；二是游戏刚更新而原项目暂时还没跟进，需要快速生成一个可用版本先恢复核心功能。从 0 生成的快速适配版不是原作者项目的完整替代品，通常只覆盖原项目功能的一部分，重点维护 ESP，以及少量我实际会用到的功能。

## 用途

这个 skill 适合以下情况：

- 原作者源码对应当前游戏版本，但注入、Enable、名字、中文显示等存在 bug，需要在完整版上打补丁。
- 游戏更新后，原来的 `chameleonEsp` 无法启动或注入后崩溃。
- SDK 可能变了，需要重新使用 Dumper-7 生成 SDK。
- ESP 名字显示为 `Unknown`、`Player`，或者在人多房间名字缺失。
- 猎人/幸存者/敌人 ESP 判断失效。
- 中文 UI 或中文名字显示成方块。
- 需要按浅到深生成多版 DLL，让使用者逐个测试。

输出分两类：

- 作者源码补丁版：保留原作者项目的功能，只做必要 bug 修复和中英双语/字体增强。
- 从 0 快速适配版：为了尽快恢复可用，默认重点保留下面这些功能。

- ESP
- Teleport
- Rename / Name Changer
- Prevent Server Kick

从 0 快速适配版里，其他功能会被隐藏、禁用或不作为目标维护。

## 使用方法

推荐像普通 GitHub 项目一样安装。你把本仓库上传到 GitHub 后，其他用户可以在自己的 Codex 里直接说：

```text
请从 https://github.com/CarzyBilly/Chameleon-ESP-Updater-Skill 安装 chameleon-esp-updater skill
```

或者更明确一点：

```text
请把 https://github.com/CarzyBilly/Chameleon-ESP-Updater-Skill 里的 skills/chameleon-esp-updater 安装到我的 Codex skills 目录
```

Codex 会把 skill 安装到类似下面的位置：

```text
C:\Users\<用户名>\.codex\skills\chameleon-esp-updater
```

也可以手动安装：把本仓库里的 `skills\chameleon-esp-updater` 文件夹复制到 Codex 的 skills 目录：

```text
C:\Users\<你的用户名>\.codex\skills\chameleon-esp-updater
```

最终结构应该类似：

```text
.codex
└── skills
    └── chameleon-esp-updater
        └── SKILL.md
```

然后在 Codex 里用自然语言触发，例如：

```text
游戏更新了，请用 chameleon-esp-updater 从 0 开始适配。
```

或者：

```text
重新 dump SDK，然后分阶段构建 6 个 DLL 给我测试。
```

## 工作流程

skill 会按这个顺序执行：

1. 自动识别用户真实桌面路径。
2. 读取游戏显示版本、Steam buildid 和 exe/UE 版本。
3. 对比原作者当前发布或声明支持的版本。
4. 如果游戏版本和原作者一致，优先走轻量路线：下载/解压原作者源码，然后打已验证的崩溃、名字、中英双语和字体补丁。
5. 如果本地游戏版本高于原作者，或者 Steam buildid 明显更新而作者还没跟进，则走从 0 路线：重新 dump SDK。
6. 创建 `chameleon-work` 工作目录。
7. 检查 Visual Studio Build Tools / MSBuild。
8. 找到游戏安装路径和运行进程。
9. 准备或下载 Dumper-7。
10. 准备或下载 Xenos。
11. 引导用户从 Steam 启动游戏，并手动完成 Dumper 注入和 F8 dump。
12. 自动寻找生成的 SDK。
13. 逐字阅读 SDK 里的关键类、字段和函数。
14. 分阶段构建 DLL。
15. 每次构建后更新 Xenos 配置。
16. 用户测试后，根据截图、现象或崩溃日志继续修。
17. 崩溃时优先自动读取 Unreal Crash Reporter 生成的文件，不要求用户先复制窗口文字。

版本判断里不要把三个版本混在一起：`2.5.0` 这类是游戏显示版本，Steam `buildid` 是安装包构建号，`++UE5+Release-...` 是引擎/exe 版本。优先用游戏显示版本和作者声明对比，Steam buildid 用来判断同版本下是否又更新了包。

## 分阶段构建

skill 设计为从浅到深生成多个版本：

```text
v1_baseline_compile      只保证原项目 + 新 SDK 能编译
v2_crash_safety          修启动/渲染崩溃
v3_player_names          修 Unknown / Player / 人多没名字
v4_roles_enemy           修猎人/幸存者/敌人 ESP
v4.5_cjk_font            修中文方块字
v5_feature_trim          只保留目标功能
v6_final                 最终测试版
```

## 崩溃日志读取

如果游戏崩溃，skill 会优先读取：

```text
%LOCALAPPDATA%\Chameleon\Saved\Crashes
%LOCALAPPDATA%\CrashReportClient\Saved\Logs
```

它会提取：

```text
CrashGUID
LoginId
ErrorMessage
EngineVersion
CallStack
PCallStack
崩溃 DLL 名
源码文件和行号
```

注意：`LoginId` 可能在多次崩溃中重复，不能单独当作唯一判断依据，需要结合最新时间、CrashGUID、崩溃模块和错误信息。

## 依赖工具

使用这个 skill 时，通常会涉及以下工具或项目：

- Visual Studio Build Tools: https://visualstudio.microsoft.com/visual-cpp-build-tools/
- chameleonEsp: https://github.com/phxgg/chameleonEsp
- Dumper-7: https://github.com/Encryqed/Dumper-7
- Xenos: https://github.com/DarthTon/Xenos

本仓库不包含这些第三方项目的源码、二进制文件或发布包。skill 只记录流程和操作规则。

## 项目定位

这个 skill 的目标是快速适配最新游戏版本，而不是长期分叉维护 `chameleonEsp`。原作者项目功能更多、更完整；这里的流程更偏实用，优先保证游戏更新后能第一时间恢复可用的 ESP。生成出来的版本通常只保留原项目大约六七成的功能，主要围绕 ESP、Teleport、Rename / Name Changer 和 Prevent Server Kick。

## 声明

这个仓库只是 Codex skill 文本，用于记录自动化适配、调试和构建流程。

请只在你有权限的环境中使用，并自行遵守游戏、平台、开源项目和第三方工具的许可协议、服务条款和当地法律。仓库作者不隶属于 MECCHA CHAMELEON、phxgg/chameleonEsp、Dumper-7 或 Xenos 项目，也不代表这些项目维护者。

使用本 skill 产生的任何构建结果、测试行为和后果由使用者自行负责。
