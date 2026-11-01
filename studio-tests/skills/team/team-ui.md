# 技能测试规范：/team-ui

## 技能摘要

为单个 UI 功能编排 UI 团队走完整个 UX 流水线。
协调 ux-designer、ui-programmer、art-director、引擎 UI 专家
和 accessibility-specialist，经过五个结构化阶段：上下文收集 +
UX 规范（阶段 1a/1b）→ UX 评审门（阶段 1c）→ 视觉设计（阶段 2）→
实现（阶段 3）→ 并行评审（阶段 4）→ 打磨（阶段 5）。
阶段 0 解析 `review_mode`、`automation` 和 `team.size`，技能在任何 spawn 之前
公布当前生效的集合——默认 `individual` 启动 ui-programmer +
ux-designer，`small` 增加 accessibility-specialist + art-director，`studio` 增加
引擎 UI 专家和对抗式阶段 4 评审。阶段 1a 将五个输入逐一报告为存在或缺失，
缺失的 `design/accessibility-requirements.md` 会以
`Accessibility: NOT ASSESSED` 继续传递。在每个阶段转换处使用 `AskUserQuestion`。
将所有文件写入委托给子代理和子技能（`/ux-design`、
`ui-programmer`）。产出总结报告，结论为 COMPLETE / BLOCKED，并
交接至 `/ux-review`、`/code-review`、`/team-polish`。

---

## 静态断言（结构性）

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题（阶段 1a 至阶段 5 均存在）
- [ ] 包含结论关键词：COMPLETE、BLOCKED
- [ ] 文件写入协议：子代理在有界例外下写入（指定路径、`production/`、`docs/` 或 `tests/` 下的新构建产物、受门控的阶段），例外之外需询问；子技能（`/ux-design`）写入前需询问；编排器不直接写文件；实现文件（代码、着色器、音频、资源）在例外之外——每个代理列出它将创建或更改的文件，技能在写入任何文件之前对该集合询问一次
- [ ] 为每个代理指定 `production/ui/[feature-name]/` 下的目标文件：`visual-design.md`（art-director，阶段 2）、`engine-notes.md`（引擎 UI 专家）、`review-[agent].md`（每个阶段 4 评审者各一个文件）
- [ ] 将阶段门定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门处声称 spawn 总监
- [ ] 末尾有下一步交接（引用 `/ux-review`、`/code-review`、`/team-polish`）
- [ ] 存在错误恢复协议章节：验证指定构建产物在磁盘上，立即上报 BLOCKED，始终产出部分报告，完整流程（上报 → 评估 → 选项 → 部分报告）见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在阶段转换处使用 `AskUserQuestion` 获取用户批准后再继续（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，技能要求在任何 spawn 之前做一行生效集合公告
- [ ] 阶段 4 明确标记为并行（ux-designer、art-director、accessibility-specialist）
- [ ] UX 评审门（阶段 1c）定义为阻塞门——技能不得在未获 APPROVED 结论的情况下进入阶段 2，除非用户明确覆盖 NEEDS REVISION 或 NOT ASSESSED，最终报告记录该覆盖
- [ ] 团队构成列出全部五个角色（ux-designer、ui-programmer、art-director、引擎 UI 专家、accessibility-specialist）
- [ ] 引擎 UI 专家从 `project.yaml` 的 `specialists.ui` 读取（回退到 `.claude/docs/technical-preferences.md`），`specialists.ui: null` 视为未设置
- [ ] 引用交互模式库（`design/ux/interaction-patterns.md`）——ui-programmer 必须使用现有模式
- [ ] 阶段 1a 在设计开始前读取 `design/accessibility-requirements.md`

---

## 夹具默认值

每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和
`modes.automation: collaborative`，且无 `project.local.yaml`，除非其自身的
夹具另有说明。规则：`studio-tests/CLAUDE.md` §
夹具默认值。

---

## 测试用例

### 用例 1：正常路径——从 UX 规范到打磨的完整流水线成功

