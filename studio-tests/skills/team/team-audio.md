# 技能测试规范：/team-audio

## 技能摘要

通过四步流水线编排音频团队：音频方向（audio-director）→ 音效设计 + 无障碍审查并行（sound-designer + accessibility-specialist）→ 技术实现 + 引擎验证并行（technical-artist + 主引擎专家）→ 代码集成（gameplay-programmer）。阶段 0 解析 `review_mode`、`automation` 和 `team.size`，技能在步骤 1 之前用一行公告当前活跃的 agent 集合——在默认 `team.size: individual` 下只生成 sound-designer，其他所有 agent 均通过它咨询。生成 agent 前读取相关 GDD、声音圣经（如存在）和现有音频资产列表。子 agent 将工作产物写入编排器指定的路径，受限于有界写入例外；编排器自行汇编音频设计文档，且仅在通过 `AskUserQuestion` 询问后才写入 `design/audio/audio-[feature].md`。在每个步骤转换处使用 `AskUserQuestion`。音频设计文档产出后判定为 COMPLETE，未解决的依赖阻断流水线时判定为 BLOCKED。未配置引擎时跳过引擎专家并记录 `Engine validation: NOT ASSESSED`。

---

## 静态断言（结构性）

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个步骤/阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"文件写入协议"章节
- [ ] 子 agent 写入仅限于有界例外——编排器指定的路径、`production/`、`docs/` 或 `tests/` 下的新产物、由 `AskUserQuestion` 门控的步骤；这些边界内无逐次写入提示，边界外需"我可以写入吗"询问；实现文件（代码、着色器、音频、资产）在此之外——每个 agent 列出其将创建或更改的文件，技能在写入任何文件前对该集合询问一次
- [ ] 为每个 agent 在 `production/audio/[feature]/` 下指定目标文件：`direction.md`（audio-director）、`sfx-spec.md`（sound-designer）、`accessibility.md`（accessibility-specialist）、`integration-plan.md`（technical-artist）、`engine-notes.md`（引擎专家）
- [ ] 将阶段门定义为流水线自身列出的决策点，无论 `automation` 模式如何，且声明阶段门不生成 director
- [ ] 汇编的音频设计文档由编排器仅在通过 `AskUserQuestion` 询问"我可以将音频设计写入 `design/audio/audio-[feature].md` 吗？"之后写入（`design/` 在有界例外之外）
- [ ] 每个 agent 提示以返回契约结尾：写入的路径、≤5 条要点摘要、BLOCKED/CONCERNS 项各一行
- [ ] 末尾有下一步交接（引用 `/dev-story`、`/asset-audit`）
- [ ] 存在错误恢复协议章节：在将步骤视为完成前验证指定产物在磁盘上，立即暴露 BLOCKED，始终产出部分报告，完整流程见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在继续前于步骤转换处使用 `AskUserQuestion`（`collaborative` 模式）
- [ ] 阶段 0 解析 `review_mode`、`automation` 和 `team.size`，并要求在步骤 1 前有一行活跃集合公告
- [ ] 步骤 2 明确并行生成 sound-designer 和 accessibility-specialist，且"如何委派"列出 `accessibility-specialist`
- [ ] 步骤 2 将无视觉提示或字幕的游戏关键音频事件标记为 BLOCKING，步骤 3 等待用户解决或接受
- [ ] 步骤 3 明确并行生成 technical-artist 和引擎专家（当引擎已配置时）
- [ ] 技能在上下文收集期间读取 `design/audio/sound-bible.md` 处的声音圣经（如存在）；仍位于 `design/gdd/sound-bible.md`（较早位置）的声音圣经会被读取，并建议移动
- [ ] 输出文档保存至 `design/audio/audio-[feature].md`

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有步骤完成，音频设计文档已保存

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- `project.yaml` 中 `engine.name` 为 Godot
- 目标功能的 GDD 存在于 `design/gdd/combat.md`
- 声音圣经存在于 `design/audio/sound-bible.md`
- 现有音频资产列于 `assets/audio/`
- 计划的音频事件列表中无无障碍缺口

