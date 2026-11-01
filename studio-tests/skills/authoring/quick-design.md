# 技能测试规范：/quick-design

## 技能摘要

`/quick-design [变更的简要描述]` 为小到不值得写完整 GDD 的变更生成一份轻量级 Quick Design Spec。它先将变更归类为 **Tuning**、**Tweak**、**Addition** 或 **New Small System**，并通过 `AskUserQuestion` 确认分类，其选项 `[F]` 会重定向到 `/design-system`。若变更引入的系统具有显著的跨系统依赖、需要约一周以上的工作，或改变系统的核心规则，则改为重定向到 `/design-system`（判定 `REDIRECTED`）。

随后扫描上下文（`design/gdd/` 中相关的 GDD、系统索引（若存在）、`design/quick-specs/` 中的先前规范——逐一列出，无论一致还是冲突——以及 Tuning 情况下的 `assets/data/`），按所属类别的格式起草规范，完整呈现并给出 Approve / Revise / 重定向选项，批准后询问"May I write this Quick Design Spec to `design/quick-specs/[kebab-case-title]-[YYYY-MM-DD].md`?"。规范所需的任何 GDD 修改都单独询问，并展示旧文本与新文本。最后以交接块和判定 `COMPLETE` 结束。

该技能没有总监门禁，也不解析 `review_mode`；按设计，quick spec 绕过 `/design-review` 和 `/review-all-gdds`。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：`COMPLETE`、`REDIRECTED`
- [ ] 包含"May I write"协作协议用语（针对 quick spec 文件，并单独针对任何 GDD 更新）
- [ ] 末尾有下一步交接
- [ ] 说明按设计 quick spec 绕过 `/design-review` 和 `/review-all-gdds`，并列出何时重定向到完整流水线
- [ ] 记录将超范围变更重定向到 `/design-system` 的范围检查

---

## 总监门禁检查

无。该技能只解析 `automation` 和 `workflow`（不解析 `review_mode`），`allowed-tools` 中没有 `Agent` 工具，也不生成任何总监或专家 agent。review mode 设置对它没有影响。

---

## Fixture 默认值

除非其自身的 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——对现有系统的 Tweak

**Fixture：**
- `design/gdd/movement.md` 存在，并在其 Detailed Rules 中描述了冲刺无敌
- `design/gdd/systems-index.md` 存在
- `design/quick-specs/dash-cooldown-tuning-[earlier-date].md` 调整了 movement 中的冲刺冷却，未涉及无敌

**输入：** `/quick-design make dash invincible on frame 1`

**预期行为：**
1. 技能将变更归类为 Tweak，并询问"I've classified this as **Tweak** — [reason]. Is that correct?"，选项为 `[A]`–`[F]`
2. 上下文扫描读取 `design/gdd/movement.md` 的相关章节、系统索引和 `design/quick-specs/`，并报告"Found GDD at `design/gdd/movement.md`. Relevant section: [section]. Prior quick specs for this system: `design/quick-specs/dash-cooldown-tuning-[earlier-date].md` — agrees."
3. 按 Tweak/Addition 格式起草：Change Summary、Motivation、Design Delta（引用当前 GDD 规则）、New Rules / Values、Affected Systems、Acceptance Criteria、GDD Update Required?
4. 呈现完整草稿，然后询问 `[A] Approve — write it as shown` / `[B] Revise — I'll describe what to change` / `[C] This grew too large — redirect to /design-system instead`
5. 选择 `[A]` 时："May I write this Quick Design Spec to `design/quick-specs/dash-invincibility-tweak-[YYYY-MM-DD].md`?"——确认后，按需创建 `design/quick-specs/` 并写入文件
6. 由于规范改变了 `movement.md` 中的一条规则，随后单独询问"May I update `design/gdd/movement.md` — specifically the [section] section?"，先展示旧文本与新文本
7. 打印交接块（路径、类型、系统、GDD 更新状态）和"Verdict: **COMPLETE**"

