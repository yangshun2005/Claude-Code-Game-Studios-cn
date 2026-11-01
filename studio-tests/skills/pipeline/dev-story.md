# 技能测试规范：/dev-story

## 技能摘要

`/dev-story` 实现单个故事。它查找故事（参数，否则取 `production/session-state/active.md` 中的活跃故事），解析故事所属系统的工作流层级，按层级验证必需输入（TR 注册表、治理 ADR、控制清单），用一次 grep 读取 ADR 的 `## Status`、`## Last Verified` 和 `## Date`——`Proposed`、`Deprecated` 或 `Superseded` 状态在任何层级都会阻断故事，而 ADR 字段为 `N/A` 的故事不引用任何 ADR——并通过比较故事的 `ADR Version` 与 ADR 的当前版本（`Last Verified`，否则 `Date`，否则 `unversioned`——即 `/create-stories` 标记所用的规则）来检查 ADR 新鲜度。它检查故事的 `Manifest Version` 及其依赖，请求将故事标记为 `In Progress`，并将实现路由给程序员 agent 加项目引擎专家。它本身不写源代码，它写入的每个文件——故事字段、`sprint-status.yaml`、依赖的 `Status`、Config/Data 文件、Unity 的 `ScreenshotOnArg.cs`——都遵循一个点名该文件的询问；只有会话状态检查点是未经询问写入的。

agent 返回后，它验证 agent 已完成，运行引擎解析检查，启动构建以观察任何玩家可见的内容（`Run result:` 行），打印"Implementation Complete"摘要并覆盖 `active.md` 中的 CHECKPOINT 块，其 **Next step** 根据结果而定：Implementation Complete 后为 `/story-done`，INCOMPLETE 后为 `/dev-story` 以恢复，BLOCKED 后为解除阻断操作。它从不关闭它所实现的故事——`/story-done` 做这件事；在 `standard`/`full` 下建议先运行 `/code-review`。在 `qa.level: minimal` 下，Logic 或 Integration 故事的测试要求被豁免并打印豁免；UI 或 Visual/Feel 故事的截图从不豁免，且运行并观察步骤在每个层级都执行。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含状态关键词：BLOCKED、INCOMPLETE、In Progress、NOT VERIFIED
- [ ] 包含"May I write"协作协议语言（它对自己写入的文件自行询问；源代码和测试委托给子 agent）
- [ ] 末尾有下一步交接（`/story-done`，在 standard/full 下先 `/code-review`）
- [ ] 记录各层级的输入要求（`full` / `standard` / `minimal`）
- [ ] 注明实现委托给专家 agent（不直接执行）

---

## Director Gate 检查

无 director gate。`/dev-story` 生成实现 agent（一名主程序员和引擎专家），而非 director 审查 gate，且不解析 `review_mode`。代码审查是单独的 `/code-review` 技能，在 `standard`/`full` 下被列为下一步。

---

## Fixture 默认值

除非其自身 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——在 full 工作流下实现 Logic 故事

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`specialists.code: godot-gdscript-specialist`、`modes.workflow: full`、`qa.level: standard`
- `production/epics/combat/story-001-damage-calc.md`：`Status: Ready`、Layer Core、Type Logic、`Requirement: TR-combat-001`、`ADR Governing Implementation: ADR-0003`、`ADR Version` 等于 ADR-0003 的 `## Last Verified` 日期、`Manifest Version` 等于清单头部日期、Dependencies None、Test Evidence `tests/unit/combat/combat_damage_calc_test.gd`
- `docs/architecture/tr-registry.yaml` 含 `TR-combat-001`；ADR-0003 存在且为 Accepted；`docs/architecture/control-manifest.md` 存在
- `production/sprint-status.yaml` 不存在

**输入：** `/dev-story production/epics/combat/story-001-damage-calc.md`

