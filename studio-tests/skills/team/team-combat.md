# 技能测试规范：/team-combat

## 技能摘要

为单个战斗功能端到端编排完整的战斗团队流水线。
通过六个结构化阶段协调 game-designer、gameplay-programmer、ai-programmer、technical-artist、
sound-designer、主引擎专家和 qa-tester：设计 → 架构（含引擎专家验证）→ 实现
（并行）→ 集成 → 验证 → 签核。阶段 0 解析 `review_mode`、
`automation`、`team.size` 和 `workflow`（后者决定 GDD 的规模）；完整流水线在 `team.size: small` 及以上运行，
而默认的 `individual` 仅运行 gameplay-programmer（仅当标记了 AI 工作时才运行 ai-programmer），
并在阶段 1 之前宣布这一收缩。在每个阶段转换处使用
`AskUserQuestion`，并在实现之前设有显式的 [A]/[B]/[C] 架构门禁。每个 agent 都写入技能
路径表中指定的目标位置；编排器唯一的写入是 GDD，即 `design/gdd/[feature].md`，
来自 game-designer 的草稿，经"May I write"批准后写入。产出摘要报告，
结论为 COMPLETE / NEEDS WORK / BLOCKED，并交接至 `/code-review`、
`/balance-check` 和 `/team-polish`。

---

## 静态断言（结构性）

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题（阶段 1 至阶段 6 均存在）
- [ ] 包含结论关键词：COMPLETE、NEEDS WORK、BLOCKED
- [ ] 包含"File Write Protocol"——子 agent 草稿和 QA 产物保留在有界例外之内，实现文件（代码根目录、`assets/`）在其之外，作为一组一次性请求；编排器唯一的写入是 `design/gdd/[feature].md`，在其自身的"May I write"询问之后，因为 `design/` 在有界例外之外
- [ ] 为每个阶段指定具体目标位置：`production/combat/[feature]-gdd-draft.md`（game-designer 对 `design/gdd/[feature].md` 的草稿）、`docs/architecture/[feature]-sketch.md`、`docs/architecture/[feature]-engine-notes.md`、`production/qa/test-cases/[feature]-cases.md`、`production/qa/bugs/BUG-[NNNN].md`
- [ ] 将阶段门禁定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门禁处声称生成 director
- [ ] 末尾有下一步交接（引用 `/code-review`、`/balance-check`、`/team-polish`）
- [ ] 存在错误恢复协议章节：在将阶段视为完成之前，验证指定产物已在磁盘上，立即暴露 BLOCKED，始终产出部分报告，完整流程（暴露 → 评估 → 选项 → 部分报告）见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在阶段转换处使用 `AskUserQuestion` 以在继续之前获得用户批准（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，且技能要求在阶段 1 之前进行一行活跃集公告
- [ ] 阶段 0 解析 `workflow`，阶段 1 据此确定 GDD 规模：`full` 时为 `## Summary` 加全部 8 个章节，`standard` 时为 5 个标准章节加数值规则的 Formulas，`minimal` 时为可选（游戏简报即为记录）
- [ ] 阶段 3 明确标记为并行（gameplay-programmer、ai-programmer、technical-artist、sound-designer）
- [ ] 阶段 2 包含生成主引擎专家，从 `project.yaml` 中的 `engine.name` 推导（Godot → `godot-specialist`），回退到 `.claude/docs/technical-preferences.md` 中的 Primary 行
- [ ] 团队构成列出全部七个角色（game-designer、gameplay-programmer、ai-programmer、technical-artist、sound-designer、引擎专家、qa-tester）

---

## 夹具默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和
`modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身的
Fixture 另有说明。规则：`studio-tests/CLAUDE.md` §
Fixture defaults。

---

## 测试用例

### 用例 1：正常路径——所有 agent 成功，完整流水线运行至完成

**Fixture：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`、`workflow: full`
- `design/gdd/game-concept.md` 存在且已填充
- `project.yaml` 中的 `engine.name` 为 Godot
- 所请求的战斗功能没有现有 GDD
- 该功能涉及 NPC 反应（敌人招架和反击），因此标记了 AI 工作

