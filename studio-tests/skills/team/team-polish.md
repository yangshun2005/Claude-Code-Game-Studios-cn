# 技能测试规范：/team-polish

## 技能摘要

通过六阶段流水线编排打磨团队：性能评估（performance-analyst，其报告携带优化清单，每项标注负责人）→ 优化（清单由各负责人在活跃集中实施——engine-programmer 负责引擎级根因，tools-programmer 负责内容工具；performance-analyst 不编写游戏代码）→ 视觉打磨（technical-artist，与阶段 2 并行）→ 音频打磨（sound-designer，与阶段 2 并行）→ 加固（qa-tester）→ 签核（编排器收集所有结果并给出 READY FOR RELEASE、NOT ASSESSED 或 NEEDS MORE WORK——当预算未提交或某阶段未运行时为 NOT ASSESSED）。阶段 0 解析 `review_mode`、`automation`、`team.size` 和 `workflow`，技能在阶段 1 前公布活跃集——默认 `individual` 仅生成 performance-analyst + technical-artist，因此没有程序员活跃，报告将优化清单标注为交给 `/dev-story`；`small` 增加 sound-designer、qa-tester 和 engine-programmer（完整流水线）；`studio` 保留该集合并增加对抗性审查。每个 agent 在受限写入例外下（无逐次写入提示）将带日期的报告写入 `production/polish/` 下的指定路径，编排器声明目前没有任何东西读取 `production/polish/`。阶段 2–4 的代码、着色器、音频和资源变更不在该例外内：一次询问覆盖整个集合。在每个阶段转换处使用 `AskUserQuestion`。engine-programmer 仅在阶段 1 识别出引擎级根因时条件性生成。

---

## 静态断言（结构性）

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含裁决关键词：READY FOR RELEASE、NOT ASSESSED、NEEDS MORE WORK——并声明 NOT ASSESSED 的优先级高于 READY FOR RELEASE、低于 NEEDS MORE WORK
- [ ] 包含"File Write Protocol"章节
- [ ] 文件写入委托给子 agent——编排器不直接写文件
- [ ] 为每个 agent 在 `production/polish/` 下指定目标路径：`[area]-report-[date].md`（performance-analyst）、`[area]-render-notes-[date].md`（technical-artist）、`[area]-engine-fixes-[date].md`（engine-programmer）、`[area]-tool-fixes-[date].md`（tools-programmer）、`[area]-audio-notes-[date].md`（sound-designer）、`[area]-verification-[date].md`（qa-tester）
- [ ] 子 agent 写入这些指定的 `production/` 路径时无需逐次写入提示（受限例外），任何例外之外的写入必须询问；实现文件（代码、着色器、音频、资源）在例外之外——每个 agent 列出其将创建或更改的文件，技能在写入任何文件前对整个集合询问一次——阶段 2–4 整个集合一次询问，绝不每阶段一次
- [ ] `small` 和 `studio` 集合包含 `engine-programmer`；在 `individual` 下没有程序员活跃，报告将优化清单标注为交给 `/dev-story`
- [ ] 将阶段门定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门声称生成导演
- [ ] 末尾有下一步交接（引用 `/release-checklist`、`/sprint-plan update`、`/gate-check`）
- [ ] 存在错误恢复协议章节：验证指定产物在磁盘上，立即暴露 BLOCKED，始终生成部分报告，完整流程见 `.claude/docs/error-recovery-protocol.md`
- [ ] 在继续前于阶段转换处使用 `AskUserQuestion`（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，技能要求在阶段 1 前进行一行活跃集公告
- [ ] 阶段 3（视觉打磨）和阶段 4（音频打磨）明确与阶段 2 并行运行
- [ ] 阶段 1 的报告携带优化清单，每项标注负责人；performance-analyst 在任何阶段都不编写游戏代码
- [ ] engine-programmer 仅在阶段 1 识别出引擎级根因时于阶段 2 条件性生成
- [ ] 阶段 6 签核在给出裁决前将指标与预算比较
- [ ] 阶段 6 列出每个剩余问题及其严重程度、有测量差距时的差距（例如"超出预算 3 ms"）和建议