**断言：**
- [ ] 在任何上下文扫描或起草之前，通过 `AskUserQuestion` 确认分类
- [ ] 即使先前的冲刺冷却规范不冲突，也在上下文报告中列出其名称
- [ ] 草稿使用 Tweak/Addition 格式，其 Design Delta 引用当前 GDD 文本
- [ ] 在询问写入批准之前展示完整草稿
- [ ] 文件路径为 `design/quick-specs/[kebab-case-title]-[YYYY-MM-DD].md`，日期为今天
- [ ] 在写入 quick spec 之后单独询问 GDD 修改，并展示旧文本与新文本
- [ ] 运行以交接块和判定 `COMPLETE` 结束

---

### 用例 2：失败路径——范围过大；重定向到 /design-system

**Fixture：**
- 场景 (a)：请求为"redesign the entire combat system"
- 场景 (b)：起草了一个 Tweak，在批准步骤用户选择 `[C] This grew too large — redirect to /design-system instead`

**输入：** (a) `/quick-design redesign the entire combat system` (b) `/quick-design allow combo to cancel into roll`

**预期行为：**
1. (a) 变更改变了系统的核心规则，因此技能在起草前停止并重定向到 `/design-system`——直接重定向，或通过分类提示的选项 `[F]`
2. (b) 技能在批准步骤停止
3. 两个场景均输出："Verdict: **REDIRECTED** — use `/design-system` for this change."
4. 不在 `design/quick-specs/` 下写入任何文件

**断言：**
- [ ] (a) 检测到范围超限，技能在起草前停止
- [ ] (a) 即使技能在分类控件之前停止，也打印判定行，而不仅是通过 `[F]`
- [ ] 消息将 `/design-system` 列为替代方案
- [ ] 不写入任何 quick spec 文件
- [ ] 判定为 `REDIRECTED`，而非 `COMPLETE`

---

### 用例 3：边界情况——Tuning 变更超出文档化的旋钮范围

**Fixture：**
- `design/gdd/player-controller.md` 有一个 Tuning Knobs 条目 `jump_height`，文档化范围为 3–5
- `assets/data/player.json` 中 `jump_height: 5`
- 先前的 quick spec `design/quick-specs/jump-height-tuning-[earlier-date].md` 曾将 `jump_height` 设为 5

**输入：** `/quick-design increase jump height from 5 to 6 units`

**预期行为：**
1. 技能将变更归类为 Tuning 并确认
2. 上下文扫描还检查 `assets/data/` 中保存该值的数据文件，并列出触及该参数的先前 quick spec，标记为冲突——它设置的值正是本次变更要替换的
3. 草稿使用 Tuning 格式：单个 Change 表（Parameter、Old Value、New Value、Rationale）、Tuning Knob Mapping 和 Acceptance Criteria
4. Tuning Knob Mapping 说明新值超出文档化范围，并解释为何应扩展该范围
5. 验收标准引用从 `assets/data/[file]` 读取该值

**断言：**
- [ ] 由于变更是 Tuning，检查 `assets/data/` 中的数据文件
- [ ] 报告触及 `jump_height` 的先前 quick spec，而非静默矛盾
- [ ] 草稿使用单表 Tuning 格式，而非 Tweak/Addition 章节
- [ ] Tuning Knob Mapping 说明该值超出文档化范围并解释扩展

---

### 用例 4：边界情况——无参数且无系统索引

**Fixture：**
- 无 `design/gdd/systems-index.md`
- `design/quick-specs/` 可能存在也可能不存在

**输入：** `/quick-design`（无参数）

**预期行为：**
1. 技能用纯文本提示要求用户描述变更
2. 在用户描述变更之前，不显示分类控件、不挑选变更、不写入任何内容
3. 描述之后，照常分类并确认
4. 上下文扫描注明"No systems index found — skipping dependency tier check."并继续

