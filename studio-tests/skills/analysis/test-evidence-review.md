# Skill 测试规范：/test-evidence-review

## Skill 摘要

`/test-evidence-review` 审查故事背后测试文件和证据文档的**质量**——`/smoke-check` 只覆盖存在性和通过情况。它接受一个范围：单个故事路径、`sprint`（`production/sprints/` 中最近的计划）或系统名称（`production/epics/[system]/story-*.md`）；无参数时询问要审查哪个范围。若解析出的范围包含零个故事，则以 `NOT ASSESSED — no stories in scope` 停止，并路由到创建故事的 Skill。对每个故事，它用定向 grep 收集故事类型、证据路径和验收标准，定位证据（`tests/unit/[system]/`、`tests/integration/[system]/`、`production/qa/evidence/`、`production/qa/smoke-*.md`），并审查自动化测试的断言数量、边界情况、命名和公式可追溯性，以及人工证据的标准关联、签核、保留截图和时效性。它按 `/story-done` 的方式解析 `qa.level` 和 `testing.strict`：在 `qa.level: minimal` 下，Logic、Integration 或 Config/Data 故事若无测试证据则为 `WAIVED (qa.level: minimal)`，而非 MISSING；已存在测试中的缺口为 ADVISORY；Visual/Feel 和 UI 证据永不豁免；否则每个缺口按故事类型从 `testing.strict.<type>` 取门禁级别（未设置 → 纯布尔 `testing.strict` 旧值适用于所有类型；否则用 `coding-standards.md` 默认值）。每个故事得到 ADEQUATE、INCOMPLETE、MISSING、NOT ASSESSED 或 WAIVED；总体判定为最差的现存判定，其中 NOT ASSESSED 仅高于 ADEQUATE，WAIVED 低于 ADEQUATE。报告在对话中呈现，Skill 作为可选步骤询问"May I write this test evidence review to `production/qa/evidence-review-[date].md`?"。它从不修改测试文件或证据文档，也不调用任何主管门禁。运行以 COMPLETE 结束，发现 BLOCKING 项时以 CONCERNS 结束。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：ADEQUATE、INCOMPLETE、MISSING、NOT ASSESSED，以及运行级 COMPLETE / CONCERNS
- [ ] 对可选的 `production/qa/evidence-review-[date].md` 包含"May I write"措辞
- [ ] 具有下一步交接（第 7 节：`/story-done` 前的 BLOCKING 项、`/test-helpers`、签核路由）
- [ ] `resolve_config` 注入命名了 `qa.level` 和 `testing.strict`，且 Skill 说明了 `WAIVED (qa.level: minimal)` 结果和按类型的 `testing.strict.<type>` 门禁级别

---

## 主管门禁检查

无。测试证据审查是咨询性质量 Skill；不调用任何门禁。

---

## 夹具默认值

除非自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——单个 Logic 故事，测试质量良好

**夹具：**
- `project.yaml`：`engine.name: godot`，因此单元测试根为 `tests/unit/`；`modes.automation` 未设置（collaborative）
- `production/epics/combat/story-001-damage-calc.md`（slug `damage-calc`）：`## Test Evidence` 给出故事类型 Logic 和路径 `tests/unit/combat/damage-calc_test.gd`；`## Acceptance Criteria` 包含"damage is never below 1"和"critical hits multiply damage by 2.0"
- `tests/unit/combat/damage-calc_test.gd` 开头为 `# Story: production/epics/combat/story-001-damage-calc.md`，有 4 个测试函数，每个 3+ 断言，命名为 `test_damage_calc_zero_armor_returns_base_damage` 和 `test_damage_calc_max_crit_doubles_damage` 之类，并在注释中引用了 GDD 的 `damage_formula`

**输入：** `/test-evidence-review production/epics/combat/story-001-damage-calc.md`