---

## 夹具默认值

每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`，除非其自身夹具另有说明。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——完整流水线完成，READY FOR RELEASE 裁决

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- 功能存在且功能完整（例如 `combat` 系统）
- 性能预算在 `project.yaml` 中定义（`performance.target_framerate: 60`、`performance.frame_budget_ms: 16.6`、`performance.draw_call_limit: 2000`、`performance.memory_ceiling_mb: 2048`）
- 打磨开始前不存在帧预算违规
- 无缺失音频事件；VFX 资源完整
- 打磨变更未引入回归

**输入：** `/team-polish combat`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: small` 的活跃集（完整流水线）
2. 阶段 1：生成 performance-analyst；分析 combat 系统、测量帧预算、检查内存使用；写入 `production/polish/combat-report-[date].md`，显示所有指标在预算内
3. `AskUserQuestion` 呈现性能报告；用户在阶段 2、3、4 开始前批准
4. 阶段 2：报告的优化清单仅包含次要渲染项（例如绘制调用批处理），由 technical-artist 负责，因此它们加入其阶段 3 简报；未识别出引擎级根因，因此不生成 engine-programmer；performance-analyst 不更改任何代码
5. 阶段 3 和 4 与阶段 2 并行启动：
   - 阶段 3：technical-artist 审查 VFX 质量、优化粒子系统、添加屏幕震动和视觉冲击；写入 `production/polish/combat-render-notes-[date].md`
   - 阶段 4：sound-designer 审查音频事件完整性、检查混音电平、添加环境音频层；写入 `production/polish/combat-audio-notes-[date].md`
   - 更改着色器、VFX 或资源的 agent 先列出文件；一次 `AskUserQuestion` 覆盖整个阶段 2–4 集合，在同意前不编辑任何内容
6. 三个并行阶段全部完成；`AskUserQuestion` 呈现结果；用户在阶段 5 开始前批准
7. 阶段 5：qa-tester 运行边界用例测试、浸泡测试、压力测试和回归测试；全部通过；写入 `production/polish/combat-verification-[date].md`
8. `AskUserQuestion` 呈现测试结果；用户在阶段 6 前批准
9. 阶段 6：编排器收集所有结果；将前后性能指标与预算比较；所有指标通过
10. 报告说明 `production/polish/` 文件是记录，仓库中尚无任何东西读取它们
11. 裁决：READY FOR RELEASE

**断言：**
- [ ] 在生成第一个 agent 前出现命名 `team.size: small` 的活跃集行
- [ ] performance-analyst 在阶段 1 中先于任何其他 agent 生成
- [ ] `AskUserQuestion` 出现在阶段 1 输出之后、阶段 2/3/4 启动之前
- [ ] 阶段 3 和 4 的 Agent 调用与阶段 2 同时发出（而非阶段 2 完成后）
- [ ] 当阶段 1 未发现引擎级根因时不生成 engine-programmer
- [ ] performance-analyst 在任何阶段都不编辑代码、着色器或资源
- [ ] 阶段 2–4 的实现文件由每个 agent 列出，并在编辑任何文件前通过一次 `AskUserQuestion` 对整个集合询问——而非每阶段一次
- [ ] qa-tester（阶段 5）在并行阶段完成且用户批准前不启动
- [ ] 每个 agent 写入其指定的 `production/polish/[area]-…-[date].md` 路径时无需逐次写入提示，编排器在将阶段视为完成前确认文件存在
- [ ] 阶段 6 裁决基于指标与已定义预算的比较
- [ ] 摘要报告包含：前后性能指标、视觉打磨变更、音频打磨变更、测试结果
- [ ] 摘要声明目前没有任何东西读取 `production/polish/`
- [ ] 编排器不直接写入任何文件
- [ ] 裁决为 READY FOR RELEASE

