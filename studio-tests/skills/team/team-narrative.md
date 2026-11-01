# 技能测试规范：/team-narrative

## 技能摘要

通过五阶段流水线编排叙事团队：叙事方向（narrative-director）→ 世界基础、对话起草与视觉方向（world-builder、writer 和 art-director 并行）→ 关卡叙事整合（level-designer）→ 一致性审查（narrative-director）→ 打磨 + 本地化合规（writer、localization-lead 和 world-builder 并行）。阶段 0 解析 `review_mode`、`automation` 和 `team.size`，技能在阶段 1 之前公布激活集——默认 `individual` 下只生成 writer（仅在明确出现支柱冲突时生成 narrative-director），`small` 增加 narrative-director，当 `review_mode` 为 `full` 时再加 localization-lead 和 world-builder，而 `studio` 无论审查模式如何都激活所有叙事 agent。每个 agent 起草到技能路径表中指定的 `production/narrative/[content-slug]/` 下的具体路径；阶段 4 运行 ND-CONSISTENCY 门禁；编排器在一次批准后将最终文档写入 `design/narrative/`。在每个阶段转换处使用 `AskUserQuestion`。产出叙事摘要报告。所有阶段成功时判定为 COMPLETE——当 ND-CONSISTENCY 审查无法评估时为 `COMPLETE — consistency NOT ASSESSED ([input])`——当依赖未解决时为 BLOCKED。

---

## 静态断言（结构性）

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含 "File Write Protocol" 章节
- [ ] 子 agent 将草稿写入 `production/narrative/[content-slug]/` 下；编排器仅在一次 "May I write" 批准后写入最终的 `design/narrative/` 文档
- [ ] 为每个 agent 指定具体的草稿路径（包括 level-designer 和 localization-lead）以及每个设计产物最终成为的 `design/narrative/` 文档——localization-lead 的审查不是其中之一，保留在 `production/` 中
- [ ] 每个 agent 提示词以返回契约结尾：写入的路径、≤5 条要点摘要、BLOCKED/CONCERNS 项各一行
- [ ] 将阶段门禁定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门禁处声称生成 director
- [ ] 末尾有下一步交接（引用 `/design-review`、`/localize extract`、`/dev-story`）
- [ ] 存在 Error Recovery Protocol 章节：验证指定产物在磁盘上、立即暴露 BLOCKED、始终产出部分报告，完整流程见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在继续之前于阶段转换处使用 `AskUserQuestion`（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，技能要求在阶段 1 之前进行一行激活集公告
- [ ] 阶段 2 明确并行生成 world-builder、writer 和 art-director
- [ ] 阶段 5 明确并行生成 writer、localization-lead 和 world-builder

---

## 夹具默认值

每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身的夹具另有说明。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有五个阶段完成，叙事文档交付

**夹具：**
- 解析后的配置块：`team.size: studio`、`automation: collaborative`
- 目标功能存在游戏概念和 GDD（例如 `design/gdd/faction-intro.md`）
- 存在角色语音档案（例如 `design/narrative/characters/`）
- 存在用于交叉引用的现有传说条目（例如 `design/narrative/lore/`）
- 现有条目与新内容之间不存在传说矛盾