**夹具：**
- 解析后的配置块：`review_mode: full`、`team.size: studio`、`automation: collaborative`
- `design/gdd/game-concept.md` 存在，含平台目标和目标受众
- `design/player-journey.md` 存在
- `design/ux/interaction-patterns.md` 存在，含相关模式
- `design/accessibility-requirements.md` 存在，含已承诺等级（例如 Standard）
- `project.yaml` 中 `engine.name` 为 Godot，`specialists.ui` 为 `godot-specialist`

**输入：** `/team-ui inventory screen`

**预期行为：**
1. 在任何 spawn 之前，一行公告 `team.size: studio` 的生效集合
2. 阶段 1a——编排器读取 game-concept.md、player-journey.md、相关 GDD UI 章节、interaction-patterns.md、accessibility-requirements.md；将每个输入报告为存在或缺失；为 ux-designer 总结一份简报
3. 阶段 1b——调用 `/ux-design inventory-screen`（或直接 spawn ux-designer）；使用 `ux-spec.md` 模板产出 `design/ux/inventory-screen.md`；`AskUserQuestion` 在评审前确认规范
4. 阶段 1c——调用 `/ux-review design/ux/inventory-screen.md`；返回 APPROVED；门通过，进入阶段 2
5. 阶段 2——spawn art-director；评审完整 UX 规范（不仅是线框图）；应用视觉处理；验证颜色对比度；产出含资源清单的视觉设计规范；`AskUserQuestion` 在阶段 3 前确认
6. 阶段 3——先 spawn `godot-specialist`（来自 `specialists.ui`）；为 ui-programmer 产出实现说明；spawn ui-programmer，附带 UX 规范 + 视觉规范 + 引擎说明；产出实现；若引入新模式则更新 interaction-patterns.md
7. 阶段 4——并行 spawn ux-designer、art-director、accessibility-specialist，各自被要求找出实现未满足其规范之处，而非确认其匹配（`studio` 对抗式评审）；三者均在阶段 5 前返回结果
8. 阶段 5——处理评审反馈；验证动画可跳过；通过音频事件系统确认 UI 音效；最终检查 interaction-patterns.md
9. 总结报告：UX 规范状态、UX 评审结论、视觉设计状态、实现状态、无障碍合规性、输入方式支持、模式库状态；结论：COMPLETE

**断言：**
- [ ] 在第一个代理被 spawn 之前出现命名 `team.size: studio` 的生效集合行
- [ ] 阶段 1a 读取全部五个来源，并在向 ux-designer 简报前逐一列为存在或缺失
- [ ] 阶段 2 前检查 UX 评审门——在 APPROVED 之前阶段 2 不开始
- [ ] 阶段 2 的 art-director 评审完整规范，而不仅是线框图
- [ ] 引擎 UI 专家取自 `specialists.ui`，并在阶段 3 中先于 ui-programmer spawn
- [ ] ui-programmer 列出它将创建或更改的文件，并在对该集合的一次询问得到肯定答复前不写入任何文件
- [ ] 阶段 4 代理同时启动（ux-designer、art-director、accessibility-specialist）
- [ ] 在 `team.size: studio` 下，阶段 4 的提示为对抗式，且总结报告说明对抗式评审已运行
- [ ] 所有文件写入委托给子代理和子技能
- [ ] 最终总结报告结论为 COMPLETE
- [ ] 下一步包括 `/ux-review`、`/code-review`、`/team-polish`

---

### 用例 2：UX 评审门——规范未通过评审；技能在实现前停止

**夹具：**
- 解析后的配置块：`automation: collaborative`（任意 `team.size`——ux-designer 在每个规模下都活跃）
- 阶段 1b 产出的 `design/ux/inventory-screen.md`
- `/ux-review` 返回 NEEDS REVISION，并标记具体问题（例如手柄导航流程不完整、对比度低于最低要求）

**输入：** `/team-ui inventory screen`

**预期行为：**
1. 阶段 1a + 1b 完成——产出 UX 规范
2. 阶段 1c——`/ux-review design/ux/inventory-screen.md` 返回 NEEDS REVISION
3. 技能不进入阶段 2
4. `AskUserQuestion` 呈现具体标记的问题和选项：
   - (a) 返回 ux-designer 解决问题并重新评审
   - (b) 接受风险，仍进入阶段 2（有意识的决定）