**预期行为：**
1. 读取故事并从注册表中 grep `id: TR-combat-001`
2. grep ADR-0003 的 `^## (Status|Last Verified|Date)`——Status 为 Accepted 且其当前版本（`Last Verified` 日期）与故事匹配，因此不读取 ADR
3. 从清单中 grep `^## Core Layer Rules`；Manifest Version 匹配，无询问
4. 询问"May I mark this story In Progress?"，点名故事文件（而非不存在的 `sprint-status.yaml`）；确认后将故事的 `Status:` 设为 `In Progress` 及其 `Last Updated:`；打印"Sprint status not updated: production/sprint-status.yaml absent"
5. 生成 `gameplay-programmer`，以 `godot-gdscript-specialist` 为次要，简报故事路径、TR-ID 查找、故事的 ADR Decision Summary 和 Implementation Notes 内联、Core 层规则，以及带测试要求的测试文件路径
6. 确认 agent 已完成，运行 Godot 解析检查并报告其退出码
7. 打印"Implementation Complete: [title]"，含文件、测试函数数量、Verification 和一行 `Run result:`；下一步 `/code-review` 然后 `/story-done`
8. 覆盖 `active.md` 中的 `<!-- CHECKPOINT -->` 块

**断言：**
- [ ] 当 ADR-0003 的 `Last Verified` 与故事的 `ADR Version` 匹配时，不完整读取 ADR-0003
- [ ] In Progress 询问点名故事文件；`Status:` 仅在其后、任何 agent 生成之前变为 `In Progress`，且打印 sprint-status 缺失行
- [ ] 主要 agent 为 `gameplay-programmer`；次要来自 `specialists.code`，而非名册表
- [ ] ADR 指导以内联方式传递，而非 ADR 路径
- [ ] 简报点名 `tests/unit/combat/combat_damage_calc_test.gd`（`[system]_[feature]_test.gd`）并要求每个验收标准一个测试
- [ ] 报告解析检查的命令和退出码，且恰好出现一行 `Run result:`
- [ ] 故事未被标记为 Complete；交接为 `/code-review` 然后 `/story-done`
- [ ] `active.md` 中的 CHECKPOINT 块被覆盖，而非追加

---

### 用例 2：失败路径——引用的 ADR 为 Proposed

**Fixture：**
- `project.yaml`：`modes.workflow: standard`
- 故事文件含 `Status: Ready` 和 `ADR Governing Implementation: ADR-0005`
- `docs/architecture/adr-0005-*.md` 存在，`## Status` 为 Proposed

**输入：** `/dev-story production/epics/[epic-slug]/story-[NNN]-[slug].md`

**预期行为：**
1. 技能读取故事并解析出 `standard` 层级
2. 技能确定引用的 ADR-0005 为 Proposed
3. 故事在会话状态中被设为 BLOCKED；不生成程序员 agent
4. 输出点名 ADR-0005 并建议 `/architecture-decision accept ADR-0005`
5. 生成部分报告；故事文件的 `Status:` 不变

**断言：**
- [ ] Proposed 状态从 ADR-0005 的 `## Status` 节读取，而非推断
- [ ] 不生成程序员或引擎专家 agent
- [ ] 输出点名 ADR-0005 并建议 `/architecture-decision`
- [ ] 故事头部保持 `Status: Ready`，且不显示"May I mark this story In Progress?"询问
- [ ] BLOCKED 记录在会话状态中，检查点的 **Next step** 点名 `/architecture-decision accept ADR-0005`——而非 `/story-done`

---

### 用例 2b：失败路径——`full` 层级下的 Proposed ADR

文件检查表的 `full` 列有自己的 STOP 规则，因此上面的 `standard` 用例未覆盖它。

**Fixture：**
- `project.yaml`：`modes.workflow: full`
- 故事文件含 `Status: Ready`、`ADR Governing Implementation: ADR-0005`，且 `ADR Version` 早于 ADR-0005 的 `## Last Verified` 日期
- `docs/architecture/adr-0005-*.md` 存在，`## Status` 为 Proposed；`docs/architecture/tr-registry.yaml` 和控制清单存在