---

### 用例 2：性能阻塞——帧预算违规无法完全解决

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`、`workflow: standard`（固定：未设置解析为 minimal，其下一步将问题添加到简报的构建顺序而非 `/sprint-plan update`；用例 8 覆盖该情况）
- 正在打磨的功能：`particle-storm` VFX 系统
- 阶段 1 识别出帧预算违规：particle-storm 在目标硬件上耗时 12ms（该系统预算为 6ms）
- 阶段 1 的优化清单将粒子工作分配给 technical-artist，将引擎级粒子更新热路径分配给 engine-programmer；两者应用后成本为 9ms——仍超出 6ms 预算
- 阶段 2 无法在不进行根本性设计变更的情况下完全解决违规

**输入：** `/team-polish particle-storm`

**预期行为：**
1. 阶段 1：performance-analyst 识别出 12ms 帧成本 vs. 6ms 预算；报告"FRAME BUDGET VIOLATION: particle-storm costs 12ms, budget is 6ms"，优化清单标注每项负责人
2. `AskUserQuestion` 呈现违规；用户选择继续尝试优化
3. 阶段 2：engine-programmer 实施引擎级项，technical-artist 实施粒子项（在对集合的一次询问之后）；重新分析读数为 9ms——已降低但仍超预算："Optimization reduced cost to 9ms (was 12ms) — 3ms over budget. No further gains achievable without design changes."
4. 阶段 3 和 4 与阶段 2 并行运行（视觉和音频打磨）
5. 阶段 5：qa-tester 运行回归和边界用例测试；全部通过
6. 阶段 6：编排器收集结果；帧预算违规（9ms vs 6ms 预算）仍未解决
7. 裁决：NEEDS MORE WORK
8. 报告列出具体未解决问题及严重程度："particle-storm frame cost (9ms) exceeds budget (6ms) by 3ms — requires design scope reduction or budget renegotiation"
9. 下一步：在 `/sprint-plan update` 中安排剩余问题；修复后重新运行 `/team-polish`

**断言：**
- [ ] 帧预算违规在阶段 1 以具体数字标记（实际 vs. 预算）
- [ ] 阶段 2 明确报告优化后指标（达到 9ms，仍超出 3ms）
- [ ] 当预算违规仍存在时裁决为 NEEDS MORE WORK（而非 READY FOR RELEASE）
- [ ] 具体未解决问题按名称列出，剩余差距量化
- [ ] 下一步引用 `/sprint-plan update` 安排剩余修复
- [ ] 阶段 3 和 4 仍运行（打磨工作不因阶段 2 部分解决而放弃）
- [ ] 阶段 5 qa-tester 仍运行（回归测试独立于性能结果）

---

### 用例 3：无参数——显示用法指导

**夹具：**
- 任意项目状态

**输入：** `/team-polish`（无参数）

**预期行为：**
1. 技能检测到未提供参数
2. 输出用法指导：例如"Usage: `/team-polish [feature or area to polish] [--review full|lean|solo]` — specify the feature or area to polish (e.g., `combat`, `main menu`, `inventory system`, `level-1`)"
3. 技能退出，不生成任何 agent

**断言：**
- [ ] 未提供参数时技能不生成任何 agent
- [ ] 用法消息显示完整 argument-hint——`/team-polish [feature or area to polish] [--review full|lean|solo]`——和参数示例
- [ ] 技能不尝试从项目文件猜测功能
- [ ] 不使用 `AskUserQuestion`——输出为直接指导

---

### 用例 4：引擎级瓶颈——engine-programmer 在阶段 2 条件性生成

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- 正在打磨的功能：`open-world` 环境流式加载
- 阶段 1 识别出根因在渲染管线的性能瓶颈："draw call overhead is caused by the engine's scene tree traversal in the spatial indexer — this is an engine-level issue, not a game code issue"
- 性能预算已定义；渲染开销超出目标帧预算

**输入：** `/team-polish open-world`

**预期行为：**
1. 阶段 1：performance-analyst 分析环境；识别帧预算违规；根因分析指向引擎级渲染管线（空间索引器遍历开销）
2. 阶段 1 输出明确将根因归类为引擎级
3. `AskUserQuestion` 呈现包含引擎级根因的性能报告；用户在阶段 2 前批准
4. 阶段 2：engine-programmer（在 `small` 集合中）为优化清单上的引擎级渲染项生成，写入 `production/polish/open-world-engine-fixes-[date].md`；performance-analyst 不编写代码
5. 阶段 3 和 4 也与阶段 2 并行运行（视觉和音频打磨）；engine-programmer 列出其将更改的引擎文件及其他 agent 的文件，一次询问覆盖该集合
6. engine-programmer 在同意后处理空间索引器遍历；提供分析器验证显示修复降低开销
7. 阶段 5：qa-tester 运行回归测试，包括引擎级变更的测试
8. 阶段 6：编排器收集所有结果；若指标现在在预算内，裁决为 READY FOR RELEASE；否则为 NEEDS MORE WORK

**断言：**
- [ ] 除非阶段 1 明确识别出引擎级根因，否则 engine-programmer 不在阶段 2 生成
- [ ] 当阶段 1 识别出引擎级根因时，engine-programmer 在阶段 2 生成
- [ ] engine-programmer 的阶段 2 Agent 调用与阶段 3 和 4 调用同时发出（非顺序），且不重新生成 performance-analyst 来更改代码
- [ ] 在对阶段 2–4 实现集合的一次询问得到同意前，engine-programmer 不编辑任何内容
- [ ] 阶段 3 和 4 也与阶段 2 并行运行（不推迟到阶段 2 完成）
- [ ] engine-programmer 的输出包含修复的分析器验证
- [ ] 阶段 5 的 qa-tester 运行覆盖引擎级变更的回归测试
- [ ] 裁决正确反映包括引擎修复在内的所有指标现在是否满足预算

---

### 用例 5：发现回归——打磨变更破坏了现有功能

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`、`workflow: standard`（固定原因同用例 2）
- 正在打磨的功能：`inventory-ui`
- 阶段 1–4 成功完成；性能和打磨变更已应用
- 阶段 5：qa-tester 运行回归测试，发现阶段 3 应用的着色器优化破坏了悬停时的物品高亮发光效果——这是打磨前正常工作的现有功能

