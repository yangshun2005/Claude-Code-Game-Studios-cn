# 技能测试规范：/ux-review

## 技能摘要

`/ux-review` 在 UX 文档进入实现流水线之前对其进行校验。它接受一个文件路径、`hud`（`design/ux/hud.md`）、`patterns`（`design/ux/interaction-patterns.md`）、`all`（`design/ux/` 下的每个文件，先输出汇总表），或不带参数（此时它会询问要校验哪份规范）。

阶段 2 加载交叉引用上下文：来自 `project.yaml` `platform` 块的输入方式（输入覆盖检查的权威依据）、`design/accessibility-requirements.md` 中承诺的无障碍等级、模式库、规范头部所列 GDD 的 UI 需求，以及玩家旅程。随后按文档类型运行检查清单——UX 规范用阶段 3A（15 项完整性检查加质量检查），HUD 设计用 3B，模式库用 3C——并输出结构化报告。类型取自文件的 `> **Template**:` 头部行（由 `/ux-design` 写入）；没有该行的文件按文件名分类（`hud.md` → 3B，`interaction-patterns.md` → 3C，其余 → 3A），报告会说明假定了哪份检查清单。

判定结果：**APPROVED**、**NOT ASSESSED**（规范无法读取，或某个维度没有判定标准——例如未承诺无障碍等级）、**NEEDS REVISION**、**MAJOR REVISION NEEDED**。该技能为只读（`allowed-tools: Read, Glob, Grep`，外加一次 `resolve_config` 调用，用于为模式库规则解析 `workflow`），不生成任何 agent，其判定仅供参考。交接：APPROVED → `/team-ui` 阶段 2；NEEDS REVISION → 修复后重新运行 `/ux-review`；MAJOR REVISION NEEDED → 返回 `/ux-design`。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：`APPROVED`、`NOT ASSESSED`、`NEEDS REVISION`、`MAJOR REVISION NEEDED`
- [ ] 不包含"May I write"措辞，且 `allowed-tools` 中没有 Write 或 Edit（技能为只读）
- [ ] 具有针对具体判定的下一步交接（`/team-ui`、NOT ASSESSED 时缺失的输入、重新运行 `/ux-review`、`/ux-design`）

---

## 总监门禁检查

无。`/ux-review` 本身就是 UX 规范的评审步骤；它没有 `Agent` 工具，也不调用任何总监门禁。

---

## Fixture 默认值

除非某个用例自身的 Fixture 另有说明，否则每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——完整的 UX 规范，APPROVED

**Fixture：**
- `design/ux/inventory.md` 遵循 `.claude/docs/templates/ux-spec.md`：头部包含 Status、Author、Platform Target（PC、Console）、Related GDDs（`design/gdd/inventory.md`）、Accessibility Tier 和 `> **Template**: UX Spec`；每个章节都已填写，包括 States & Variants（loading、empty、populated、error）、带焦点顺序的键盘和 d-pad 导航、针对键盘/鼠标和游戏手柄的 Input Method Completeness Checklist 块（每一项都已勾选），以及至少 5 条可测试的验收标准
- `design/accessibility-requirements.md` 承诺 Standard 等级
- `design/ux/interaction-patterns.md` 存在，且规范中每个交互组件都引用了其中的模式
- `project.yaml` `platform` 块：`targets: [PC, Console]`、`gamepad_support: Full`
- `design/gdd/inventory.md` 的 UI 需求均被规范覆盖

**输入：** `/ux-review design/ux/inventory.md`

**预期行为：**
1. 阶段 2 从 `platform` 块推导输入方式（键盘/鼠标、游戏手柄），读取已承诺的等级、模式库（若存在）以及 `design/gdd/inventory.md` 的 UI 需求
2. 阶段 3A 运行完整性检查和各项质量检查
3. 报告将阶段 3A 列为所用检查清单（来自 Template 行），并显示"Completeness: [X/Y sections present]"、Quality Issues、"GDD Alignment: ALIGNED"、对照 Standard 等级的"Accessibility: COMPLIANT"，以及"Pattern Library: CONSISTENT"
4. 判定：APPROVED——"This spec is ready for handoff to `/team-ui` Phase 2 (Visual Design)."
5. 阶段 5 建议运行 `/team-ui`

