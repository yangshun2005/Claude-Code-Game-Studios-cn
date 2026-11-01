# 技能测试规范：/propagate-design-change

## 技能摘要

`/propagate-design-change` 处理 GDD 修订向架构的级联影响。
给定一个变更的 GDD 路径，它向 git 询问变更内容（`git diff HEAD`，然后最后一次提交），
总结变更的章节，检查实体注册表以找出需要重新检查的下游 GDD，然后找出所有引用该 GDD 的 ADR ——
通过扫描 `## GDD Requirements Addressed` 表格，加上对 GDD 基名的散文 grep ——
并且只读取受影响的那一组。每个受影响的 ADR 被分类为仍然有效 / 需要审查 / 可能被取代，
并且在任何操作之前显示完整的设计变更影响报告。

在 `full` 审查模式下，TD-CHANGE-IMPACT 门禁（技术总监）在解决之前审查报告；
在 `lean`/`solo` 下则跳过并附注说明。然后用户按 ADR 决定（标记为已取代 / 就地更新 / 保持原样 / 跳过），
每个文件变更 —— ADR 状态行、可追溯性索引、
`docs/architecture/change-impact-[date]-[system].md` 报告 —— 都有各自的询问。
级联范围遵循变更系统的工作流层级（`full` 所有 ADR，
`standard` 关键 ADR 加上引用该 GDD 的 ADR，`minimal` 不适用）。
判定：COMPLETE（影响报告已保存）或 BLOCKED（用户拒绝写入）。

---

## 静态断言（结构性）

由 `/skill-test spec` 针对 SKILL.md 检查 —— 无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 对其更改的每个文件包含 "May I" 写入前询问语言
- [ ] 末尾有下一步交接（`/architecture-decision`、`/architecture-review`）
- [ ] 记录在任何更改之前显示影响报告
- [ ] 记录门禁 TD-CHANGE-IMPACT：在 full 中运行，在 lean/solo 中跳过

---

## 总监门禁检查

一个门禁：**TD-CHANGE-IMPACT**（`technical-director`，
`.claude/docs/director-gates/td-change-impact.md`），在设计变更影响报告呈现之后、
按 ADR 解决工作流之前生成。

- `full` → 以完整影响报告（变更摘要、分类、建议操作）生成
- `lean` → 跳过；输出注明 "TD-CHANGE-IMPACT skipped — Lean mode."
- `solo` → 跳过；输出注明 "TD-CHANGE-IMPACT skipped — Solo mode."

判定处理：APPROVE → 解决工作流；CONCERNS → `AskUserQuestion` 提供
`Revise the impact assessment` / `Accept with noted concerns` / `Discuss further`；
REJECT → 不解决；先重新分析影响；NOT ASSESSED [missing input] → 绝不读作 APPROVE：
提供输入并重新运行门禁，或者运行继续并在报告和判定行中声明
`TD-CHANGE-IMPACT: NOT ASSESSED — [input]`。

---

## Fixture 默认值

每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和
`modes.automation: collaborative`，没有 `project.local.yaml`，除非其自身的
Fixture 另有说明。规则：`studio-tests/CLAUDE.md` §
Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径 —— GDD 公式变更影响两个引用 ADR 中的一个

**Fixture：**
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: lean`
- `design/gdd/combat.md` 在其 Formulas 章节有一个未提交的伤害公式编辑
- `docs/architecture/` 有 4 个 ADR；ADR-0003 和 ADR-0005 在
  `## GDD Requirements Addressed` 中列出 `combat.md`；ADR-0003 的决策假设旧公式，
  ADR-0005 的决策不依赖它
- `docs/architecture/requirements-traceability.md` 存在

**输入：** `/propagate-design-change design/gdd/combat.md`

**预期行为：**
1. 验证文件存在；运行 `git diff HEAD -- design/gdd/combat.md` 并从
   hunk 推导出变更的章节
2. 打印变更摘要（变更 / 未变更章节、关键变更）
3. Glob ADR（N = 4），grep 需求表和 `combat`
   基名，并报告 "Loaded 4 ADRs by scan. 2 reference combat.md (…)"
4. 读取 ADR-0003 和 ADR-0005（先用 `wc -c` 检查大小）并分类
   ADR-0003 为需要审查或可能被取代，ADR-0005 为仍然有效
5. 呈现设计变更影响报告
6. 注明 "TD-CHANGE-IMPACT skipped — Lean mode."
7. 询问 "ADR-0003 ([title]) — [status]. What would you like to do?" 提供四个
   选项；不对 ADR-0005 发出解决询问