**输入：** `/team-audio combat`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: studio` 的活跃集合
2. 上下文收集：编排器在生成任何 agent 前读取 `design/gdd/combat.md`、`design/audio/sound-bible.md` 和 `assets/audio/` 资产列表
3. 步骤 1：生成 audio-director；定义战斗的音色身份、情感基调、自适应音乐方向、混音目标和自适应音频规则
4. `AskUserQuestion` 呈现音频方向；用户批准后步骤 2 开始
5. 步骤 2：并行生成 sound-designer 和 accessibility-specialist；sound-designer 产出音效规格、含触发条件的音频事件列表和混音组；accessibility-specialist 识别关键游戏音频事件并指定视觉回退和字幕要求
6. `AskUserQuestion` 呈现音效规格和无障碍要求；用户批准后步骤 3 开始
7. 步骤 3：并行生成 technical-artist 和 `godot-specialist`（由 `engine.name` 推导）；technical-artist 设计总线结构、中间件集成、内存预算和流式策略；引擎专家验证集成方法对固定引擎是否地道
8. `AskUserQuestion` 呈现技术方案；用户批准后步骤 4 开始
9. 步骤 4：生成 gameplay-programmer；首先返回其将创建或更改的代码和测试文件，编排器对该集合询问一次，仅在同意后将音频事件接入游戏玩法触发器、实现自适应音乐、设置遮挡区域并编写音频事件触发器的单元测试
10. 对每个步骤，编排器在将步骤视为完成前检查 agent 返回契约中指定的路径是否存在
11. 编排器自行将所有输出汇编为单一音频设计文档（不重新生成子 agent 来编写）
12. 编排器通过 `AskUserQuestion` 询问："我可以将音频设计写入 `design/audio/audio-combat.md` 吗？"并仅在批准后写入
13. 摘要输出列出：音频事件数、估计资产数、实现任务和任何未决问题
14. 判定：COMPLETE

**断言：**
- [ ] 在生成第一个 agent 前出现命名 `team.size: studio` 的活跃集合行
- [ ] 声音圣经存在时在上下文收集期间（步骤 1 前）被读取
- [ ] audio-director 在 sound-designer 或 accessibility-specialist 之前生成
- [ ] `AskUserQuestion` 出现在步骤 1 输出之后、步骤 2 启动之前
- [ ] 步骤 2 中 sound-designer 和 accessibility-specialist 的 Agent 调用同时发出
- [ ] 步骤 3 中 technical-artist 和 `godot-specialist` 的 Agent 调用同时发出
- [ ] 步骤 3 的 `AskUserQuestion` 批准前不启动 gameplay-programmer
- [ ] 在对其实现集合的一次询问得到同意前，gameplay-programmer 不写入任何代码或测试文件
- [ ] 音频设计文档写入 `design/audio/audio-combat.md`（非其他路径），由编排器写入，且仅在其 `AskUserQuestion` 批准后
- [ ] 摘要包含音频事件数和估计资产数
- [ ] 文档交付后判定为 COMPLETE

---

### 用例 2：无障碍缺口——关键游戏音频事件无视觉回退

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- 目标功能的 GDD 存在
- 步骤 1 和步骤 2 进行中
- sound-designer 的音频事件列表包含"EnemyNearbyAlert"——一种空间音频提示，警告玩家有敌人从屏幕外接近
- accessibility-specialist 审查事件列表，发现"EnemyNearbyAlert"无视觉回退（未指定屏幕指示器、字幕或手柄震动）

**输入：** `/team-audio stealth`（步骤 2 场景）

**预期行为：**
1. 步骤 1–2 进行；并行生成 accessibility-specialist 和 sound-designer
2. accessibility-specialist 返回审查，含 BLOCKING 关注项："`EnemyNearbyAlert` 是关键游戏音频事件（警告玩家屏幕外威胁），无视觉回退——听障玩家无法察觉此威胁。"
3. 编排器在呈现 `AskUserQuestion` 前在对话中写入该关注项
4. `AskUserQuestion` 将无障碍关注项作为 BLOCKING 问题呈现，选项如：
   - 为 EnemyNearbyAlert 添加视觉指示器（如 HUD 上的方向箭头）并继续
   - 添加手柄触觉反馈作为回退并继续
   - 在此停止，在进入步骤 3 前解决所有无障碍缺口
5. 在用户解决或明确接受缺口前，不启动步骤 3（technical-artist + 引擎专家）
6. 若用户停止：产出步骤 1–2 输出的部分报告，判定为 `BLOCKED — [原因]`，指明未解决的 EnemyNearbyAlert 缺口
7. 若用户接受缺口并继续：缺口列于最终摘要的未决问题中

**断言：**
- [ ] 无障碍缺口在报告中标记为 BLOCKING（非建议性）
- [ ] 陈述具体事件名（"EnemyNearbyAlert"）和缺口性质
- [ ] `AskUserQuestion` 在启动步骤 3 前呈现缺口
- [ ] 至少提供一个解决选项（添加视觉回退、添加触觉回退）
- [ ] 缺口未解决且无用户明确授权时不启动步骤 3
- [ ] 若用户停止，判定为 `BLOCKED — [原因]` 并指明缺口，步骤 1–2 输出保留在部分报告中；若用户继续推进，最终摘要将其列为未决问题

---

### 用例 3：无参数——用法指引或设计文档推断

**夹具：**
- 任意项目状态

**输入：** `/team-audio`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法指引：如"用法：`/team-audio [要设计音频的功能或区域] [--review full|lean|solo]`——指定要设计音频的功能或区域（如 `combat`、`main menu`、`forest biome`、`boss encounter`）"
3. 技能退出，不生成任何 agent

**断言：**
- [ ] 未提供参数时技能不生成任何 agent
- [ ] 用法消息显示完整 argument-hint——`/team-audio [要设计音频的功能或区域] [--review full|lean|solo]`——及参数示例
- [ ] 技能不尝试在无用户指示下从现有设计文档推断功能
- [ ] 不使用 `AskUserQuestion`——输出为直接指引

---

### 用例 4：缺少声音圣经——技能记录缺口并在无它情况下继续

**夹具：**
- 解析的配置块：`team.size: small`、`automation: collaborative`
- 目标功能的 GDD 存在于 `design/gdd/main-menu.md`
- `design/audio/sound-bible.md` 或 `design/gdd/sound-bible.md` 处均无声音圣经
- 引擎已配置；其他上下文文件存在

**输入：** `/team-audio main menu`

**预期行为：**
1. 上下文收集：编排器读取 `design/gdd/main-menu.md` 并检查 `design/audio/sound-bible.md`
2. 未找到声音圣经；编排器用一行记录缺口："`design/audio/sound-bible.md` 处无声音圣经；音频方向仅从 GDD 开始"
3. 流水线在无声音圣经作为输入的情况下正常通过全部四步
4. 步骤 1 的 audio-director 被告知无声音圣经，必须仅从功能 GDD 建立音色身份
5. 下一步建议创建声音圣经并说明由谁编写：`audio-director` agent 从 `.claude/docs/templates/sound-bible.md` 起草 `design/audio/sound-bible.md`，从本次运行的步骤 1 方向开始，并在写入前询问

**断言：**
- [ ] 编排器在上下文收集期间（步骤 1 前）检查声音圣经
- [ ] 缺失的声音圣经在对话中明确记录——非静默忽略
- [ ] 流水线不因缺少声音圣经而停止
- [ ] audio-director 在其提示上下文中被告知无声音圣经
- [ ] 下一步建议创建声音圣经并指明作者——`audio-director`，从 `.claude/docs/templates/sound-bible.md` 起草
- [ ] 技能在本次运行中不自行写入 `design/audio/sound-bible.md`
- [ ] 若所有其他步骤成功，判定仍为 COMPLETE

---

### 用例 5：引擎未配置——引擎专家步骤跳过并记录

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- `project.yaml` 中 `engine.name` 未设置，且 `.claude/docs/technical-preferences.md` 无主引擎专家（显示 `[TO BE CONFIGURED]`）
- 目标功能的 GDD 存在
- 声音圣经可能存在或不存在

**输入：** `/team-audio boss encounter`

**预期行为：**
1. 步骤 1–2 正常进行（audio-director、sound-designer、accessibility-specialist）
2. 步骤 3：technical-artist 正常生成；引擎专家生成被跳过
3. 运行输出记录：`Engine validation: NOT ASSESSED — no engine configured (engine.name unset in project.yaml)`
4. 步骤 4：gameplay-programmer 仍运行——缺失引擎不被视为阻断
5. 判定：`COMPLETE — engine validation NOT ASSESSED (no engine configured)`——跳过是优雅的，非阻断，且绝非纯 COMPLETE

**断言：**
- [ ] 未配置引擎时不生成引擎专家
- [ ] 技能不因缺少引擎配置而报错
- [ ] 运行输出包含 `Engine validation: NOT ASSESSED — no engine configured` 及 `engine.name` 原因——跳过非静默省略
- [ ] 本次运行中引擎集成绝不被描述为已验证或通过
- [ ] 步骤 3 仍生成 technical-artist（跳过仅适用于引擎专家）
- [ ] 步骤 4 仍运行 gameplay-programmer
- [ ] 判定为 `COMPLETE — engine validation NOT ASSESSED (…)`，绝非纯 COMPLETE（引擎未配置是优雅情况，非阻断）

---

### 用例 6：`review_mode` 不门控任何内容——full、lean 和 solo 下的相同请求

**夹具：**
- `project.yaml`：`modes.review_mode: lean`（固定：以下运行覆盖或保留它）、`team.size: studio`（固定：以便流水线中每个步骤和 agent 均活跃）
- `engine.name` 为 Godot；`design/gdd/combat.md` 和 `design/audio/sound-bible.md` 存在；事件列表中无无障碍缺口
- 运行 A：`/team-audio combat --review full`。运行 B：`/team-audio combat`（解析为 `lean`）。运行 C：`/team-audio combat --review solo`

**预期行为：**
1. 三次运行均以 `team.size: studio` 的相同活跃集合行开始
2. 三次运行均以相同 agent 执行相同四步：audio-director，然后 sound-designer + accessibility-specialist，然后 technical-artist + `godot-specialist`，然后 gameplay-programmer。这些是执行步骤的团队成员，非 director 门
3. 任何运行在任何步骤均不生成 CD-、TD-、PR- 或 AD-PHASE-GATE："任何 `review_mode` 下，以下阶段均不生成 CD-、TD-、PR- 或 AD-PHASE-GATE"
4. 每个步骤转换处的 `AskUserQuestion`、音频设计文档的单次询问和判定在三次运行中均相同

**断言：**
- [ ] 运行 A（`full`）、运行 B（`lean`）或运行 C（`solo`）中均不生成 director 门（CD-、TD-、PR- 或 AD-PHASE-GATE）
- [ ] 活跃集合、每步生成的 agent 和步骤转换询问次数在三次运行中完全相同
- [ ] `--review full` 不增加审查步骤，`--review solo` 不删除步骤：仅 `team.size` 改变生成谁
- [ ] 每次运行仅在其自身询问后写入 `design/audio/audio-combat.md` 并以 COMPLETE 结束

---

## 协议合规性

- [ ] 在默认 `team.size: individual` 下，运行以 `Active set (team.size: individual): sound-designer.` 和一行 `Not spawned this run:` 开始，命名 audio-director、accessibility-specialist、technical-artist、引擎专家和 gameplay-programmer，通过 sound-designer