5. 若用户选择 (a)：ux-designer 修订规范，重新运行 `/ux-review`；循环持续直到 APPROVED 或用户覆盖
6. 若用户选择 (b)：技能继续，并在最终报告中明确记录 NEEDS REVISION
7. 技能不会静默越过门

**断言：**
- [ ] UX 评审结论为 NEEDS REVISION 时阶段 2 不开始
- [ ] `AskUserQuestion` 在提供选项前呈现具体标记的问题
- [ ] 用户必须做出有意识的覆盖选择——技能不假定覆盖
- [ ] 若用户接受风险，最终报告记录该覆盖：NEEDS REVISION 结论、遗留未解决的问题，以及用户选择继续
- [ ] 提供修订并重新评审的循环（而非一次性失败）
- [ ] 技能在评审失败时不丢弃已产出的 UX 规范

**变体 B——`/ux-review` 返回 NOT ASSESSED：**
- 夹具同上，但 `design/accessibility-requirements.md` 缺失，其他评审维度未发现缺口，因此 `/ux-review` 返回 NOT ASSESSED（无障碍：无已承诺等级）
- 预期：阶段 2 不开始；`AskUserQuestion` 指出未评估的维度及其缺失输入（`/ux-design accessibility`），选项为产出该输入并重新运行评审，或有意识地继续；若用户继续，最终报告记录 NOT ASSESSED 覆盖，且阶段 4 仍报告 `Accessibility: NOT ASSESSED — no committed tier (design/accessibility-requirements.md absent)`

**断言（变体 B）：**
- [ ] 在 NOT ASSESSED 下，未经用户明确决定阶段 2 不开始——它像 NEEDS REVISION 一样阻塞
- [ ] 提示指出 `/ux-review` 无法评估的维度以及使其可检查所需的输入，而非规范修订
- [ ] 技能不会针对同一缺失输入重新运行 `/ux-review` 并将结果视为新的
- [ ] 最终报告记录覆盖，含 NOT ASSESSED 结论和未评估的维度

---

### 用例 3：无参数——显示用法指引

**夹具：**
- 任意项目状态

**输入：** `/team-ui`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法消息，说明必需参数（UI 功能描述）
3. 显示调用格式 `/team-ui [UI feature description] [--review full|lean|solo]` 及示例（`inventory screen`、`main menu`、`combat HUD`）
4. 技能退出，不 spawn 任何子代理，也不读取任何项目文件

**断言：**
- [ ] 无参数时技能不 spawn 任何子代理
- [ ] 用法消息显示完整 argument-hint：`/team-ui [UI feature description] [--review full|lean|solo]`
- [ ] 至少显示一个有效调用示例
- [ ] 失败前不读取任何 UX 规范文件或 GDD
- [ ] 不显示结论（流水线从未启动）

---

### 用例 4：无障碍并行评审——阶段 4 同时运行三个流

**夹具：**
- 解析后的配置块：`review_mode: full`、`team.size: small`、`automation: collaborative`
- `design/ux/inventory-screen.md` 存在（APPROVED）
- 视觉设计规范完整
- 实现完成
- `design/accessibility-requirements.md` 已承诺等级：Standard

**输入：** `/team-ui inventory screen`（从阶段 3 完成处恢复）

**预期行为：**
1. 确认实现完成后阶段 4 开始
2. 同时发出三个 Agent 调用：ux-designer、art-director、accessibility-specialist
3. 每个流独立运行：
   - ux-designer：验证实现与线框图匹配，测试仅键盘和仅手柄导航，检查无障碍功能是否可用
   - art-director：在支持的最低和最高分辨率下验证与美术圣经的视觉一致性
   - accessibility-specialist：对照 `design/accessibility-requirements.md` 中的 Standard 无障碍等级进行审计；任何违规标记为阻塞项
4. 技能等待全部三个结果后才进入阶段 5
5. `AskUserQuestion` 在阶段 5 开始前呈现全部三个评审结果