**输入：** `/team-polish inventory-ui`（阶段 5 场景）

**预期行为：**
1. 阶段 1–4 完成；打磨变更包括 technical-artist 的着色器优化
2. 阶段 5：qa-tester 运行回归测试并检测到"Item highlight glow on hover no longer renders — regression introduced by shader optimization in Phase 3"
3. qa-tester 将其发现写入 `production/polish/inventory-ui-verification-[date].md`（无逐次写入提示），并在其返回契约中将回归作为 CONCERNS/BLOCKED 行返回
4. 编排器确认验证文件存在，然后在阶段 5 → 阶段 6 转换处于对话中写入 qa-tester 的完整分析——命名损坏的行为和导致它的阶段 3 着色器变更——并通过 `AskUserQuestion` 以 qa-tester 的建议为选项捕获决策
5. 阶段 6：回归被列入剩余问题，附严重程度和建议
6. 裁决：NEEDS MORE WORK
7. 下一步：在 `/sprint-plan update` 中安排修复，修复后重新运行 `/team-polish`

**断言：**
- [ ] 回归在阶段 6 签核前暴露
- [ ] 具体损坏的行为和负责的变更都在报告中命名
- [ ] qa-tester 的发现写入 `production/polish/inventory-ui-verification-[date].md`，无需单独批准提示
- [ ] 阶段 5 → 6 转换处的 `AskUserQuestion` 将回归和 qa-tester 建议的解决方案作为可选项呈现
- [ ] 阶段 6 将回归