**输入：** `/dev-story production/epics/[epic-slug]/story-[NNN]-[slug].md`

**预期行为：**
1. 技能 grep ADR-0005 的 `^## (Status|Last Verified|Date)` 并读到 `Proposed`
2. 它在新鲜度比较之前停止："ADR [path] is still Proposed. Accept it with `/architecture-decision accept ADR-0005` before implementing."
3. 故事在会话状态中被设为 BLOCKED；不生成程序员 agent

**断言：**
- [ ] 停止发生在 `full` 层级
- [ ] Status 在新鲜度检查之前决定：尽管版本不同，不显示 ADR 版本不匹配提示，且 ADR 不超出那一次 Grep 被读取
- [ ] 不生成程序员或引擎专家 agent，故事保持 `Status: Ready`

---

### 用例 3：故事编写后 ADR 已变更

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`modes.workflow: full`
- 故事文件含 `Status: Ready`、Type Logic、`Requirement: TR-combat-002`、`ADR Governing Implementation: ADR-0004`、`ADR Version` 早于 ADR-0004 的 `## Last Verified` 日期、`Manifest Version` 等于清单头部日期、Dependencies None
- `docs/architecture/tr-registry.yaml` 含 `TR-combat-002`；ADR-0004 存在，`## Status` 为 Accepted 且小于 50KB；`docs/architecture/control-manifest.md` 存在
- 用户在提示处选择 `[A]`

**输入：** `/dev-story production/epics/[epic-slug]/story-[NNN]-[slug].md`

**预期行为：**
1. 对 ADR-0004 执行 `Grep "^## (Status|Last Verified|Date)"`，读到 Accepted 和一个与故事的 `ADR Version` 不同的 `Last Verified` 日期
2. `AskUserQuestion`："Story was written against ADR v[story-date]. The ADR is now v[current-date]. Its decision may have changed. How do you want to proceed?"，选项为 `[A] Re-read the changed ADR sections…`（点名刷新本故事的 ADR 摘要和 ADR Version）/ `[B] Implement from the story's summary…` / `[C] Stop…`
3. 选 [A]：用 `wc -c` 检查大小；小于约 50KB，用一次 `Read` 读取 ADR
4. 将故事的 `ADR Version` 设为 ADR-0004 的 `Last Verified` 日期，并用重读的指导替换其 ADR Decision Summary 和 Implementation Notes
5. 生成程序员，内联该指导

**断言：**
- [ ] 不匹配提示在任何 agent 生成之前显示
- [ ] 提供全部三个选项（[A] 重读 / [B] 接受漂移 / [C] 停止），且 [A] 点名它所做的故事编辑
- [ ] 选 [A] 时检查大小，小于 50KB 的 ADR 用单次 `Read` 读取
- [ ] 在生成程序员之前，故事的 `ADR Version` 变为 ADR-0004 的 `Last Verified` 日期——而非今天的日期
- [ ] 程序员简报内联携带新读取的 Decision 指导，而非 [A] 之前故事所持有的摘要

**Fixture（Date 回退）：**
- 同上，但 ADR-0004 无 `## Last Verified` 节；其 `## Date` 等于故事的 `ADR Version`（`/create-stories` 从中标记的值）

**预期行为（Date 回退）：**
1. 同一次 Grep 未找到 `Last Verified`，因此 ADR 的当前版本为其 `## Date`
2. 这与故事的 `ADR Version` 匹配：信任故事，不显示提示，且不读取 ADR

**断言（Date 回退）：**
- [ ] 不显示不匹配提示——从其 `## Date` 标记的 ADR 不因缺少 `## Last Verified` 而被视为过时
- [ ] ADR-0004 不超出那一次 Grep 被读取

---

### 用例 4：边界情况——无参数；从会话状态读取