**预期行为：**
1. 单故事模式：Skill 读取故事的 Test Evidence 和 Acceptance Criteria
2. 第 3 节在 `tests/unit/combat/` 下找到测试文件
3. 第 4 节：断言覆盖正常（每函数 3+）；"zero"和"max"用例覆盖数值标准；命名遵循 `test_[scenario]_[expected]`；公式被引用
4. 故事判定：**ADEQUATE**；总体判定 ADEQUATE；0 个 BLOCKING 项
5. 报告在对话中呈现，然后 Skill 作为可选步骤询问"May I write this test evidence review to `production/qa/evidence-review-[date].md`?"
6. 判定：**COMPLETE**

**断言：**
- [ ] 对 Logic 故事，测试文件在 `tests/unit/combat/` 下被定位
- [ ] 断言覆盖、边界情况、命名和公式可追溯性均被报告
- [ ] 故事判定为 ADEQUATE
- [ ] 报告在可选的"May I write"提示之前显示
- [ ] 运行以 COMPLETE 结束

---

### 用例 2：空洞和单薄测试——INCOMPLETE，含 BLOCKING 项

**夹具：**
- `project.yaml`：`engine.name: godot`，`modes.rigor: standard`（因此 `qa.level: standard`——要求测试）；无 `testing.strict` 块
- `production/epics/combat/story-002-crit-roll.md`（slug `crit-roll`）：故事类型 Logic
- `tests/unit/combat/crit-roll_test.gd` 有 `func test_1()` 无断言，以及 `func test_crit_roll_applies_multiplier()` 一个断言

**输入：** `/test-evidence-review production/epics/combat/story-002-crit-roll.md`

**预期行为：**
1. Skill 找到测试文件并统计每函数断言数
2. `test_1` 有 0 断言 → 标记 BLOCKING（空洞通过，什么也没证明）；`test_1` 也被标记为通用命名
3. 单断言函数被记为可能单薄
4. 故事判定：**INCOMPLETE**
5. 后续："These must be resolved before `/story-done` can mark the story Complete…"和"Consider running `/test-helpers [system]`…"
6. 因发现 BLOCKING 项，运行以 **CONCERNS** 结束

**断言：**
- [ ] 零断言的测试函数被标记 BLOCKING
- [ ] `test_1` 被标记为命名问题
- [ ] 故事判定为 INCOMPLETE，而非 ADEQUATE
- [ ] 输出说明 BLOCKING 项必须在 `/story-done` 前解决，并建议 `/test-helpers`
- [ ] 运行以 CONCERNS 结束
- [ ] 测试文件未被修改
- [ ] 变体——`project.yaml` 中 `testing.strict.logic: false` 时，零断言发现为 ADVISORY，故事仍为 INCOMPLETE，运行以 COMPLETE 结束

---

### 用例 3：最小工作流下的空 Sprint 范围——NOT ASSESSED，而非干净报告

**夹具：**
- `project.yaml`：`modes.rigor: minimal`（解析出的块显示 `workflow: minimal`）
- `production/sprints/` 不存在

**输入：** `/test-evidence-review sprint`

**预期行为：**
1. Sprint 模式找不到 sprint 计划，因此范围包含零个故事
2. Skill 停止并报告 `NOT ASSESSED — no stories in scope`，指明 sprint 范围和空的 `production/sprints/` 路径
3. 因工作流为 minimal（无 sprint），路由到 `/test-evidence-review [epic-slug]` 而非 `/sprint-plan new`
4. Skill 不继续到第 3 节；不渲染 Summary 表，也不渲染"BLOCKING items: 0"行

**断言：**
- [ ] 判定为 NOT ASSESSED — no stories in scope
- [ ] 输出指明搜索了哪个范围、哪个路径为空
- [ ] 最小工作流下路由为 `/test-evidence-review [epic-slug]`，而非 `/sprint-plan new`
- [ ] 不产生"BLOCKING items: 0 / ADVISORY items: 0"的空报告