**输入：** `/team-combat parry and riposte system`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: small` 的活跃集（完整团队构成流水线）
2. 阶段 1——生成 game-designer；将设计草稿写入 `production/combat/parry-riposte-gdd-draft.md`，覆盖全部 8 个必需章节（概述、玩家幻想、规则、公式、边界情况、依赖、调参旋钮、验收标准）；`AskUserQuestion` 呈现草稿并询问"May I write the design document to `design/gdd/parry-riposte.md`?"；编排器在批准后写入
3. 阶段 2——生成 gameplay-programmer + ai-programmer；产出架构草图，包含类结构、接口和文件列表（`docs/architecture/parry-riposte-sketch.md`）；然后生成 `godot-specialist` 以验证惯用法（`docs/architecture/parry-riposte-engine-notes.md`）；引擎专家输出被纳入
4. `AskUserQuestion` "Architecture sketch complete. Approve to proceed with parallel implementation."，选项为 [A] Proceed / [B] Revise the architecture first / [C] Stop here——实现 agent 仅在 [A] 时生成
5. 阶段 3——并行生成 gameplay-programmer、ai-programmer、technical-artist、sound-designer；三个实现 agent 返回它们将创建或更改的文件，一个 `AskUserQuestion` 请求整个集合，它们仅在同意后写入；sound-designer 将其事件列表草稿写入 `production/combat/parry-riposte-audio-events.md`；四者全部返回后阶段 4 才开始
6. 阶段 4——集成将阶段 3 的所有输出连接起来；调参旋钮验证为数据驱动；`AskUserQuestion` 在阶段 5 之前确认集成
7. 阶段 5——生成 qa-tester；根据验收标准将测试用例写入 `production/qa/test-cases/parry-riposte-cases.md`；验证边界情况；对照预算检查性能影响
8. 阶段 6——产出摘要报告：设计状态、每位团队成员的实现状态、测试结果、未决问题；结论：COMPLETE
9. 列出下一步：`/code-review`、`/balance-check`、`/team-polish`

**断言：**
- [ ] 在生成第一个 agent 之前出现命名 `team.size: small` 的活跃集行
- [ ] 在每个阶段转换处调用 `AskUserQuestion`（至少在阶段 3 之前和阶段 5 之前）
- [ ] 阶段 3 之前的门禁提供 [A] Proceed / [B] Revise / [C] Stop，且除非选择 [A]，否则不生成任何实现 agent
- [ ] 阶段 3 的 agent 同时启动——gameplay-programmer、ai-programmer、technical-artist、sound-designer 之间无顺序依赖
- [ ] 引擎专家在阶段 2 运行，早于阶段 3 开始（输出被纳入架构）
- [ ] 每个 agent 提示都指定其路径表中的目标位置；没有子 agent 写入 `design/` 下
- [ ] 在覆盖整个阶段 3 集合的一次询问之前，不写入任何实现文件（代码根目录、`assets/`）
- [ ] 在阶段 1 的"May I write"批准之前，`design/` 下不写入任何内容，且编排器唯一的 Write 是 `design/gdd/parry-riposte.md`
- [ ] 最终报告中存在结论 COMPLETE
- [ ] 下一步包括 `/code-review`、`/balance-check`、`/team-polish`
- [ ] 设计文档覆盖全部 8 个必需 GDD 章节

---

### 用例 2：Agent 受阻——一个子 agent 在流水线中途返回 BLOCKED

**Fixture：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `design/gdd/parry-riposte.md` 存在（阶段 1 已完成）
- ai-programmer agent 返回 BLOCKED，因为不存在 AI 系统架构 ADR（ADR 状态为 Proposed）

**输入：** `/team-combat parry and riposte system`

**预期行为：**
1. 阶段 1——找到设计文档；game-designer 确认其有效；阶段获批
2. 阶段 2——gameplay-programmer 完成架构草图；ai-programmer 返回 BLOCKED："ADR for AI behavior system is Proposed — cannot implement until ADR is Accepted"
3. 触发错误恢复协议："ai-programmer: BLOCKED — AI behavior ADR is Proposed"
4. 呈现 `AskUserQuestion`，选项为：(a) 跳过 ai-programmer 并记录缺口；(b) 以更窄范围重试；(c) 在此停止并先运行 `/architecture-decision`
5. 如果用户选择 (a)：阶段 3 仅以 gameplay-programmer、technical-artist、sound-designer 继续；ai-programmer 缺口记录在部分报告中
6. 产出最终报告：记录部分实现，ai-programmer 部分标记为 BLOCKED，总体结论：BLOCKED

**断言：**
- [ ] 在任何依赖阶段继续之前出现 BLOCKED 暴露消息
- [ ] `AskUserQuestion` 至少提供三个选项：跳过 / 重试 / 停止
- [ ] 产出部分报告——已完成 agent 的工作不被丢弃
- [ ] 当任何 agent 未解决时，总体结论为 BLOCKED（而非 COMPLETE）
- [ ] 受阻原因引用 ADR 并建议 `/architecture-decision`
- [ ] 编排器不会静默越过受阻依赖继续

---

### 用例 3：无参数——显示清晰的用法指引

**Fixture：**
- 任意项目状态

**输入：** `/team-combat`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法消息，说明必需参数（战斗功能描述）
3. 显示调用格式 `/team-combat [combat feature description] [--review full|lean|solo]` 及示例（`melee parry system`、`ranged weapon spread`）
4. 技能退出，不生成任何子 agent

**断言：**
- [ ] 未提供参数时，技能不生成任何子 agent
- [ ] 用法消息显示完整的 argument-hint：`/team-combat [combat feature description] [--review full|lean|solo]`
- [ ] 错误消息包含至少一个有效调用示例
- [ ] 除检测缺失参数所需之外，不读取任何文件
- [ ] 不显示结论（流水线从不运行）

---

### 用例 4：并行阶段验证——阶段 3 的 agent 同时运行

**Fixture：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `design/gdd/parry-riposte.md` 存在、完整，并标记了敌人 AI 反应
- 架构草图已以选项 [A] 批准
- 引擎专家已验证架构

**输入：** `/team-combat parry and riposte system`（从阶段 2 完成处恢复）

**预期行为：**
1. 架构批准后阶段 3 开始
2. 全部四个 Agent 调用——gameplay-programmer、ai-programmer、technical-artist、sound-designer——在任何结果被等待之前发出
3. 技能等待全部四个 agent 完成后才进入阶段 4
4. 如果任何单个 agent 提前完成，技能不会在全部四个返回之前开始阶段 4

**断言：**
- [ ] 四个 Agent 调用在单一批次中发出（它们之间无顺序等待）
- [ ] 在全部四个阶段 3 agent 返回结果之前，阶段 4 不开始
- [ ] 技能不会将一个阶段 3 agent 的输出作为另一个阶段 3 agent 的输入（它们相互独立）
- [ ] 阶段 4 集成步骤中引用了全部四个阶段 3 agent 的结果

---

### 用例 5：架构阶段引擎路由——从配置解析引擎专家

**Fixture（变体 A——已配置引擎）：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `project.yaml` 中的 `engine.name` 为 Godot；引擎版本固定于 `docs/engine-reference/godot/VERSION.md`
- gameplay-programmer 产出的架构草图可用，且编排器已读取

**Fixture（变体 B——未配置引擎）：**
- 同变体 A，但 `project.yaml` 中 `engine.name` 未设置，且 `.claude/docs/technical-preferences.md` 没有 Primary 引擎专家（显示 `[TO BE CONFIGURED]`）

**输入：** `/team-combat parry and riposte system`

**预期行为（变体 A）：**
1. 阶段 2——gameplay-programmer 产出架构草图
2. 技能从 `engine.name` 推导主引擎专家（Godot → `godot-specialist`）；仅当 `engine.name` 缺失时才查阅 `technical-preferences.md`
3. 以架构草图的精简简报（而非编排器已读取的文档路径）生成引擎专家，并询问阶段 2 的三个问题：惯用的类/节点结构、应优先使用的引擎原生系统、固定引擎版本中已弃用或已更改的 API
4. 引擎专家将其笔记写入 `docs/architecture/parry-riposte-engine-notes.md`；编排器在将该步骤视为完成之前检查文件存在
5. 编排器在向用户呈现阶段 2 结果之前将引擎笔记纳入架构
6. `AskUserQuestion` 架构门禁在架构草图旁包含引擎专家的笔记

**预期行为（变体 B）：**
1. 阶段 2——gameplay-programmer 产出架构草图
2. 不生成引擎专家，也不预期引擎笔记文件
3. 运行输出记录 `Engine validation: NOT ASSESSED — no engine configured (engine.name unset in project.yaml)`
4. 架构门禁仍然运行；它呈现草图，但不将其描述为经引擎验证
5. 运行完成时，结论为 `COMPLETE — engine validation NOT ASSESSED (no engine configured)`

**断言：**
- [ ] 引擎专家 agent 类型从 `engine.name` 推导（回退到 `technical-preferences.md`）——而非硬编码（变体 A）
- [ ] 由于编排器已读取草图，引擎专家的简报内联携带相关草图内容，而非草图路径（变体 A）
- [ ] 引擎专家对照固定引擎版本检查已弃用或已更改的 API（变体 A）
- [ ] 引擎专家输出在阶段 3 开始之前被纳入（而非跳过或单独附加）（变体 A）
- [ ] 不生成引擎专家，且运行输出记录 `Engine validation: NOT ASSESSED — no engine configured`——跳过从不静默（变体 B）
- [ ] 当没有专家运行时，架构从不被描述为经引擎验证（变体 B）
- [ ] 运行完成时，其结论读作 `COMPLETE — engine validation NOT ASSESSED ([reason])`——从不是单纯的 COMPLETE（变体 B）

---

## 协议合规

- [ ] 在默认 `team.size: individual` 下，运行以 `Active set (team.size: individual): gameplay-programmer` 行和 `Not spawned this run:` 行开头，命名 game-designer、technical-artist、sound-designer、引擎专家和 qa-tester（除非标记了 AI 工作，否则也包括 ai-programmer）——在任何 agent 生成之前
- [ ] 在 `collaborative` 模式下，在每个阶段转换处使用 `AskUserQuestion`——用户在流水线推进之前批准
- [ ] 子 agent 写入通过 Agent 委托——草稿和 QA 产物在有界例外之内，实现文件仅在为整个集合的一次询问之后；编排器仅写入 `design/gdd/[feature].md`，在"May I write"批准之后
- [ ] 遵循错误恢复协议：暴露 → 评估 → 提供选项 → 部分报告
- [ ] 指定产物在磁盘上缺失的阶段被视为失败，而非完成
- [ ] 阶段 3 的 agent 按技能规范并行启动
- [ ] 即使 agent 受阻也始终产出部分报告
- [ ] 结论为 COMPLETE / NEEDS WORK / BLOCKED 之一
- [ ] 输出末尾存在下一步：`/code-review`、`/balance-check`、`/team-polish`
- [ ] 在 `team.size: studio` 下，主引擎专家的提示允许其子专家，阶段 5 的 qa-tester 被要求找出功能如何失效而非确认其有效，且报告说明对抗性检查已运行；低于 `studio` 时两者均不出现

---

## 覆盖说明

- NEEDS WORK 结论路径（qa-tester 在阶段 5 发现失败）在此未单独测试；
  它遵循与用例 2 相同的错误恢复和部分报告协议。
- "以更窄范围重试"错误恢复选项在断言中列出，但其完整的