**断言：**
- [ ] 无参数时，技能以纯文本要求描述，而非猜测变更
- [ ] 仅在用户描述变更后才进行分类
- [ ] 用文档化的语句注明缺少系统索引，并继续运行
- [ ] 在批准步骤之前不写入任何文件

---

### 用例 5：无总监门禁——review mode 无影响

**Fixture：**
- 变更在范围内（一个 Addition："add a parry window to the block mechanic"）
- `design/gdd/combat.md` 存在（在此层级自愿编写的 GDD），描述了格挡机制
- `project.yaml` 有 `modes.review_mode: full` 和 `modes.workflow: minimal`

**输入：** `/quick-design add a parry window to the block mechanic`

**预期行为：**
1. 技能运行正常流程；`review_mode` 不在其解析的键中，也没有任何分支依赖它
2. 不生成任何类型的 agent（技能没有 `Agent` 工具）
3. 规范在"May I write"之后起草、批准并写入
4. Pipeline Notes 说明按设计 quick spec 绕过 `/design-review` 和 `/review-all-gdds`，并列出何时重定向到 `/design-system`
5. 在 `workflow: minimal` 下，交接说明直接前往 `/dev-story` 而非 `/story-readiness`

**断言：**
- [ ] 不生成总监门禁或其他 agent
- [ ] `modes.review_mode: full` 不改变技能行为
- [ ] 输出说明 quick spec 绕过 `/design-review` 和 `/review-all-gdds`，并为较大变更指明 `/design-system`
- [ ] 在 `workflow: minimal` 下交接直接指向 `/dev-story`

---

### 用例 6：边界情况——系统无 GDD（默认层级）

**Fixture：**
- `project.yaml` 有 `modes.rigor: minimal`；`design/gdd/` 中没有系统 GDD
- `design/game-brief.md` 描述了跳跃；`production/epics/movement/story-002-jump.md` 以跳跃高度 5 实现它

**输入：** `/quick-design increase jump height from 5 to 6 units`

**预期行为：**
1. 归类为 Tuning 并确认
2. 上下文扫描未找到 GDD，报告"No GDD for movement — using the brief and `production/epics/movement/story-002-jump.md`"
3. 规范以 `design/game-brief.md` 作为其 GDD Reference 起草、展示，并在"May I write"之后写入
4. 不提议 GDD 更新（没有可更新的 GDD）；交接前往 `/dev-story`

**断言：**
- [ ] 技能不因没有 GDD 而停止，也不重定向到 `/design-system`
- [ ] 报告列出替代 GDD 使用的 brief 和 story
- [ ] 规范的 GDD Reference 为 `design/game-brief.md`
- [ ] 在"May I write"批准之前不写入任何内容

---

## 协议合规性

- [ ] 起草前对变更进行分类并确认；超范围变更重定向到 `/design-system`
- [ ] 规范格式与确认的类别匹配（Tuning 表；Tweak/Addition 章节；New Small System 精简 GDD）
- [ ] 在"May I write"询问之前展示完整草稿
- [ ] 写入前询问"May I write this Quick Design Spec to `design/quick-specs/[kebab-case-title]-[YYYY-MM-DD].md`?"
- [ ] GDD 修改单独询问，展示旧文本与新文本，未经批准绝不执行
- [ ] 无总监门禁；不读取 review mode
- [ ] 以交接块和下一步（`/story-readiness`，或在 `minimal` 下为 `/dev-story`）结束

---

## 覆盖说明

- New Small System 格式（Overview、Core Rules、Tuning Knobs、Acceptance Criteria、Systems Index）未经 fixture 测试。
- 批准步骤的 `[B] Revise` 循环未单独测试。
- `guided` 和 `autonomous` 自动化模式未测试。
- 范围阈值是主观判断；技能的类别定义和重定向列表是权威定义，不通过计算工时来测试。