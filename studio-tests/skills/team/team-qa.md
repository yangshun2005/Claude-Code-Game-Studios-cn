# 技能测试规范：/team-qa

## 技能摘要

通过 6 阶段结构化测试周期编排 QA 团队。协调 qa-lead（策略、冒烟检查评估、签核报告）和 qa-tester（测试用例编写、缺陷报告编写）。阶段 1 加载范围（sprint 文件、`feature:` 形式，或从会话状态推断）和项目阶段；阶段 2 对故事分类，并从最新的 `production/qa/smoke-*.md` 报告获取冒烟检查结论（无报告时为 UNKNOWN；报告标注时为 NOT ASSESSED——两者均带警告继续并排除 APPROVED；FAIL 则停止周期）；在 `qa.level: minimal`（测试豁免）下，无测试的 Logic、Integration 或 Config/Data 故事为 WAIVED，其证据为手动 QA 中的验收标准走查；阶段 3 询问后编写 QA 计划；阶段 4 为每个手动 QA 故事生成一个 qa-tester，并指定输出路径——`team.size: studio` 时每个故事并行生成一个，低于该值时整个组生成一个；阶段 5 走查手动 QA、提交缺陷、让测试人员保留截图，并在询问后编写证据产物；阶段 6 生成签核报告，范围内每个故事一行，结论为 NOT ASSESSED / APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED。阶段 0 解析 `team.size`，技能在阶段 1 前宣布活动集。编排器级结论为 COMPLETE 或 BLOCKED。

---

## 静态断言（结构性）

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题（阶段 1 至阶段 6）
- [ ] 包含结论关键词：COMPLETE、BLOCKED
- [ ] 包含签核结论关键词：NOT ASSESSED、APPROVED、APPROVED WITH CONDITIONS、NOT APPROVED
- [ ] 对每次编排器写入均包含"May I write"措辞：QA 计划（阶段 3）、证据产物（阶段 5）和签核报告（阶段 6）
- [ ] 具有错误恢复协议章节
- [ ] 在阶段转换时使用 `AskUserQuestion` 获取用户批准后再继续
- [ ] 阶段 2 从最新的 `production/qa/smoke-*.md` 报告获取冒烟结论，FAIL 在阶段 3 前停止周期
- [ ] 阶段 4 为每个 qa-tester 指定输出路径：`production/qa/test-cases/[story-slug]-cases.md`
- [ ] 缺陷报告写入 `production/qa/bugs/`，命名为 `BUG-[NNNN].md`——沿用 `/bug-report` 的编号，以便 `/bug-report verify` 能找到它们
- [ ] 通过的 Visual/Feel 和 UI 故事，其涉及的每个屏幕或效果在 `production/qa/evidence/[story-slug]/` 下保留截图，并在磁盘上确认，同时在 `production/qa/evidence/[story-slug]-evidence.md` 生成引用该截图的证据产物——截图关闭 UI 故事；Visual/Feel 故事还需负责人签核
- [ ] 后续步骤指导因结论而异（NOT ASSESSED / APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED）
- [ ] 阶段 0 解析 `team.size`，技能要求在阶段 1 前进行一行活动集宣布
- [ ] 阶段 0 解析 `qa.level`；在 `minimal` 下，无测试的 Logic、Integration 或 Config/Data 故事在阶段 4–5 走查其验收标准——测试 WAIVED，绝不计为缺失证据
- [ ] 阶段 2 冒烟结论列表包含 NOT ASSESSED，其像 UNKNOWN 一样带警告继续，并使签核为 NOT ASSESSED
- [ ] 阶段 1 从 `production/qa/bugs/` 读取范围内的开放缺陷（除非 `Closed` 或 `Verified Fixed`，否则为开放），并传入阶段 2 和阶段 6 的简报
- [ ] 阶段 6 简报携带证据和结论规则，不指定结论；编排器在写入前按规则检查 qa-lead 的结论
- [ ] 阶段 4 在 `team.size: studio` 时每个故事并行生成一个 qa-tester；在 `small` 和 `individual` 时一个 qa-tester 生成覆盖整个组
- [ ] 阶段 6 签核为范围内每个故事提供测试覆盖率摘要行——包括 BLOCKED 和未执行的故事——其模板显示 BLOCKED 行

