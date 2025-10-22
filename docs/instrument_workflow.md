# Instruments & Animation Guide

*修改 By Lynn*

本说明整理了当前模组内置的所有乐器，并总结了新增或调整乐器时需要关注的代码、资源与演奏动作流程。

## 现有乐器一览

| 名称 | 类型 | 类别 | 备注 |
| ------ | ------ | ------ | ------ |
| U1H | 方块 | `instrument/UprightPiano.java` | 立式钢琴，音域 `a1-c8` |
| CFX | 方块 | `instrument/GrandPiano.java` | 三角钢琴，音域 `a1-c8` |
| Violin | 物品 | `instrument/Violin.java` | 便携，含 pizzicato 变奏 |
| Viola | 物品 | `instrument/Viola.java` | 便携，含 pizzicato 变奏 |
| Cello | 物品 | `instrument/Cello.java` | 便携，含 pizzicato 变奏 |
| Double Bass | 物品 | `instrument/DoubleBass.java` | 便携，含 pizzicato 变奏 |
| Oboe | 物品 | `instrument/Oboe.java` | 便携 |
| Clarinet | 物品 | `instrument/Clarinet.java` | 便携 |
| Flute | 物品 | `instrument/Flute.java` | 便携 |
| Bassoon | 物品 | `instrument/Bassoon.java` | 便携 |
| Horn | 物品 | `instrument/Horn.java` | 便携 |
| Trumpet | 物品 | `instrument/Trumpet.java` | 便携 |
| Trombone | 物品 | `instrument/Trombone.java` | 便携，配套滑管动画 |
| Tuba | 物品 | `instrument/Tuba.java` | 便携 |
| Timpani | 方块 | `instrument/Timpani.java` | 包含敲击动画 |
| Bass Drum | 方块 | `instrument/BassDrum.java` | 包含敲击动画 |
| Cymbal | 物品 | `instrument/Cymbal.java` | 快速敲击动画 |
| AA775 | 方块 | `instrument/AA775.java` | 特殊键盘乐器 |

来源：`src/main/java/org/ywzj/midi/all/AllInstruments.java`

## 新增乐器步骤

1. **定义乐器类**  
   - 在 `src/main/java/org/ywzj/midi/instrument/` 下创建类，继承 `Instrument` 并实现 `receiver(...)`（返回适合的 `MidiReceiver`）。  
   - 通过构造函数设置是否循环、是否便携、音域范围；需要变奏时调用 `extra(...)`。

2. **注册乐器**  
   - 在 `AllInstruments.registerInstrument(...)` 中添加条目，选择 `Type.ITEM` 或 `Type.BLOCK`。  
   - 对应方块/物品的具体实现（例如自定义 `Block`、`Item`）放在 `block/` 或 `item/` 包并在注册时引用。  
   - 注册流程会自动向 `AllBlocks` / `AllItems` / `AllSounds` 填充信息，并把乐器纳入创造模式标签。

3. **准备声音资源**  
   - 在资源包内为每个音符提供 `.ogg` 文件，命名遵循 `乐器名_音高[_变奏].ogg`，例如 `violin_c4.ogg`、`violin_c4_pizz.ogg`。  
   - 更新 `assets/<namespace>/sounds.json`，与 `AllSounds.registerKeys` 自动生成的 `SoundEvent` 名称保持一致。  
   - 运行 `./gradlew build` 以验证采样是否被正确打包；缺失时会在日志中看到 `Unknown sound sample`。

4. **（可选）自定义 GUI / 逻辑**  
   如果乐器需要专属界面或交互，可参考 `gui/screen` 与 `instrument/player` 包下的现有实现。

## 调整音色或性能

- 替换现有 `.ogg` 文件即可更新音色，保持文件名不变可避免额外改动。  
- 需调整音域或循环属性时，在对应 `Instrument` 构造参数中修改，并确保提供完整的音频样本。  
- 若想改变默认音量或播放逻辑，可修改对应 `MidiReceiver` 实现或 `NotePlayer` 的调用方式。

## 演奏动作与动画

演奏姿态通过 `PoseManager` 与 mixin 注入实现：

1. **基础持姿 (`AllHoldPose`)**  
   - 在 `src/main/java/org/ywzj/midi/all/AllHoldPose.java` 注册手持姿势，指定乐器、主副手、`PoseManager.PlayPose` 参数。  
   - `HumanoidModelMixin` 会在渲染时自动应用。

2. **动态演奏动作 (`pose.action` & `pose.handler`)**  
   - 在 `pose/action` 包中创建动作类，生成一组 `PoseManager.PlayPose` 序列或根据实时音符计算姿态。  
   - 如需响应音符事件，在 `pose/handler` 中实现 `NotesHandler` 并通过 `AllNotesHandler` 注册，或在乐器 GUI / MIDI 播放流程中手动调用 `PoseManager.publish(...)`。
   - 特殊模型行为（例如长号滑管）可结合 `ItemProperties` 自定义渲染参数。

3. **假人模型**  
   - `FakePlayerEntity` 使用 `render/model/FakePlayerModel.java` 控制坐姿与手臂约束，若添加新的静态姿势可参考该类。

完成以上步骤后，通过 `./gradlew runClient` 验证乐器在客户端的持姿、演奏动作与音色是否符合预期。任何姿态问题可在调试时观察日志或在 `PoseManager` 中添加额外输出。***