**Fixture：**
- 未提供参数
- `production/session-state/active.md` 引用一个活跃故事文件
- 该故事文件存在，`Status: In Progress`

**输入：** `/dev-story`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 技能读取 `production/session-state/active.md`
3. 技能找到活跃故事引用
4. 技能与用户确认："Continuing work on [story title] — is that correct?"
5. 确认后，技能继续处理该故事

**断言：**
- [ ] 未提供参数时技能读取会话状态
- [ ] 技能在继续之前与用户确认活跃故事
- [ ] 技能不会未经确认就静默假定活跃故事

**Fixture（无活跃故事）：**
- 未提供参数
- `production/session-state/active.md` 未点名任何活跃故事
- `production/epics/` 含两个 `Status: Ready` 的故事和一个 `In Progress` 的故事

**预期行为（无活跃故事）：**
1. 技能询问"Which story are we implementing?"
2. 它 glob `production/epics/**/*.md` 并列出两个 `Status: Ready` 的故事

**断言（无活跃故事）：**
- [ ] 询问问题而非猜测故事
- [ ] 仅列出 `Status: Ready` 的故事

---

### 用例 5：Minimal 层级，Logic 故事——测试豁免

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`modes.workflow: minimal`、`qa.level: minimal`
- 故事 Type Logic，`Requirement: Brief MVP feature 1`，ADR 字段 `N/A (minimal — no ADRs)`
- 无 TR 注册表、无 ADR、无控制清单
- `docs/engine-reference/godot/VERSION.md` 将固定版本评为 HIGH 风险

**输入：** `/dev-story production/epics/[slug]/story-001-[slug].md`

**预期行为：**
1. 阶段 2 不因缺少注册表、ADR 或清单而停止或设为 BLOCKED
2. 生成之前，打印一行 `Briefing omits: …`，点名 TR 注册表、ADR 指导和清单
3. 生成 `gameplay-programmer` 加引擎专家（VERSION.md 风险 HIGH）
4. 简报省略测试要求并告知 agent："Do not write a test file for this story — test evidence is waived at `qa.level: minimal`."
5. 阶段 5 被跳过；摘要打印"Test evidence: waived at `qa.level: minimal` — no test was required or written for this story."
6. 运行并观察仍执行并打印一行 `Run result:`
7. 下一步直接为 `/story-done [story-path]`——无 `/code-review`

**断言：**
- [ ] 在 `minimal` 下不因缺少 TR 注册表、ADR 或清单而 STOP 或 BLOCKED
- [ ] `Briefing omits:` 行点名全部三个被丢弃的输入，包括 ADR
- [ ] 因 VERSION.md 评为 HIGH 风险而生成引擎专家
- [ ] 测试豁免告知 agent 并在摘要中打印；不出现测试行或"run your test suite"段落
- [ ] 打印一行 `Run result:`——运行并观察未被豁免
- [ ] 交接为 `/story-done`，无 `/code-review` 步骤

---

### 用例 5b：Minimal 层级，UI 故事——截图仍必需

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`modes.workflow: minimal`、`qa.level: minimal`
- 故事 Type UI，`Requirement: Brief MVP feature 2`，ADR 字段 `N/A (minimal — no ADRs)`
- 无 TR 注册表、无 ADR、无控制清单

**输入：** `/dev-story production/epics/[slug]/story-002-[slug].md`

**预期行为：**
1. 生成 `ui-programmer`；简报无测试项，也无"Do not write a test file"句子——测试要求只覆盖 Logic 和 Integration
2. 摘要打印"Retained screenshot required under `production/qa/evidence/[story-slug]/` before this story can be closed…"
3. 运行并观察在 `production/qa/evidence/[story-slug]/` 下保留截图，并打印 `Run result: OBSERVED — …` 及路径
4. 下一步为 `/story-done [story-path]`

**断言：**
- [ ] UI 故事未被告知其证据被豁免——既不在