**断言：**
- [ ] 输入方式覆盖是依据 `project.yaml` platform 块检查的，而不仅是规范头部
- [ ] 使用了 UX 规范检查清单（阶段 3A），包括 States & Variants、Interaction Map 覆盖和 Input Method Completeness Checklist 项
- [ ] 每个评审维度都有可对照检查的依据——已承诺的等级、模式库、引用的 GDD——因此没有维度是 NOT ASSESSED
- [ ] 无障碍是依据 `design/accessibility-requirements.md` 中承诺的等级检查的
- [ ] 判定为 APPROVED，并交接至 `/team-ui`
- [ ] 不写入任何文件

---

### 用例 2：无障碍章节为空——NEEDS REVISION

**Fixture：**
- 与用例 1 相同，但规范的 Accessibility 章节为空
- `design/accessibility-requirements.md` 承诺 Standard 等级

**输入：** `/ux-review design/ux/inventory.md`

**预期行为：**
1. Accessibility Requirements 完整性项失败，Standard 等级的文本对比度检查无法满足；焦点顺序检查仍然通过，因为规范仍定义了键盘和 d-pad 焦点顺序（用例 1 的 fixture）
2. 每个发现都列为编号质量问题，包含 BLOCKING/ADVISORY、"What's wrong"、"Where" 和 "Fix"
3. 判定：NEEDS REVISION——"Address the [N] blocking issues above, then re-run `/ux-review`."
4. 技能会主动提出帮助起草缺失内容，但不编辑文件；它等待用户

**断言：**
- [ ] 返回 NEEDS REVISION（而非 APPROVED 或 MAJOR REVISION NEEDED）
- [ ] 发现项指明章节并给出具体修复
- [ ] 交接为修复问题并重新运行 `/ux-review`
- [ ] 技能提供帮助但不自动修复，且不写入任何文件

---

### 用例 3：状态不完整——NEEDS REVISION 优先于 NOT ASSESSED

**Fixture：**
- `design/ux/settings-menu.md` 遵循 UX 规范模板，带有 `> **Template**: UX Spec`，其余部分完整
- 其 States & Variants 章节只记录了 populated 状态——没有 loading、empty 或 error 状态；该屏幕异步获取数据
- `design/ux/interaction-patterns.md` 存在，且规范的组件引用了其中的模式
- 没有 `design/accessibility-requirements.md`，因此没有已承诺的等级

**输入：** `/ux-review design/ux/settings-menu.md`

**预期行为：**
1. States & Variants 完整性项失败（"at minimum: loading, empty/populated, and error states"）
2. Completeness 列表将其标出，例如"States & Variants — MISSING: error state not documented"，并列出每个缺失的状态（loading、empty、error）
3. 报告显示"Accessibility: NOT ASSESSED — no committed tier (design/accessibility-requirements.md absent)"，并将其列在"Dimensions not assessed"下
4. 判定为 NEEDS REVISION：已发现的缺口优先于无法检查的维度。它不是 NOT ASSESSED，对于这个可修复的缺口也不是 MAJOR REVISION NEEDED
5. 交接为修复缺口并重新运行 `/ux-review`

**断言：**
- [ ] 返回 NEEDS REVISION，而非 NOT ASSESSED——NOT ASSESSED 的优先级低于修订类判定
- [ ] 未评估的 Accessibility 维度仍在报告中列出
- [ ] 缺失的 loading、empty 和 error 状态均在输出中列出
- [ ] 对于这个可修复的缺口，不返回 MAJOR REVISION NEEDED
- [ ] 交接为修复后重新运行 `/ux-review`

---

### 用例 4：无可对照评估的依据——NOT ASSESSED

**Fixture：**
- 场景 (a)：`design/ux/inventory-screen.md` 不存在
- 场景 (b)：`design/ux/inventory.md` 完整（如用例 1），但 `design/accessibility-requirements.md` 不存在；规范头部声明"Accessibility Tier: Standard"

**输入：** (a) `/ux-review design/ux/inventory-screen.md` (b) `/ux-review design/ux/inventory.md`