---

## 夹具默认值

除非自身夹具另有说明，每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有故事通过手动 QA，结论 APPROVED

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `production/sprints/sprint-03.md` 列出 4 个故事，其故事文件存在
- 故事类型混合：1 个 Logic（其单元测试存在且通过）、1 个 Integration（无自动化测试）、2 个 Visual/Feel
- 所有故事均已填充验收标准
- 最新的 `production/qa/smoke-[date].md` 报告结论为 PASS
- `production/qa/bugs/` 不含缺陷

**输入：** `/team-qa sprint-03`

**预期行为：**
1. 在任何生成之前，一行宣布 `team.size: small` 的活动集
2. 阶段 1：Glob `production/sprints/sprint-*.md`，匹配 sprint 编号 3（`sprint-03.md`）并读取；从 `project.yaml` 读取 `project.stage`（回退 `production/stage.txt`）；读取范围内开放缺陷（文件中无）；报告"QA cycle starting for sprint-03. Found 4 stories. Open bugs in scope: none on file. Current stage: [stage]. Ready to begin QA strategy?"
3. 阶段 2：通过 Agent 生成 `qa-lead`；在策略表中分类全部 4 个故事；读取冒烟报告并记录 `Smoke Check: PASS — source: production/qa/smoke-[date].md`，不重新询问用户；呈现策略；`AskUserQuestion` "QA Strategy Review"：用户选择"Looks good — proceed to test plan"
4. 阶段 3：生成 QA 计划（范围、分类表、自动化/手动范围、进入和退出标准）；询问"May I write the QA plan to `production/qa/qa-plan-sprint-03-[date].md`?"；批准后写入
5. 阶段 4：通过 Agent 为 Integration 和两个 Visual/Feel 故事生成一个 `qa-tester`（逐故事并行生成是 `studio` 形态），其提示为每个故事指定 `production/qa/test-cases/[story-slug]-cases.md`；测试用例按故事分组呈现；每组 `AskUserQuestion`；用户批准
6. 阶段 5：每个故事 `AskUserQuestion`（PASS / PASS WITH NOTES / FAIL / BLOCKED）；用户全部标记为 PASS；对每个 Visual/Feel 故事，测试人员在 `production/qa/evidence/[story-slug]/` 下保存截图，编排器确认其在磁盘上；在"May I write the evidence for [story titles] to `production/qa/evidence/`?"之后，写入 `production/qa/evidence/[story-slug]-evidence.md`，引用截图，签核行除非本次会话中已签核否则留空；结果摘要："Stories PASS: 4, PASS WITH NOTES: 0, FAIL: 0, BLOCKED: 0"
7. 阶段 6：通过 Agent 生成 `qa-lead` 生成签核报告；每个故事均有已执行证据；无缺陷提交；结论：APPROVED；询问"May I write this QA sign-off report to `production/qa/qa-signoff-sprint-03-[date].md`?"；批准后写入
8. 结论：COMPLETE——QA 周期完成

**断言：**
- [ ] 在生成第一个 agent 之前出现命名 `team.size: small` 的活动集行
- [ ] 阶段 1 正确计数并报告 4 个故事及当前阶段
- [ ] 阶段 2 策略表以正确类型分类全部 4 个故事
- [ ] 冒烟检查结论取自现有 `production/qa/smoke-*.md` 报告并显示其来源
- [ ] QA 计划仅在"May I write?"批准后写入
- [ ] 阶段 4 在 `team.size: small` 时生成单个 qa-tester，其提示为每个故事指定 `production/qa/test-cases/` 路径
- [ ] 每个通过的 Visual/Feel 故事在 `production/qa/evidence/[story-slug]/` 下磁盘上有保留截图，其证据产物引用它且仅在"May I write"询问后写入
- [ ] 签核报告包含测试覆盖率摘要表和结论：APPROVED
- [ ] 签核报告仅在"May I write?"批准后写入
- [ ] 最终输出中出现结论：COMPLETE
- [ ] 后续步骤："Run `/gate-check` to validate advancement."