---

### 用例 4：故事类型未知——单故事 NOT ASSESSED 拉高总体判定

**夹具：**
- `project.yaml`：`engine.name: godot`
- `production/epics/combat/` 包含 `story-001-damage-calc.md`（Logic，测试同用例 1）和 `story-003-hit-feedback.md`，其 `## Test Evidence` 节未说明故事类型和证据路径
- 完整阅读 `story-003-hit-feedback.md` 也未揭示类型

**输入：** `/test-evidence-review combat`

**预期行为：**
1. 系统模式 glob `production/epics/combat/story-*.md` 并找到两个故事
2. `story-001-damage-calc` 为 ADEQUATE
3. `story-003-hit-feedback` 为 **NOT ASSESSED**，并说明原因：无法确定故事类型，因此所需证据未知
4. 不报告为 MISSING
5. 总体判定：**NOT ASSESSED**——它高于 ADEQUATE

**断言：**
- [ ] 无类型故事为 NOT ASSESSED，而非 MISSING
- [ ] 为该故事说明 NOT ASSESSED 原因（无法确定类型）
- [ ] 总体判定为 NOT ASSESSED，而非 ADEQUATE
- [ ] 有类型故事仍被审查并报告为 ADEQUATE

---

### 用例 5：Visual/Feel 证据无保留截图，完整审查模式——无门禁

**夹具：**
- `project.yaml`：`engine.name: godot`，`modes.review_mode: full`
- `production/epics/combat/story-004-hit-flash.md`（slug `hit-flash`）：故事类型 Visual/Feel
- `production/qa/evidence/hit-flash-evidence.md` 引用了每个验收标准，所有签核已填写，日期在 sprint 开始之后
- `production/qa/evidence/` 中不存在该故事的 `*.png`、`*.jpg` 或 `*.gif`

**输入：** `/test-evidence-review production/epics/combat/story-004-hit-flash.md`

**预期行为：**
1. 第 3 节找到 `production/qa/evidence/hit-flash-evidence.md`
2. 第 5 节：标准关联和签核完整，但该故事无保留图像
3. 文档描述了视觉检查却无证据 → 故事判定 **INCOMPLETE**，标记 BLOCKING
4. 无论审查模式如何，不调用主管门禁
5. 证据文档未被编辑
6. 运行以 **CONCERNS** 结束

**断言：**
- [ ] 无保留图像的 Visual/Feel 证据文档为 INCOMPLETE，即使签核完整
- [ ] 缺失截图列为 BLOCKING 问题
- [ ] 任何审查模式下都不调用主管门禁
- [ ] 证据文档未被修改

---

### 用例 6：UI 故事由其截图关闭——无需签核

**夹具：**
- `project.yaml`：`engine.name: godot`
- `production/epics/ui/story-007-inventory-screen.md`（slug `inventory-screen`）：故事类型 UI；其
  `## Test Evidence` 命名 `production/qa/evidence/inventory-screen-populated.png` 和
  `production/qa/evidence/inventory-screen-empty.png`——它涉及的每个屏幕状态各一张
- 两张图像都存在，日期在 sprint 开始之后
- 不存在 `inventory-screen-evidence.md`、无签核记录、无走查日志

**输入：** `/test-evidence-review production/epics/ui/story-007-inventory-screen.md`

**预期行为：**
1. 第 3 节先使用故事声明的两个路径，再搜索
2. 第 5 节：UI 故事由它涉及的每个屏幕的保留截图关闭
3. 故事判定 **ADEQUATE**；运行以 **COMPLETE** 结束

**断言：**
- [ ] 先检查故事声明的证据路径，两张图像都被找到
- [ ] 缺失签核不使故事 INCOMPLETE——UI 故事无需签核
- [ ] 不要求走查日志
- [ ] 故事判定为 ADEQUATE

---

### 用例 7：Unity Logic 故事——证据在声明的引擎路径

