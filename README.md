# Limitless Concert (ywzj_midi)

*修改 By Lynn*

一个基于 Minecraft Forge 1.20.1 的音乐扩展模组，为游戏带来完整的 MIDI 演奏、乐队协同与自定义音乐播放器体验。本仓库包含所有源代码、资源以及构建脚本。

## 功能亮点
- **大型乐器库**：`AllInstruments` 统一注册钢琴、弦乐、木管、铜管与打击乐，对应方块/物品、音域与变奏采样。
- **高保真声音系统**：`AllSounds` 动态生成每个音符的 `SoundEvent`，并通过 `NotePlayer` 实现音量、延音踏板与循环控制。
- **MIDI 指挥流程**：服务器侧的 `ServerMidiPlayer` 可解析 `.mid` 文件，结合 `ConductorConfig` 分配声部、循环播放并驱动假人指挥姿势。
- **实时网络同步**：自定义 `SimpleChannel` 覆盖音符、音乐流、姿态、文件上传等报文，确保客户端/服务器之间的演奏一致性。
- **自定义音乐播放**：`MusicPlayerItem` 搭配多种客户端音频流实现本地文件回放、网络流播放与便携式扬声器共享。

## 开发环境
- JDK 17（Forge 自 1.18 起要求）
- Gradle（项目自带 `gradlew/gradlew.bat` 脚本）
- Minecraft Forge 47.1.44（1.20.1）
- IntelliJ IDEA / VS Code 等 Java IDE（可使用官方 MDK 工作流导入）

> **网络资源**：仓库默认启用 `mixingradle` 与 `shadow` 插件，首次构建时 Gradle 会自动下载 Forge、Mixin 与音频相关依赖。

## 快速开始
```bash
# 克隆仓库
git clone <repo-url>
cd ywzj_midi

# 首次构建（生成模组 jar）
./gradlew build

# 启动开发客户端（可选）
./gradlew runClient
```

构建完成后，模组 jar 位于 `build/libs/`，可放入 Forge 1.20.1 的 `mods` 目录中测试。

## 目录结构速览
```
src/main/java/org/ywzj/midi/
├── all/              # 统一注册入口（乐器、方块、实体、声音、配置）
├── audio/            # 音频播放与同步（NotePlayer、MusicPlayer、ServerSoundManager 等）
├── block(,entity)/   # 自定义方块与方块实体
├── entity/           # 假人、座椅等实体实现
├── gui/              # 各类演奏/指挥界面与控件
├── instrument/       # 乐器定义、MIDI 接收器与播放逻辑
├── network/          # SimpleChannel 与报文处理器
├── pose/             # 姿态管理、动作与 mixin
├── storage/          # MIDI/音乐文件 & 指挥配置持久化
└── util/             # 工具类（音高转换、文件上传、UI 文本等）
```

`src/main/resources/` 下包含 `mods.toml`、`mixins.ywzj_midi.json` 与资源包元数据；运行时生成的声音注册文件在构建阶段写入。

## 数据与配置
- **MIDI/音乐文件**：首次运行会在 `FMLPaths.CONFIGDIR/limitless_concert/` 下生成 `mid/` 与 `music/` 文件夹，可放置 `.mid`、`.mp3` 供服务器或客户端播放。
- **指挥配置**：`conductor_config/` 存储 `.cc` JSON，用于 `ServerMidiPlayer` 指定声部乐器、目标玩家与音量。
- **同步限制**：`common.toml` 中的 `max_sync_music_to`（默认 3）控制一次最多同步给多少名附近玩家。

## 常见开发任务
- **新增乐器**：在 `AllInstruments` 中注册 `Instrument` 实例，同时提供方块/物品与音频采样；必要时扩展 `instrument/receiver` 和 `pose` 对应逻辑。
- **扩展网络协议**：在 `Channel#onCommonSetupEvent` 注册新报文，分别实现 `handler` 与 `message`。
- **自定义姿态**：使用 `PoseManager.registerHoldPose` 或实现 `NotesHandler`，与 mixin 注入的 `HumanoidModel` 动作同步。
- **文件上传**：调用 `FileTransfer.upload` 将本地 `.mid` 推送到服务器，由 `FileTransfer.receive` 写入配置目录。

## 许可说明
仓库提供 `LICENSE.txt`（GNU GPLv3）。模组内 `mods.toml` 默认标签为 “All Rights Reserved”，如需发布请根据实际发行策略同步更新许可证信息。

## 参与贡献
欢迎提交 Issue 或 Pull Request：
1. Fork 仓库并创建特性分支。
2. 完成修改后运行 `./gradlew build` 确认通过。
3. 提交 PR 时说明变更范围、影响模块与测试步骤。

如需进一步了解模块间交互，可参考源码注释及 `src/main/java/org/ywzj/midi` 下的各子包。

祝你演奏愉快！🎵
