# 技能测试规范：/team-level

## 技能摘要

为单个关卡或区域编排完整的关卡设计团队。通过五个步骤协调 narrative-director、world-builder、level-designer、systems-designer、art-director、accessibility-specialist 和 qa-tester：叙事 + 视觉方向并行（步骤 1）、布局（步骤 2）、系统（步骤 3）、制作概念 + 无障碍并行（步骤 4）、QA 规划（步骤 5）。阶段 0 解析 `review_mode`、`automation` 和 `team.size`，技能在步骤 1 前公布激活集——`individual` 仅生成 level-designer，`small` 增加 systems-designer + art-director + qa-tester，`studio` 增加 narrative-director、world-builder 和 accessibility-specialist。编排器自行汇总所有团队输出，仅在通过 `AskUserQuestion` 询问"我可以写入……吗？"后才写入 `design/levels/[level-name].md`；各 agent 的产物由子 agent 在有界写入例外下写入。在每个步骤门禁使用 `AskUserQuestion`。生成摘要报告，结论为 COMPLETE / BLOCKED，并交接至 `/design-review`、`/dev-story`、`/qa-plan`。

---

## 静态断言（结构性）

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段/步骤标题（步骤 1 至步骤 5 均存在）
- [ ] 包含结论关键词：COMPLETE、BLOCKED
- [ ] 文件写入协议：各 agent 产物由子 agent 在有界例外下写入（指定路径、`production/`、`docs/` 或 `tests/` 下的新产物、门禁步骤）；唯一例外是汇总的关卡文档，由编排器在其自身的"我可以写入……吗？"提示后自行写入
- [ ] 为每个 agent 指定 `production/levels/[level-name]/` 下的目标文件——`narrative.md`、`lore.md`、`visual-direction.md`、`layout.md`、`systems.md`、`production-concepts.md`、`accessibility.md`——以及 qa-tester 的 `production/qa/test-cases/[level-name]-cases.md`
- [ ] 将阶段门禁定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门禁处生成 director
- [ ] 末尾有下一步交接（引用 `/design-review`、`/dev-story`、`/qa-plan`）
- [ ] 存在错误恢复协议章节：在将步骤视为完成前验证指定产物已在磁盘上，立即暴露 BLOCKED，始终生成部分报告，完整流程（暴露 → 评估 → 选项 → 部分报告）见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在步骤转换处使用 `AskUserQuestion` 获取用户批准后再继续（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，技能要求在步骤 1 前进行一行激活集公告
- [ ] 步骤 1 明确并行生成 narrative-director、world-builder 和 art-director
- [ ] 步骤 4 明确标记为并行（art-director 和 accessibility-specialist 同时运行）并等待两者
- [ ] 上下文收集读取：`design/gdd/game-concept.md`（或 `design/game-brief.md`）、`design/gdd/game-pillars.md`、`design/levels/`、`design/narrative/` 以及相关世界构建文档
- [ ] `## How to Delegate` 列出全部七个角色（narrative-director、world-builder、level-designer、systems-designer、art-director、accessibility-specialist、qa-tester）
- [ ] accessibility-specialist 输出包含严重性评级（BLOCKING / RECOMMENDED / NICE TO HAVE）
- [ ] 最终关卡设计文档保存至 `design/levels/[level-name].md`，其中 `[level-name]` 是参数 slug 化后的结果——小写、空格 → 连字符——相邻区域路径使用相同规则

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有团队成员产出输出，文档汇总并保存

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- `design/gdd/game-concept.md` 存在且已填充
- `design/gdd/game-pillars.md` 存在
- `design/levels/` 目录存在（可能包含其他关卡文档）
- `design/narrative/` 目录存在且含相关叙事文档
- 森林区域的世界构建文档存在

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: studio` 的激活集
2. 上下文收集——编排器读取 game-concept.md、game-pillars.md、`design/levels/` 中的现有关卡文档、`design/narrative/` 中的叙事文档以及森林区域的世界构建文档
3. 步骤 1——narrative-director、world-builder 和 art-director 一起生成（所有三个 Agent 调用在任何结果被等待之前发出）：叙事目的和情感弧线；背景设定和世界规则；视觉主题目标、光照氛围、形状语言和地标；`AskUserQuestion` 在步骤 2 前呈现所有三个输出
4. 步骤 2——level-designer 带着步骤 1 的输出生成，包括 art-director 的视觉目标作为显式约束；设计空间布局、节奏曲线、遭遇、谜题、入口/出口点；编排器检查 `design/levels/` 中引用的每个相邻区域；`AskUserQuestion` 在步骤 3 前确认布局
5. 步骤 3——systems-designer 生成：敌人组成、战利品表、难度平衡、区域特定机制、资源分布；`AskUserQuestion` 在步骤 4 前确认系统
6. 步骤 4——art-director（位置特定制作概念）和 accessibility-specialist 并行生成；每个无障碍问题评级为 BLOCKING / RECOMMENDED / NICE TO HAVE；编排器等待两者；`AskUserQuestion` 在步骤 5 前呈现两个输出
7. 步骤 5——qa-tester 生成：关键路径测试用例、边界/边缘用例（序列破坏、软锁）、试玩清单、验收标准
8. 编排器自行将所有团队输出汇总到关卡设计模板中——不重新生成 level-designer 来做这件事——然后通过 `AskUserQuestion` 询问："我可以将汇总的关卡设计写入 `design/levels/forest-dungeon.md` 吗？"并在批准后写入
9. 摘要报告：区域概览、遭遇数量、预估资产列表、叙事节拍、跨团队依赖、未解决的跨关卡依赖、带解决状态的无障碍问题；结论：COMPLETE
10. 列出的下一步：`/design-review design/levels/forest-dungeon.md`、`/dev-story`、`/qa-plan`

**断言：**
- [ ] 在第一个 agent 生成之前出现命名 `team.size: studio` 的激活集行
- [ ] 在任何 agent 生成之前，上下文收集期间读取所有五个来源
- [ ] 步骤 1 中 narrative-director、world-builder 和 art-director 的 Agent 调用并行发出，且三者都在步骤 2 前完成
- [ ] level-designer 的简报将 art-director 的步骤 1 视觉目标作为约束
- [ ] 在每个步骤门禁调用 `AskUserQuestion`（最少：步骤 1、步骤 2、步骤 3、步骤 4 之后）
- [ ] 步骤 4 的 agent（art-director、accessibility-specialist）同时启动
- [ ] 汇总的关卡文档由编排器写入，而非重新生成的 agent，且仅在其自身的"我可以写入"批准后
- [ ] 关卡文档保存至 `design/levels/forest-dungeon.md`（从参数 slug 化）
- [ ] 最终摘要报告中结论为 COMPLETE
- [ ] 下一步包括 `/design-review`、`/dev-story`、`/qa-plan`
- [ ] 摘要报告包括：区域概览、遭遇数量、预估资产列表、叙事节拍

---

### 用例 2：Agent 受阻（world-builder）——生成部分报告并注明缺口

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- `design/gdd/game-concept.md` 存在
- 森林区域的世界构建文档不存在
- world-builder agent 返回 BLOCKED："未找到森林区域的世界构建文档——无法提供背景设定"

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 上下文收集完成
2. 步骤 1——narrative-director 和 art-director 成功完成；world-builder 返回 BLOCKED
3. 触发错误恢复协议："world-builder：BLOCKED——无森林区域的世界构建文档"
4. `AskUserQuestion` 呈现选项：
   - (a) 跳过 world-builder 并在最终报告中注明缺口
   - (b) 以更窄范围重试（world-builder 仅关注可从 game-concept.md 推断的内容）
   - (c) 在此停止并先创建世界构建文档
5. 如果用户选择 (a)：流水线在没有背景设定基础的情况下继续步骤 2–5；缺失的世界构建上下文在最终报告中列为缺口
6. 如果 world-builder 仍未解决，最终报告将 world-builder 部分标记为 BLOCKED，总体结论为 BLOCKED

**断言：**
- [ ] world-builder 失败时立即出现 BLOCKED 暴露消息——在无用户输入的情况下步骤 2 开始之前
- [ ] `AskUserQuestion` 至少提供三个选项（跳过 / 重试 / 停止）
- [ ] 生成部分报告——narrative-director 和 art-director 的已完成工作不被丢弃
- [ ] 缺失的世界构建上下文在最终报告中列为缺口
- [ ] world-builder 仍未解决时总体结论为 BLOCKED（非 COMPLETE）
- [ ] 不代替 world-builder 写入任何内容——协议选项是"跳过此 agent 并注明缺口"（`.claude/docs/error-recovery-protocol.md`），而非替代背景设定

---

### 用例 3：无参数——显示用法指引

**夹具：**
- 任意项目状态

**输入：** `/team-level`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法消息，说明所需参数（要设计的关卡名称或区域）
3. 显示调用格式 `/team-level [level name or area to design] [--review full|lean|solo]` 及示例（`forest temple`、`tutorial village`、`final boss arena`）
4. 技能退出，不读取任何项目文件或生成任何子 agent

**断言：**
- [ ] 无参数时技能不生成任何子 agent
- [ ] 用法消息显示完整的 argument-hint：`/team-level [level name or area to design] [--review full|lean|solo]`
- [ ] 至少显示一个有效调用示例
- [ ] 失败前不读取任何 GDD 或关卡文件
- [ ] 不显示结论（流水线从未启动）

---

### 用例 4：无障碍审查门禁——签核前暴露阻断性问题

**夹具：**
- 解析的配置块：`team.size: studio`、`automation: collaborative`
- 步骤 1–3 成功完成
- `design/accessibility-requirements.md` 承诺层级：Standard
- accessibility-specialist（步骤 4，并行）标记一个 BLOCKING 问题：穿过森林地牢的关键路径要求玩家仅通过颜色区分两种环境危害（毒池 vs. 浅水）——没有形状、图标或音频提示区分它们

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 步骤 1–3 完成；步骤 4 并行阶段开始
2. accessibility-specialist 返回：BLOCKING 问题——"关键路径危害区分仅依赖颜色（毒池 vs. 浅水）。按 Standard 无障碍层级需要形状、图标或音频提示。"
3. art-director 返回步骤 4 输出（完整）
4. 技能通过 `AskUserQuestion` 呈现两个步骤 4 结果——BLOCKING 问题突出显示
5. `AskUserQuestion` 提供：
   - (a) 返回 level-designer + art-director 在步骤 5 前重新设计被标记的元素
   - (b) 记录为已知无障碍缺口并继续步骤 5，在最终报告中明确记录该问题
6. 技能在用户确认 BLOCKING 问题之前不继续步骤 5
7. 最终摘要无论用户选择如何都列出无障碍问题及其解决状态

**断言：**
- [ ] BLOCKING 无障碍问题不被视为建议性——它作为阻断性问题暴露
- [ ] `AskUserQuestion` 呈现具体问题文本（而非仅"发现无障碍问题"）
- [ ] 用户确认 BLOCKING 问题之前步骤 5（qa-tester）不开始
- [ ] 提供修订路径：level-designer + art-director 可在继续前被送回
- [ ] 最终报告包括无障碍问题及其解决状态
- [ ] accessibility-specialist 阻断时 art-director 的已完成输出不被丢弃

---

### 用例 5：循环关卡引用——标记相邻区域依赖

**夹具：**
- 解析的配置块：`automation: collaborative`（任意 `team.size`——level-designer 在每个规模下都激活）
- 步骤 1–3 进行中
- level-designer（步骤 2）生成的布局指定连接至"the crystal caves"（相邻区域）的入口/出口点
- `design/levels/crystal-caves.md` 不存在——crystal caves 区域尚未设计

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 步骤 2——level-designer 生成布局，包括："西出口连接至 crystal-caves 入口点 A"
2. 编排器检查 `design/levels/` 中的 `crystal-caves.md`；文件未找到
3. 暴露依赖缺口："关卡引用 crystal-caves 作为相邻区域，但 `design/levels/crystal-caves.md` 不存在"
4. `AskUserQuestion` 呈现选项：
   - (a) 以占位引用继续——在关卡文档中将依赖标记为 UNRESOLVED
   - (b) 暂停并先运行 `/team-level crystal caves` 建立该区域
5. 技能不虚构 crystal caves 内容来满足引用
6. 如果用户选择 (a)：关卡文档汇总时西出口标记为 UNRESOLVED；在摘要报告的未解决跨关卡依赖章节中标记
7. 最终报告包括未解决跨关卡依赖章节

**断言：**
- [ ] 技能通过检查 `design/levels/` 检测缺失的相邻区域——不假设它稍后会创建
- [ ] 技能不虚构 crystal caves 内容（背景设定、布局、连接）来解决引用
- [ ] `AskUserQuestion` 提供引用 `/team-level` 的"先设计 crystal caves"选项
- [ ] 如果用户以占位继续，关卡文档明确将西出口标记为 UNRESOLVED
- [ ] 摘要报告包括列出未解决引用的未解决跨关卡依赖章节
- [ ] 循环或前向引用不会导致技能循环或崩溃

---

## 协议合规

- [ ] 在默认 `team.size: individual` 下，运行以 `Active set (team.size: individual): level-designer` 行和 `Not spawned this run:` 行开头，列出流水线命名的所有其他 agent——在任何 agent 生成之前
- [ ] 在 `collaborative` 模式下，每个步骤转换处使用 `AskUserQuestion`——用户在流水线推进前批准
- [ ] 各 agent 产物由子 agent 写入编排器指定的路径；仅汇总的关卡文档由编排器在其自身的 `AskUserQuestion` 批准后写入
- [ ] 指定产物在磁盘上缺失的步骤被视为失败，而非完成
- [ ] 遵循错误恢复协议：暴露 → 评估 → 提供选项 → 部分报告
- [ ] 步骤 1（narrative-director、world-builder、art-director）和步骤 4（art-director、accessibility-specialist）按技能规范并行启动
- [ ] 即使 agent 为 BLOCKED 也始终生成部分报告
- [ ] 无障碍 BLOCKING 问题在步骤 5 前暴露并要求用户明确确认
- [ ] 结论为 COMPLETE / BLOCKED 之一
- [ ] 末尾有下一步：`/design-review`、`/dev-story`、`/qa-plan`

---

## 覆盖说明

- 受阻 world-builder 用例（用例 2）中的"以更窄范围重试"选项——重试行为本身未深入测试；其完整路径类似于用例 2 及其他 team-* 规范中覆盖的受阻 agent 模式。
- systems-designer（步骤 3）阻断场景未单独测试；适用相同的错误恢复协议，模式由用例 2 验证。
- 步骤 4 并行顺序（art-director 在 accessibility-specialist 之前或之后完成）不影响结果——无论顺序如何，两者都必须在步骤 5 前返回。
- 关卡文档 slug 规则（小写、空格 → 连字符）由用例 1（`forest dungeon` → `forest-dungeon.md`）和用例 5（`crystal caves` → `crystal-caves.md`）检验；特殊字符和超长名称未覆盖。
- `small` 激活集（增加 systems-designer + art-director）未单独设用例。