---

### 用例 2：冒烟检查失败——QA 周期在阶段 2 停止

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `production/sprints/sprint-04.md` 列出 3 个故事
- 最新的 `production/qa/smoke-[date].md` 报告结论为 FAIL，列出 2 项失败（启动时构建不稳定、核心导航损坏）

**输入：** `/team-qa sprint-04`

**预期行为：**
1. 阶段 1 正常完成
2. 阶段 2：通过 Agent 生成 `qa-lead`；qa-lead 读取冒烟报告并记录 `Smoke Check: FAIL`，突出列出两项失败
3. 呈现 `AskUserQuestion` "QA Strategy Review"，包含选项"Smoke check failed — fix issues and re-run /team-qa"
4. 技能不进入阶段 3：呈现冒烟报告中的失败并停止——用户必须修复、重新运行 `/smoke-check sprint`，然后重新运行 `/team-qa`
5. 不生成 QA 计划、测试用例、手动 QA 或签核报告；给出部分报告
6. 结论：BLOCKED——冒烟检查失败

**断言：**
- [ ] 冒烟检查结论来自 `production/qa/smoke-*.md` 报告——qa-lead 不重新推导或重新询问用户
- [ ] 冒烟检查 FAIL 在阶段 2 停止流水线——阶段 3、4、5、6 不执行
- [ ] 失败列表明确展示给用户（非模糊概括）
- [ ] 技能建议 `/smoke-check sprint` 和重新运行 `/team-qa` 作为补救步骤
- [ ] 不写入或提供 QA 计划和 QA 签核报告
- [ ] 编排器结论为 BLOCKED，非 COMPLETE

---

### 用例 3：发现缺陷——Visual/Feel 故事手动 QA 失败，提交缺陷报告

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `production/sprints/sprint-05.md` 列出 2 个故事：1 个 Logic（其自动化测试通过）、1 个 Visual/Feel
- 最新的 `production/qa/smoke-[date].md` 报告结论为 PASS
- Visual/Feel 故事的动画时序明显错误（验收标准未满足）
- `production/qa/bugs/` 已包含 `BUG-0001.md` 和 `BUG-0002.md`

**输入：** `/team-qa sprint-05`

**预期行为：**
1. 阶段 1–4 正常完成；为 Visual/Feel 故事编写测试用例
2. 阶段 5：用户将 Visual/Feel 故事标记为 FAIL；`AskUserQuestion` 收集失败描述："Animation plays at 2x speed — jitter visible on every loop"
3. 阶段 5：通过 Agent 生成 `qa-tester` 在 `production/qa/bugs/BUG-0003.md` 编写正式缺陷报告；报告包含严重性字段
4. 结果摘要："Stories PASS: 1, PASS WITH NOTES: 0, FAIL: 1 — bugs filed: BUG-0003, BLOCKED: 0"
5. 阶段 6：生成 `qa-lead` 生成签核报告；每个故事均有已执行证据；缺陷发现表列出 BUG-0003 及故事、严重性和状态 Open；结论：NOT APPROVED（故事 FAIL 且无记录变通方案）
6. 提供签核报告写入；批准后写入
7. 后续步骤："Resolve S1/S2 bugs and re-run `/team-qa` or targeted manual QA before advancing."

**断言：**
- [ ] 阶段 5 的 FAIL 结果触发 `AskUserQuestion` 在编写缺陷报告前收集失败描述
- [ ] 通过 Agent 生成 `qa-tester` 编写缺陷报告——编排器不直接编写
- [ ] 缺陷报告遵循 `/bug-report` 的命名：`production/qa/bugs/` 中的 `BUG-[NNNN].md`
- [ ] ID 为现有最高缺陷号加一，零填充至四位（BUG-0003）
- [ ] 阶段 6 签核报告的缺陷发现表包含缺陷 ID、故事名称、严重性和状态
- [ ] 签核报告中的结论为 NOT APPROVED
- [ ] 后续步骤明确提及重新运行 `/team-qa`
- [ ] 编排器仍发出结论：COMPLETE（QA 周期完成——签核结论为 NOT APPROVED，但技能完成了其流水线）