**夹具：**
- `project.yaml`：`engine.name: unity`
- `production/epics/inventory/story-003-stack-merge.md`：故事类型 Logic；其
  `## Test Evidence` 命名 `Assets/Tests/EditMode/InventoryTests.cs`
- 该文件存在，每个验收标准一个 `[Test]`（共 4 个），每个有 3 个或更多 `Assert.That` 调用，命名为 `[Scenario]_[Expected]` 形式（`MergeStacks_FullStack_SpillsToNewSlot`），覆盖空栈和满栈用例
- inventory GDD 无 Formulas 节，因此公式可追溯性不适用
- 任何位置都不存在名为 `stack-merge_test.*` 的文件，`tests/unit/` 不存在

**输入：** `/test-evidence-review production/epics/inventory/story-003-stack-merge.md`

**预期行为：**
1. 第 3 节在 `Assets/Tests/EditMode/` 下声明的路径找到测试
2. 第 4 节审查其断言；故事判定 **ADEQUATE**

**断言：**
- [ ] 测试在声明的 Unity 路径被找到——故事不报告为 MISSING
- [ ] 审查不要求 `[story-slug]_test.*` 命名或 `tests/unit/` 位置

---

### 用例 8：Logic 故事无测试——MISSING，且高于 NOT ASSESSED

**夹具：**
- `project.yaml`：`engine.name: godot`，`modes.rigor: standard`（因此 `qa.level: standard`——Logic 故事的测试是必需的）
- `production/epics/combat/` 包含 `story-003-hit-feedback.md`（无故事类型，同用例 4）和 `story-005-knockback.md`：故事类型 Logic，未声明证据路径
- 不存在其测试：`tests/unit/combat/` 不存在，且无其他测试文件命名 slug `knockback` 或引用该故事

**输入：** `/test-evidence-review combat`

**预期行为：**
1. `story-005-knockback` 是 Logic 故事，因此需要单元测试；搜索单元测试根未找到
2. `story-005-knockback` 为 **MISSING**——审查已运行且未找到——缺口为 BLOCKING 项
3. `story-003-hit-feedback` 为 **NOT ASSESSED**（类型无法确定），而非 MISSING
4. 总体判定：**MISSING**——已知失败高于无法评估的故事
5. 因发现 BLOCKING 项，运行以 **CONCERNS** 结束

**断言：**
- [ ] 无测试文件的 Logic 故事为 MISSING，而非 NOT ASSESSED
- [ ] 无类型故事为 NOT ASSESSED，而非 MISSING——两者被区分开
- [ ] 总体判定为 MISSING，而非 NOT ASSESSED
- [ ] 缺失测试列为 BLOCKING，运行以 CONCERNS 结束

---

### 用例 9：`qa.level: minimal`——Logic 故事无测试为 WAIVED；UI 截图则不然

**夹具：**
- `project.yaml`：`engine.name: godot`，无 `modes` 块（因此 `qa.level: minimal`——测试被豁免）
- `production/epics/combat/` 包含 `story-005-knockback.md`（Logic，任何位置无测试，同用例 8）和 `story-008-pause-menu.md`（UI，`production/qa/evidence/` 中无其保留图像）

**输入：** `/test-evidence-review combat`

**预期行为：**
1. `story-005-knockback` 为 **WAIVED (qa.level: minimal)**——而非 MISSING——且不产生 BLOCKING 项
2. `story-008-pause-menu` 为 **MISSING** 且 BLOCKING——UI 故事所需的保留截图永不豁免
3. 总体判定：**MISSING**；报告头部显示 `qa.level: minimal`
4. 仅因 UI 故事，运行以 **CONCERNS** 结束
5. 变体——无 UI 故事时，总体判定为 **WAIVED**，0 个 BLOCKING 项，运行以 **COMPLETE** 结束

**断言：**
- [ ] 无测试的 Logic 故事为 WAIVED