**输入：** `/team-narrative faction introduction cutscene for the Ironveil faction`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: studio` 的激活集
2. 阶段 1：生成 narrative-director；将叙事简报草稿写入 `production/narrative/[content-slug]/brief.md`，定义故事节拍、涉及角色、情感基调和传说依赖
3. `AskUserQuestion` 呈现叙事简报；用户在阶段 2 开始前批准
4. 阶段 2：并行生成 world-builder、writer 和 art-director；world-builder 起草 Ironveil 传说条目，writer 使用角色语音档案起草对话，art-director 起草视觉方向——各自在 `production/narrative/[content-slug]/` 下
5. 阶段门禁：收集所有三个阶段 2 输出，然后 `AskUserQuestion` 呈现传说、对话和视觉方向摘要；用户在阶段 3 开始前批准
6. 阶段 3：生成 level-designer；产出环境叙事布局、触发器放置和节奏计划
7. `AskUserQuestion` 呈现关卡叙事计划；用户在阶段 4 开始前批准
8. 阶段 4：生成 narrative-director 并运行 ND-CONSISTENCY 门禁；审查语音、传说和节奏并返回 APPROVE
9. `AskUserQuestion` 呈现审查结果；用户在阶段 5 开始前批准
10. 阶段 5：并行生成 writer、localization-lead 和 world-builder；writer 执行最终自审；localization-lead 验证 i18n 合规；world-builder 最终确定正典级别
11. 编排器列出最终文档并询问一次："May I write these [N] narrative documents to `design/narrative/`?"；批准后写入
12. 呈现最终摘要报告
13. 判定：COMPLETE

**断言：**
- [ ] 在生成第一个 agent 之前出现命名 `team.size: studio` 的激活集行
- [ ] narrative-director 在阶段 1 中先于任何其他 agent 生成
- [ ] `AskUserQuestion` 出现在阶段 1 输出之后、阶段 2 启动之前
- [ ] world-builder、writer 和 art-director 的 Agent 调用在阶段 2 中同时发出（非顺序）
- [ ] 在阶段 2 `AskUserQuestion` 批准之前不启动 level-designer
- [ ] narrative-director 在阶段 4 中使用门禁 ND-CONSISTENCY 重新生成——在每种审查模式下，因为这是本流水线自身的审查，`review_mode` 从不跳过它——其 APPROVE / CONCERNS / REJECT 判定决定阶段 5 是否开始
- [ ] 阶段 5 同时生成所有三个 agent（writer、localization-lead、world-builder）
- [ ] 每个 agent 提示词命名其来自技能路径表的草稿路径
- [ ] 在单次 "May I write" 批准之前，`design/narrative/` 下不写入任何内容
- [ ] 摘要报告包括：叙事简报状态、创建/更新的传说条目、编写的对话行数、关卡叙事整合点、一致性审查结果
- [ ] 编排器除批准的最终文档外不写入任何内容
- [ ] 交付后判定为 COMPLETE

---

### 用例 2：发现传说矛盾——world-builder 在 writer 继续之前发现冲突

**夹具：**
- 解析后的配置块：`team.size: studio`、`automation: collaborative`
- `design/narrative/lore/ironveil-history.md` 处的现有传说条目声明 Ironveil 派系成立于 200 年前
- 新叙事简报（来自阶段 1）声明 Ironveil 成立于 50 年前
- writer 已在阶段 2 中与 world-builder 并行生成

**输入：** `/team-narrative ironveil faction introduction cutscene`

**预期行为：**
1. 阶段 1–2 正常开始
2. 阶段 2 world-builder 检测到叙事简报与现有传说之间的事实矛盾：成立日期冲突
3. world-builder 返回 BLOCKED，原因："Lore contradiction found — founding date conflicts with `design/narrative/lore/ironveil-history.md`"
4. 编排器立即暴露矛盾："world-builder: BLOCKED — Lore contradiction: founding date in narrative brief (50 years ago) conflicts with existing canon (200 years ago in `ironveil-history.md`)"
5. 编排器评估依赖：writer 的对话依赖正典传说——不解决矛盾，writer 的草稿无法最终确定
6. `AskUserQuestion` 呈现错误恢复选项：
   - 跳过 world-builder 并在最终报告中注明缺口
   - 以更窄范围重试（例如 world-builder 依据用户确认的正典工作）
   - 在此停止，先解决传说文档中的矛盾
7. writer 输出保留在部分报告中——工作不被丢弃
8. 在矛盾解决或用户明确选择跳过之前，编排器不进入阶段 3

**断言：**
- [ ] 矛盾在阶段 3 开始前暴露
- [ ] 编排器不通过选择某一版本来静默解决矛盾
- [ ] `AskUserQuestion` 呈现至少 3 个选项，包括 "stop and resolve first"
- [ ] writer 的草稿输出保留在部分报告中，不被丢弃
- [ ] 在用户解决矛盾或明确选择跳过 world-builder 之前，不启动阶段 3（level-designer）
- [ ] 如果用户停止以解决矛盾，判定为 BLOCKED（非 COMPLETE）

---

### 用例 3：无参数——显示用法指引

**夹具：**
- 任意项目状态

**输入：** `/team-narrative`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法指引：例如 "Usage: `/team-narrative [narrative content description] [--review full|lean|solo]` — describe the story content, scene, or narrative area to work on (e.g., `boss encounter cutscene`, `faction intro dialogue`, `tutorial narrative`)"
3. 技能退出，不生成任何 agent

**断言：**
- [ ] 未提供参数时技能不生成任何 agent
- [ ] 用法消息显示完整的 argument-hint——`/team-narrative [narrative content description] [--review full|lean|solo]`——以及一个参数示例
- [ ] 技能不尝试从项目文件猜测或推断叙事主题
- [ ] 不使用 `AskUserQuestion`——输出为直接指引

---

### 用例 4：本地化合规——localization-lead 因不可翻译字符串而阻塞

**夹具：**
- 解析后的配置块：`team.size: studio`、`automation: collaborative`
- 阶段 1–4 成功完成
- 阶段 5 开始；writer 和 world-builder 无问题完成
- localization-lead 发现一行对话使用硬编码的格式化日期字符串（例如 `"On March 12th, Year 3"`），在没有区域感知格式化器的情况下无法在特定区域翻译中存活，并在其返回契约中作为 BLOCKED 项返回

**输入：** `/team-narrative ironveil faction introduction cutscene`（阶段 5 场景）

**预期行为：**
1. 阶段 5 并行生成 writer、localization-lead 和 world-builder
2. localization-lead 返回 BLOCKED："String key `dialogue.ironveil.intro.003` contains a hardcoded date format (`March 12th, Year 3`) that will not localize correctly — requires a locale-aware date placeholder"
3. 编排器立即暴露："localization-lead: BLOCKED — [reason with the string key]"
4. writer 和 world-builder 的阶段 5 输出保留在部分报告中
5. `AskUserQuestion` 呈现错误恢复选项：
   - 跳过并在最终报告中注明缺口（交付时标记该字符串）
   - 以更窄范围重试（例如 writer 修订那一行，然后 localization-lead 重新检查）
   - 在最终确定前停止并解决
6. 如果用户跳过，最终摘要将未修复的字符串键列入未解决项；如果用户停止，判定为 `BLOCKED — [reason]`

**断言：**
- [ ] localization-lead 在阶段 5 中与 writer 和 world-builder 同时生成
- [ ] 硬编码日期格式被识别为本地化阻塞项（非静默通过）
- [ ] 具体字符串键和原因包含在暴露的 BLOCKED 消息中
- [ ] `AskUserQuestion` 提供跳过并注明缺口、重试和停止选项
- [ ] 如果用户未修复即继续，最终报告命名未修复的字符串键；如果用户停止，判定为 BLOCKED
- [ ] 技能不未经用户批准自动重写出问题的行

---

### 用例 5：Writer 阻塞——缺少角色语音档案

**夹具：**
- 解析后的配置块：`team.size: studio`、`automation: collaborative`
- 阶段 1 narrative-director 产出引用两个角色的叙事简报：Commander Varek 和 Advisor Selene
- `design/narrative/characters/` 中不存在这两个角色的角色语音档案
- 阶段 2 开始；world-builder 和 art-director 正常进行

**输入：** `/team-narrative ironveil surrender negotiation scene`

**预期行为：**
1. 阶段 1 完成；叙事简报将 Commander Varek 和 Advisor Selene 列为角色
2. 阶段 2：writer 与 world-builder 和 art-director 并行生成
3. writer 返回 BLOCKED："Cannot produce dialogue — no voice profiles found for Commander Varek or Advisor Selene in `design/narrative/characters/`. Voice profiles required to match character tone and speech patterns."
4. 编排器立即暴露阻塞项："writer: BLOCKED — Missing prerequisite: character voice profiles for Commander Varek and Advisor Selene"
5. world-builder 输出保留；产出带传说条目的部分报告
6. `AskUserQuestion` 呈现错误恢复选项：
   - 跳过 writer 并在最终报告中注明缺口
   - 以更窄范围重试（例如内联提供最小语音方向）
   - 在此停止，先创建语音档案再继续
7. 除非用户明确选择跳过，编排器在没有 writer 输出的情况下不进入阶段 3（level-designer）

**断言：**
- [ ] Writer 阻塞在阶段 3 开始前暴露
- [ ] world-builder 完成的传说输出保留在部分报告中
- [ ] 缺失的前置条件（语音档案）被具体命名（角色名和预期文件路径）
- [ ] `AskUserQuestion` 提供至少一个解决缺失前置条件的选项
- [ ] 编排器不伪造语音档案或编造角色声音
- [ ] 在 writer 处于 BLOCKED 且无用户明确授权时，不启动阶段 3

---

### 用例 6：`review_mode` 除 `small` 名册外不门控任何内容——ND-CONSISTENCY 在每种模式下运行

**夹具：**
- `project.yaml`：运行 A–C 为 `team.size: studio`，运行 D–F 为 `team.size: small`
  （固定：名册是 `review_mode` 仍然触及的唯一内容）；`modes.review_mode`
  未设置，因此每次运行的 `--review` 值决定
- 存在 `design/gdd/faction-intro.md`、角色语音档案和现有传说
- 运行 A 和 D：`/team-narrative faction introduction cutscene --review full`。
  运行 B 和 E：`--review lean`。运行 C 和 F：`--review solo`

**预期行为：**
1. 运行 A–C（`studio`）：无论审查模式如何，所有六个 agent 都激活，因此激活集行、阶段和生成的 agent 完全相同
2. 运行 D–F（`small`）：narrative-director + writer 在三次运行中都激活。运行 D（`full`）增加 localization-lead 和 world-builder；在运行 E 和 F 中，它们通过 writer 咨询并在 `Not spawned this run:` 行中命名
3. 阶段 4 在所有六次运行中生成 `narrative-director` 并运行 ND-CONSISTENCY："This phase runs in every review mode — it is this pipeline's own review, not an optional director gate"
4. 任何运行在任何阶段都不生成 CD-、TD-、PR- 或 AD-PHASE-GATE

**断言：**
- [ ] ND-CONSISTENCY 在六次运行的每一次中都在阶段 4 生成；`lean` 和 `solo` 不跳过它
- [ ] 任何运行中都不生成 director 门禁（CD-、TD-、PR- 或 AD-PHASE-GATE）
- [ ] 运行 A–C 显示相同的激活集、阶段和询问
- [ ] 在 `small` 下，localization-lead 和 world-builder 仅在运行 D（`full`）中生成，并在运行 E 和 F 中列在 `Not spawned this run:` 下
- [ ] 在 `small` 下的 `lean` 和 `solo` 中，传说和 i18n 工作仍通过 writer 完成，且输出如此说明
- [ ] ND-CONSISTENCY 的 APPROVE / CONCERNS / REJECT 回答在每种模式下决定阶段 5 是否开始

---

## 协议合规

- [ ] 在默认 `team.size: individual` 下，运行以 `Active set (team.size: individual): writer.` 和一行 `Not spawned this run:` 开头，命名 narrative-director、world-builder、art-director、level-designer 和 localization-lead，通过 writer 咨询——在任何 agent 生成之前
- [ ] 在 `collaborative` 模式下，每个阶段输出之后、下一阶段启动之前使用 `AskUserQuestion`
- [ ] 并行生成：阶段 2（world-builder + writer + art