---

### 用例 4：无参数——技能推断活动 sprint 或询问用户

**夹具（变体 A——状态文件存在）：**
- `production/session-state/active.md` 存在并包含对 `sprint-06` 的引用
- `production/sprint-status.yaml` 存在并标识 `sprint-06` 为活动
- `production/sprints/sprint-06.md` 存在

**夹具（变体 B——无 sprint，如 `rigor: minimal` 时）：**
- `production/session-state/active.md` 不存在
- `production/sprint-status.yaml` 不存在
- 无 sprint 文件；`production/epics/` 下存在史诗

**输入：** `/team-qa`（无参数）

**预期行为（变体 A）：**
1. 阶段 1：未提供参数；读取 `production/session-state/active.md` 和 `production/sprint-status.yaml`
2. 检测 `sprint-06` 为活动 sprint
3. 如同输入为 `/team-qa sprint-06` 一样继续；阶段 1 报告行命名它："QA cycle starting for sprint-06. Found [N] stories. Open bugs in scope: [N or none on file]. Current stage: [stage]. Ready to begin QA strategy?"

**预期行为（变体 B）：**
1. 阶段 1：未提供参数；尝试读取两个状态文件——均缺失；无 sprint 存在
2. 询问用户覆盖哪个史诗，然后将运行范围设为 `feature: [epic-slug]`——不虚构 sprint

**断言：**
- [ ] 未提供参数时技能不默认使用硬编码 sprint 名称
- [ ] 技能在询问用户前读取 `production/session-state/active.md` 和 `production/sprint-status.yaml`（变体 A）
- [ ] 无法推断 sprint 时，技能询问覆盖哪个史诗并使用 `feature: [epic-slug]` 形式而非猜测（变体 B）
- [ ] 推断的 sprint 在阶段 2 开始前于阶段 1 报告行中命名（变体 A）
- [ ] 状态文件缺失时技能不报错——回退到询问（变体 B）

---

### 用例 5：混合结果——部分 PASS，一个 FAIL 带 S1 缺陷，一个 BLOCKED

**夹具：**
- 解析后的配置块：`team.size: small`、`automation: collaborative`
- `production/sprints/sprint-07.md` 列出 4 个故事
- 最新的 `production/qa/smoke-[date].md` 报告结论为 PASS
- 故事 A（Logic）：自动化测试通过——PASS
- 故事 B（UI）：手动 QA——PASS WITH NOTES（轻微文本溢出）
- 故事 C（Visual/Feel）：手动 QA——FAIL；测试人员识别技能激活时 S1 崩溃
- 故事 D（Integration）：无法测试——BLOCKED（依赖系统尚未实现）
- `production/qa/bugs/` 为空

**输入：** `/team-qa sprint-07`

**预期行为：**
1. 阶段 1–4 进行；阶段 4 测试用例覆盖故事 B、C、D
2. 阶段 5：故事 A 从其自动化测试为 PASS；用户标记故事 B：PASS WITH NOTES；故事 C：FAIL；故事 D：BLOCKED
3. 故事 B（UI）其涉及的每个屏幕获得保留截图，由测试人员保存在 `production/qa/evidence/[story-slug]/` 下并在磁盘上确认；其证据产物 `production/qa/evidence/[story-slug]-evidence.md` 引用它们且仅在"May I write"询问后写入
4. 故事 C FAIL 后：收集失败描述；生成 qa-tester 编写缺陷报告 `BUG-0001.md`，严重性 S1
5. 呈现结果摘要："Stories PASS: 1, PASS WITH NOTES: 1, FAIL: 1 — bugs filed: BUG-0001, BLOCKED: 1"
6. 阶段 6：qa-lead 生成覆盖全部 4 个故事的签核