**预期行为：**
1. (a) 规范无法读取：判定为 NOT ASSESSED，列出缺失的路径，不生成检查清单
2. (b) 报告显示"Accessibility: NOT ASSESSED — no committed tier (design/accessibility-requirements.md absent)"——绝不显示 COMPLIANT
3. (b) 头部的 Standard 等级被沿用，并作为假设而非承诺陈述
4. (b) 建议运行 `/ux-design accessibility` 来确立等级
5. 判定为 NOT ASSESSED，列出未评估的维度以及使其可检查所需的输入，且不建议交接至 `/team-ui`

**断言：**
- [ ] (a) 规范缺失时返回 NOT ASSESSED 并列出路径，不输出检查清单
- [ ] (b) 未承诺等级时，Accessibility 报告为 NOT ASSESSED，而非 COMPLIANT
- [ ] (b) 建议运行 `/ux-design accessibility`
- [ ] 报告不建议在 NOT ASSESSED 结果下交接至 `/team-ui`
- [ ] 不写入任何文件

---

### 用例 5：`all` 参数与无总监门禁

**Fixture：**
- `design/ux/` 包含 `hud.md`（由 `/ux-design hud` 写入，头部行 `> **Template**: HUD Design`）、`inventory.md`（由 `/ux-design inventory` 写入，`> **Template**: UX Spec`）和 `pause-menu.md`（依据 `.claude/docs/templates/ux-spec.md` 手写，没有 Template 行）
- `project.yaml` 有 `modes.review_mode: full`

**输入：** `/ux-review all`

**预期行为：**
1. 技能找到 `design/ux/` 下的每个文件，先输出汇总表（file | verdict | primary issue），然后输出每个文件的完整报告
2. `hud.md` 用 HUD 检查清单（阶段 3B）检查，`inventory.md` 用 UX 规范检查清单（阶段 3A）检查，各自依据其 Template 行
3. `pause-menu.md` 没有 Template 行，且不是 `hud.md` 或 `interaction-patterns.md`，因此使用阶段 3A，其报告说明检查清单是根据文件名假定的
4. 不生成任何 agent，不出现门禁 ID 或门禁跳过消息；评审模式不起作用
5. 每个判定都是 APPROVED / NOT ASSESSED / NEEDS REVISION / MAJOR REVISION NEEDED 之一

**断言：**
- [ ] 汇总表（file | verdict | primary issue）出现在逐文件详情之前
- [ ] 每个文档的检查清单来自其 Template 行（HUD 与 UX 规范）
- [ ] 没有 Template 行的文件按文件名分类，报告说明假定了哪份检查清单
- [ ] 不调用任何总监门禁，不出现门禁跳过消息
- [ ] 每个判定都来自该技能的四判定集合

---

## 协议合规性

- [ ] 在校验前加载交叉引用上下文（平台配置、无障碍等级、模式库、引用的 GDD、玩家旅程）
- [ ] 使用与文档类型匹配的检查清单（UX 规范、HUD、模式库）——来自 Template 行，或按文件名并说明假定
- [ ] 依据已承诺的等级检查无障碍，未承诺等级时报告 NOT ASSESSED——绝不报告 COMPLIANT
- [ ] NOT ASSESSED 的优先级高于 APPROVED，低于 NEEDS REVISION 和 MAJOR REVISION NEEDED
- [ ] 不写入或编辑任何文件
- [ ] 对 NEEDS REVISION 和 MAJOR REVISION NEEDED 给出具体发现（What's wrong / Where / Fix，BLOCKING 或 ADVISORY）；对 NOT ASSESSED，列出每个未评估的维度以及使其可检查所需的输入
- [ ] 以针对具体判定的交接结尾：`/team-ui`（APPROVED）、按维度列出缺失输入且不交接至 `/team-ui`（NOT ASSESSED）、重新运行 `/ux-review`（NEEDS REVISION）、`/ux-design`（MAJOR REVISION NEEDED）

---

## 覆盖说明

- MAJOR REVISION NEEDED（范围、玩家需求或完整性方面的根本性问题）没有专门的 fixture 测试。
- 模式库检查清单（阶段 3C，`patterns` 参数）没有 fixture 测试。
- 无参数提示（"which spec to validate"）未测试。
- 当 `project.yaml` 没有 `platform` 块时回退到 `.claude/docs/technical-preferences.md`、再回退到规范头部的行为未测试。