8. 询问 "May I update the traceability index?" 和 "May I write the change impact
   report to `docs/architecture/change-impact-[date]-combat.md`?"
9. 判定：COMPLETE —— 变更影响报告已保存

**断言：**
- [ ] 变更内容取自 `git diff`，而非读取两个完整文档
- [ ] 只有 2 个引用 ADR 被完整读取；另外 2 个不被描述为已验证不受影响
- [ ] 在任何解决询问或写入之前显示完整影响报告
- [ ] 对每个需要审查 / 可能被取代的 ADR 逐一发出解决询问
- [ ] 每个文件变更（ADR 状态、可追溯性索引、影响报告）都有各自的询问
- [ ] 出现 "TD-CHANGE-IMPACT skipped — Lean mode."
- [ ] 影响报告写入后判定为 COMPLETE

---

### 用例 2：零匹配 —— "无影响" 与 "无法追溯"

**Fixture（表格存在）：**
- `project.yaml`：`modes.workflow: full`
- `design/gdd/combat.md` 有一个未提交的编辑
- 3 个 ADR，每个都有 `## GDD Requirements Addressed` 章节，但表格和散文中
  都没有命名 `combat.md`

**输入（两个 fixture）：** `/propagate-design-change design/gdd/combat.md`

**预期行为（表格存在）：**
1. 两次扫描都返回 0，N = 3
2. `## GDD Requirements Addressed` 的文件匹配检查非空
3. 报告 "No ADR references combat.md — no architecture impact."

**Fixture（表格缺失）：**
- 相同的 GDD 编辑；3 个 ADR，都不包含 `## GDD Requirements Addressed` 章节，
  也都不命名 `combat.md`

**预期行为（表格缺失）：**
1. 两次扫描都返回 0，N = 3；章节检查也为空
2. 报告 "3 ADRs found, none contains a 'GDD Requirements Addressed' section —
   traceability cannot be computed (a `gate-pre-production` blocker). Run
   `/architecture-decision retrofit [adr]`."

**断言：**
- [ ] 表格存在时，打印 "no architecture impact" 消息
- [ ] 表格缺失时，技能不报告 "no impact" —— 它报告无法计算可追溯性，并命名 `/architecture-decision retrofit [adr]`，即该技能解析的形式
- [ ] 两个 fixture 中都不发生按 ADR 的解决询问，也不发生 ADR 编辑
- [ ] 未找到引用时技能不报错或崩溃

---

### 用例 3：边缘情况 —— 有历史的 GDD 上 diff 为空

**Fixture：**
- `project.yaml`：`modes.workflow: full`
- `design/gdd/combat.md` 已提交，没有未提交的更改，且未被
  最后一次提交触及

**输入：** `/propagate-design-change design/gdd/combat.md`

**预期行为：**
1. `git diff HEAD -- design/gdd/combat.md` 为空
2. `git diff HEAD~1 HEAD -- design/gdd/combat.md` 也为空
3. 报告 "no uncommitted or last-commit changes to `design/gdd/combat.md`" 并
   询问要传播哪个修订
4. 不报告 "no impact"，且在选定修订之前不扫描 ADR

**断言：**
- [ ] 两次 diff 都尝试，先工作树，然后最后一次提交
- [ ] 打印 "no uncommitted or last-commit changes" 报告
- [ ] 询问用户要传播哪个修订
- [ ] 空 diff 不被报告为 "no impact"

---

### 用例 4：边缘情况 —— 未提供参数

**Fixture：**
- `design/gdd/` 中存在多个 GDD

**输入：** `/propagate-design-change`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 技能失败并提示："Usage: `/propagate-design-change design/gdd/[system].md`
   Provide the path to the GDD that was changed."
3. 不执行 diff、扫描或分析

**断言：**
- [ ] 未提供参数时技能输出用法消息
- [ ] 用法示例显示 `design/gdd/[system].md` 路径格式
- [ ] 没有目标 GDD 时不执行 `git diff` 或 ADR 扫描
- [ ] 技能不会在没有用户输入的情况下静默选择 GDD

---

### 用例 5：总监门禁 —— TD-CHANGE-IMPACT 在 full 模式下返回 CONCERNS