**断言：**
- [ ] 在等待任何结果之前发出全部三个 Agent 调用（并行，非顺序）
- [ ] 全部三个阶段 4 代理返回前阶段 5 不开始
- [ ] accessibility-specialist 明确读取 `design/accessibility-requirements.md` 以获取已承诺等级
- [ ] 无障碍违规标记为阻塞（不仅是建议性）
- [ ] `AskUserQuestion` 在阶段 5 批准前一起显示全部三个评审流的结果
- [ ] 任何阶段 4 代理的输出都不作为另一阶段 4 代理的输入

---

### 用例 5：缺失交互模式库——技能指出缺口而非发明模式

**夹具：**
- 解析后的配置块：`automation: collaborative`（任意 `team.size`）
- `design/ux/interaction-patterns.md` 不存在
- 所有其他必需文件存在

**输入：** `/team-ui settings menu`

**预期行为：**
1. 阶段 1a——编排器尝试读取 `design/ux/interaction-patterns.md`；文件未找到
2. 技能上报缺口："interaction-patterns.md 不存在——无可复用的现有模式"
3. `AskUserQuestion` 呈现选项：
   - (a) 先运行 `/ux-design patterns` 建立模式库，然后继续
   - (b) 无模式库继续——ui-programmer 将所有创建的模式视为新模式，并在完成时逐一添加到新的 `design/ux/interaction-patterns.md`
4. 技能不从其他来源发明或假定模式
5. 若用户选择 (b)：明确指示 ui-programmer 将所有创建的模式视为新模式，并在完成时逐一添加到新的 `design/ux/interaction-patterns.md`
6. 最终报告注明 interaction-patterns.md 已创建（若用户跳过则仍缺失）

**断言：**
- [ ] 技能不静默忽略缺失的模式库
- [ ] 技能不通过仅从功能名或 GDD 猜测来发明模式
- [ ] `AskUserQuestion` 提供"先创建模式库"选项（引用 `/ux-design patterns`）
- [ ] 若用户无库继续，指示 ui-programmer 将所有模式视为新模式
- [ ] 最终报告记录模式库状态（已创建 / 缺失 / 已更新）
- [ ] 技能不整体失败——缺口被记录，用户获得选择

---

### 用例 6：`review_mode` 不门控任何内容——art-director 在 lean 和 solo 下仍在名册中

**夹具：**
- `project.yaml`：运行 A–C 为 `team.size: small`，运行 D 为 `team.size: individual`
  （固定：`small` 是包含 art-director 和
  accessibility-specialist 的最小规模）；`modes.review_mode` 未设置，因此每次运行的 `--review`
  值决定
- `design/gdd/game-concept.md`、`design/player-journey.md`、
  `design/ux/interaction-patterns.md` 和 `design/accessibility-requirements.md`
  （等级 Standard）存在；`/ux-review` 返回 APPROVED
- 运行 A：`/team-ui inventory screen --review full`。运行 B：`--review lean`。运行 C：`--review solo`。运行 D：在 `individual` 下 `--review lean`

**预期行为：**
1. 运行 A–C 以相同的 `team.size: small` 生效集合行开始：
   ui-programmer + ux-designer + accessibility-specialist + art-director
2. 在全部三次运行中，art-director 为阶段 2（视觉设计）和
   阶段 4 评审而 spawn，accessibility-specialist 为阶段 4 而 spawn——评审模式
   不跳过团队成员
3. 任何运行在任何阶段都不 spawn CD-、TD-、PR- 或 AD-PHASE-GATE："任何阶段
   在任何 `review_mode` 下都不 spawn CD-、TD-、PR- 或 AD-PHASE-GATE"
4. 运行 D（`individual`）在 `Not spawned this run:` 下列出 art-director 和 accessibility-specialist；其工作通过 ui-programmer 和 ux-designer 咨询。
   这是规模导致的，而非 `lean`

**断言：**
- [ ] 运行 A（`full`）、运行 B（`lean`）和运行 C（`solo`）中 art