**Fixture：**
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: full`
- 一个 GDD 编辑使一个引用 ADR 被分类为需要审查
- TD-CHANGE-IMPACT 返回 CONCERNS：第二个 ADR 被分类过低

**输入：** `/propagate-design-change design/gdd/[system].md`

**预期行为：**
1. 呈现设计变更影响报告
2. 通过 `Agent` 生成 `technical-director`，门禁 TD-CHANGE-IMPACT，传递
   完整报告
3. 呈现 CONCERNS，命名被标记的 ADR
4. `AskUserQuestion`：`Revise the impact assessment` / `Accept with noted concerns`
   / `Discuss further`
5. 按 ADR 的解决工作流仅在该回答之后开始

**断言：**
- [ ] TD-CHANGE-IMPACT 在影响报告显示之后、任何解决询问之前生成
- [ ] 门禁接收完整报告（变更摘要、分类、建议操作）
- [ ] CONCERNS 命名被标记的 ADR 并提供全部三个选项
- [ ] 在 CONCERNS 被回答之前不写入任何 ADR、可追溯性或报告文件
- [ ] `full` 模式下不出现 "TD-CHANGE-IMPACT skipped" 注释

---

### 用例 6：工作流层级 —— `minimal` 在级联前停止；`standard` 缩小范围

**Fixture（minimal）：**
- `project.yaml` 未设置 `modes` 键 —— `modes.rigor` 默认为 `minimal`，因此
  工作流解析为 `minimal`
- `design/gdd/loot.md` 有一个未提交的编辑

**Fixture（standard）：**
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: solo`
- `design/gdd/loot.md` 有一个未提交的编辑
- 4 个 ADR，按其 Engine Compatibility `Layer`：ADR-0001（Foundation，事件总线，
  未提及 `loot.md`）、ADR-0002（Foundation，存档系统，在
  `## GDD Requirements Addressed` 中列出 `loot.md`）、ADR-0006（Feature，掉落表，仅在散文中
  命名 `loot.md`）、ADR-0007（Feature，敌人 AI，未提及）

**输入（两个 fixture）：** `/propagate-design-change design/gdd/loot.md`

**预期行为（minimal）：**
1. diff 和变更摘要照常运行
2. 在 ADR 步骤："No ADR cascade at minimal workflow — design change recorded;
   no architecture impact analysis." 技能停止

**预期行为（standard）：**
1. 范围内集合是 Foundation ADR 加上任何引用 `loot.md` 的 ADR：
   ADR-0001、ADR-0002 和 ADR-0006，因此 N = 3；ADR-0007 超出范围
2. 报告 "Loaded 3 ADRs by scan. 2 reference loot.md (1 via requirements table,
   1 via prose reference only)."
3. 仅完整读取 ADR-0002 和 ADR-0006；注明 "TD-CHANGE-IMPACT skipped — Solo mode."

**断言：**
- [ ] Minimal：不 glob 或读取任何 ADR，不生成或注明任何门禁，也不发出写入询问
- [ ] Standard：ADR-0007 不被计数或读取，且它和 ADR-0001 都不被描述为已验证不受影响
- [ ] Standard：ADR-0006 中仅散文的引用被基名 grep 捕获
- [ ] Standard：Foundation ADR 从 `**Layer**` 行找到（`Grep pattern="\*\*Layer\*\*" glob="docs/architecture/adr-*.md"`），而非从标题猜测
- [ ] 变体 —— ADR-0007 没有 `**Layer**` 行：它被视为关键（存疑时视为关键）并加入范围内集合，因此 N = 4

---

## 协议合规性

- [ ] 在生成影响报告之前 diff GDD 并扫描 ADR
- [ ] 在任何解决询问之前完整显示影响报告
- [ ] 按 ADR 询问解决 —— 绝不对整个集合做一个决定
- [ ] 每个文件变更前询问 "May I"；ADR 内容绝不删除，仅标记为已取代
- [ ] TD-CHANGE-IMPACT 在 `full` 中运行，在 `lean`/`solo` 中跳过并附命名注释
- [ ] 以与解决相匹配的后续操作结束（被取代的 ADR 用 `/architecture-decision`，多个受影响时用 `/architecture-review`）

---

## 覆盖说明

- 通过 `system_overrides` 在 `standard` 项目上固定为 `minimal` 的系统
  （其变更为 N/A）没有自己的 fixture；用例 6 通过项目默认值达到 `minimal`。
- 通过 `design/registry/entities.yaml` 的下游 GDD 影响未独立
  fixture 测试。
- 没有 git 历史的 GDD（"appears to be a new GDD, not a revision"）和
  不存在的路径未单独测试。
- TD-CHANGE-IMPACT 的 APPROVE、REJECT 和 NOT ASSESSED 路径未单独测试。
- 故事和史诗超出此技能的范围 —— 它仅级联到 ADR 和
  可追溯性索引。