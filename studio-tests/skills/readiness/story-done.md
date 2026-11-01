# 技能测试规范：/story-done

## 技能摘要

`/story-done` 闭合设计与实现之间的循环。在实现完一个故事后运行，它读取故事文件，从 `docs/architecture/tr-registry.yaml` 重新读取当前需求文本（绝不使用故事中引用的副本）以及主导 ADR 的 `## Decision` / `## Consequences`，验证每条验收标准（自动验证、通过批量 `AskUserQuestion` 验证，或标记为 `DEFERRED — requires playtest session`），构建标准到测试的可追溯性表格，检查测试证据，并检查 GDD/ADR/manifest/范围偏差（分类为 BLOCKING / ADVISORY / OUT OF SCOPE）。随后审查模式决定 QA 覆盖率门禁（QL-TEST-COVERAGE）和代码审查步骤（`full` 模式下为 LP-CODE-REVIEW，`lean` 模式下为一个 `/code-review` 问题，`solo` 模式下记录为跳过）。它产出 COMPLETE / COMPLETE WITH NOTES / NOT ASSESSED / BLOCKED 判定，并且仅在 `AskUserQuestion` 选择之后，更新故事文件，可选地追加到 `docs/tech-debt-register.md`，并呈现下一个故事——在没有冲刺计划时，从 `bash .claude/scripts/story-status.sh` 获取，该列表由 `/help` 和 `/sprint-status` 读取，在故事文件写入后运行。它只在打开截图后才说明截图显示了什么，并且询问行为标准而非从代码中读取它们。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥5 个阶段标题（复杂的多阶段技能）
- [ ] 包含判定关键词：COMPLETE、COMPLETE WITH NOTES、NOT ASSESSED、BLOCKED
- [ ] `allowed-tools` 包含 Write/Edit，因此存在写入前询问的措辞（"Use `AskUserQuestion` before writing anything"——涵盖故事文件和技术债务登记表）
- [ ] 具有下一步交接（从冲刺中呈现下一个故事）

---

## Fixture 默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身的 Fixture 另有说明。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——所有验收标准满足，无偏差

**Fixture：**
- `project.yaml` 设置 `modes.rigor: standard`（→ `workflow: standard`、`qa.level: standard`、`review_mode: lean`）和 `engine.name: Godot`，因此 `tests/unit/` 是测试根目录
- 故事文件位于 `production/epics/core/story-light-pickup.md`，包含：
  - `Type: Logic` 和 3 条验收标准，均按描述实现：AC-1 "Picking up a light adds it to the inventory"、AC-2 "A pickup at the carry limit is refused"、AC-3 "The picked-up light leaves the world"
  - `TR-light-001`；故事引用了需求的旧措辞，而 `tr-registry.yaml` 持有当前文本，实现与之匹配
  - `ADR: ADR-0003`（Accepted），其指导已被遵循
  - 一个 `## Test Evidence` 部分，指明 `tests/unit/light/pickup_test.gd`，该文件存在且每条标准一个测试——`test_pickup_adds_to_inventory`、`test_pickup_refused_at_limit`、`test_pickup_removes_world_light`——因此每条标准都映射到一个测试
  - `Status: In Progress`
- `production/session-state/active.md` 中的 `/dev-story` 检查点将此故事命名为 **Current task**，并带有 `Run result: OBSERVED — the light leaves the floor and the inventory count rises — production/qa/evidence/story-light-pickup/01-pickup.png`；该图像存在
- 故事中列出的实现文件存在于代码根目录
- `production/sprints/` 中的冲刺计划有其他 READY / NOT STARTED 的 Must Have 故事

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. 技能读取故事并提取其字段
2. 技能在 `tr-registry.yaml` 中 grep `TR-light-001` 并使用该当前文本
3. 技能映射 ADR 的标题并只读取 `## Decision` 和 `## Consequences`
4. 技能验证每条标准并构建可追溯性表格：AC-1..AC-3 各自映射到其测试函数，全部 COVERED
5. 找到测试证据——报告为 "Test file present at `<path>`"（存在性，而非通过声明）；检查点的 `Run result: OBSERVED` 连同其保留路径被记录
6. 无偏差
7. 阶段 4b 注明 "QL-TEST-COVERAGE skipped — Lean mode."
8. 阶段 5（lean）询问 "Code review is skipped in lean mode. Did you run `/code-review` on the implemented files?"
9. 完成报告，判定为 COMPLETE
10. `AskUserQuestion` "Verification complete. How do you want to proceed?"；选择 "Close the story" 后，文件被更新
11. 技能从冲刺中呈现下一个就绪故事

**断言：**
- [ ] 需求文本来自 `tr-registry.yaml`，而非故事的过时引用——措辞差异不产生偏差
- [ ] 技能只读取 ADR 的 `## Decision` 和 `## Consequences` 片段，而非整个文件
- [ ] 每条标准都列出其状态（`auto-verified` / `confirmed` / `FAILS` / `DEFERRED`）并出现在 Criterion | Test | Status 可追溯性表格中
- [ ] 测试证据报告为存在，而不声称测试通过
- [ ] 在 `lean` 模式下，技能询问是否运行了 `/code-review`，并将答案记录在 Completion Notes 的 `Code Review:` 行
- [ ] 可追溯性表格将 AC-1..AC-3 各自映射到其测试函数，没有 UNTESTED 行
- [ ] `Run result:` 行从检查点读取（其 **Current task** 命名此故事）并连同其保留路径记录
- [ ] 判定为 COMPLETE——每条标准映射到测试，运行被观察并保留，且无偏差
- [ ] 在阶段 7 的 `AskUserQuestion` 之前不编辑任何文件；选择 "Close the story" 后，故事获得 `Status: Complete`、`Last Updated:` 日期和 `## Completion Notes` 部分
- [ ] 完成后，技能从 `production/sprints/` 呈现下一个 READY / NOT STARTED 的 Must Have 或 Should Have 故事，并建议 `/story-readiness [path]`

---

### 用例 2：需要手动或试玩验证的标准

**Fixture：**
- 同用例 1，外加两条标准：
  - (a) "Player sees correct animation on pickup"——无自动化测试；动画的一帧保留在 `production/qa/evidence/story-light-pickup/02-pickup-anim.png`
  - (b) "Pickup state persists across a full level run"——需要完整游戏构建
- 5 条标准中有 1 条最终没有覆盖测试

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. 标准 (a) 是主观/玩法标准 → 技能通过批量 `AskUserQuestion` 询问（"Does [criterion]?"——`Yes — passes` / `No — fails` / `Not tested yet`）
2. 用户回答 `Yes — passes`。保留的帧是证据规则所要求的观察；答案覆盖了静态图无法展示的运动，并计为手动测试
3. 标准 (b) 标记为 `DEFERRED — requires playtest session` 且不阻塞
4. 可追溯性表格显示 (b) UNTESTED；5 条中 1 条（≤50%）→ ADVISORY
5. Completion Notes 列出延迟项和 "Untested criteria: [AC-N list]. Recommend adding tests in a follow-up story."

**断言：**
- [ ] 技能询问用户关于不可验证的标准，而非假设其通过，每次调用批量最多 4 个此类问题——行为标准绝不通过阅读代码标记为已验证
- [ ] 在报告说明 `02-pickup-anim.png` 显示什么之前，图像用 `Read` 打开；检查点的 `Run result:` 文本作为 `/dev-story` 的描述被引用，绝不被重述为技能自己的观察
- [ ] 仅试玩的标准标记为 `DEFERRED — requires playtest session`
- [ ] 判定为 COMPLETE WITH NOTES，报告列出 DEFERRED 标准——既非 BLOCKED 也非 NOT ASSESSED，因为 DEFERRED 标准不阻塞且不触发 NOT ASSESSED 触发器
- [ ] 延迟的标准在 Completion Notes 的 `Criteria:` 行中命名，并附有未测试标准的建议
- [ ] 变体——如果用户对 (a) 回答 `No — fails`，该标准为 `FAILS`，判定为 BLOCKED，技能不会自行进入阶段 7
- [ ] 变体——此故事在 `production/qa/evidence/` 下没有保留图像，且检查点没有 `Run result:` 行：用户对 (a) 的 `Yes — passes` 是断言而非证据，因此缺失的观察在 Logic 门禁级别被标记（默认 BLOCKING），判定为 BLOCKED——绝不因确认而给出 COMPLETE WITH NOTES
- [ ] 技能在更新故事文件前仍通过 `AskUserQuestion` 询问

---

### 用例 3：阻塞路径——检测到 GDD 偏差

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- 故事 TR-ID 的当前 `tr-registry.yaml` 文本："Player can carry max 3 light sources"
- 代码根目录中的实现在玩法代码中有 `MAX_CARRIED_LIGHTS = 5`

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. GDD 规则检查（对照当前注册表文本）发现实现与 "max 3" 矛盾
2. 硬编码值检查也标记了应属于数据文件的玩法字面量
3. 矛盾分类为 BLOCKING 并引用其 GDD/TR 参考
4. 判定 BLOCKED；技能列出必须修复的内容并提供帮助，且不会自行推进到阶段 7
5. 如果用户明确要求仍然关闭，阶段 7 菜单出现（始终询问，即使在自主模式下——`scope_changes`）

**断言：**
- [ ] 技能检测当前需求文本与实现值之间的不匹配
- [ ] 偏差中立地报告为 BLOCKING 并附 GDD/TR 参考——技能不编辑代码或 GDD 来调和
- [ ] 判定为 BLOCKED，除非用户明确要求仍然关闭，否则不进入阶段 7
- [ ] 关闭 BLOCKED 故事无论自动化模式如何都经过阶段 7 的 `AskUserQuestion`
- [ ] 如果通过 "Accept deviations as-is and close anyway" 关闭，偏差记录在 Completion Notes 的 `Deviations:` 行

---

### 用例 4：边缘情况——无参数，自动检测当前故事

**Fixture：**
- `production/session-state/active.md` 将 `production/epics/core/story-oxygen-drain.md` 命名为活动故事
- 该故事文件存在且 `Status: In Progress`

**输入：** `/story-done`（无参数）

**预期行为：**
1. 技能读取 `production/session-state/active.md` 并取活动故事
2. 技能读取该故事文件并正常继续
3. 完成报告的 `**Story**:` 行命名自动检测的路径
4. 如果 `active.md` 未命名故事，技能读取最新冲刺计划中的 IN PROGRESS 故事；若有多个，询问 "Which story are we completing?"；若没有，询问路径

**断言：**
- [ ] 无参数时技能首先读取 `production/session-state/active.md`
- [ ] 报告的 `**Story**:` 行命名自动检测的故事文件
- [ ] 冲刺中有多个进行中故事时，技能通过 `AskUserQuestion` 询问哪一个，而非自行挑选
- [ ] 如果任何地方都找不到故事，技能要求用户提供路径

---

### 用例 5：主管门禁——跨审查模式的 LP-CODE-REVIEW

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full` 和 `qa.level: standard`（因此 `qa.level: minimal` 时 QA 覆盖率门禁不被跳过）
- 故事文件位于 `production/epics/core/story-light-pickup.md`，`Type: Logic`
- 所有验收标准已验证，无 GDD 偏差，实现文件存在，且有用例 1 的运行证据（`Run result: OBSERVED` 检查点及其保留图像）
- 审查模式来自 `modes.review_mode`（旧版镜像 `production/review-mode.txt`）或 `--review`

**用例 5a——full 模式：** `/story-done production/epics/core/story-light-pickup.md --review full`

**预期行为：**
1. 阶段 3–4 之后，技能以门禁 QL-TEST-COVERAGE 生成 `qa-lead`（除非 `qa.level: minimal` 或 Config/Data 故事）
2. 技能以门禁 LP-CODE-REVIEW 生成 `lead-programmer`，传递实现文件路径、故事路径、相关 GDD 部分和主导 ADR
3. 判定（APPROVE / CONCERNS / REJECT）呈现给用户
4. CONCERNS → `AskUserQuestion`：`Revise flagged issues` / `Accept and proceed` / `Discuss further`
5. REJECT → 在问题解决前无阶段 6 判定
6. NOT ASSESSED（缺少门禁输入）→ 技能命名缺失的输入，绝不将其读作 APPROVE：输入被提供且门禁重新运行，或阶段 6 判定至多为 NOT ASSESSED

**断言（5a）：**
- [ ] 技能在决定是否生成 LP-CODE-REVIEW 之前使用解析后的审查模式
- [ ] full 模式下在实现检查之后生成 LP-CODE-REVIEW，带有四个上下文项
- [ ] REJECT 判定阻止故事在解决前达到 COMPLETE 判定
- [ ] CONCERNS 判定产生三选项 `AskUserQuestion`
- [ ] 任一门的 NOT ASSESSED 答案绝不被视为 APPROVE / ADEQUATE：除非提供缺失输入并重新运行门禁，判定为 NOT ASSESSED，而非 COMPLETE
- [ ] 即使在 APPROVE 之后，技能在更新故事状态前仍通过阶段 7 的 `AskUserQuestion` 询问

**用例 5b——lean 模式：** `--review lean`

**断言（5b）：**
- [ ] LP-CODE-REVIEW 不生成
- [ ] 技能询问 "Did you run `/code-review` on the implemented files?" 并给出三个列出的选项；三者都继续
- [ ] 注明 "QL-TEST-COVERAGE skipped — Lean mode."

**用例 5c——solo 模式：** `--review solo`

**断言（5c）：**
- [ ] LP-CODE-REVIEW 和 QL-TEST-COVERAGE 均不生成，且不询问代码审查问题
- [ ] 输出注明 "LP-CODE-REVIEW skipped — Solo mode." 和 "QL-TEST-COVERAGE skipped — Solo mode."
- [ ] 技能在标记故事 Complete 前仍要求阶段 7 的 `AskUserQuestion`

---

### 用例 6：`minimal` 下的 UI 故事，无冲刺计划——截图即关闭

**Fixture：**
- `project.yaml` 设置 `modes.rigor: minimal`（→ `workflow: minimal`、`qa.level: minimal`、`review_mode: solo`）；`production/sprints/` 中没有冲刺计划——简报的构建顺序即计划
- `production/epics/mvp/` 持有 `story-001-*.md`（`Status: Complete`）、此故事 `story-002-shop-panel.md`（`Status: In Progress`）和 `story-003-*.md`（`Status: Ready`）
- `story-002-shop-panel.md` 有 `Type: UI` 和两条标准："The shop panel shows all 6 item slots inside the frame at 1280x720" 和 "Each slot shows its price"
- `production/qa/evidence/story-002-shop-panel/01-shop-open.png` 存在并显示两者；检查点带有 `Run result: OBSERVED` 及该路径；没有证据文档和签核

**输入：** `/story-done production/epics/mvp/story-002-shop-panel.md`

**预期行为：**
1. 在 `qa.level: minimal` 下，Logic、Integration 和 Config/Data 证据检查、>50% 未测试升级和阶段 4b 被跳过（"QL-TEST-COVERAGE skipped — qa.level minimal."）——但 UI 截图检查运行
2. 保留的截图满足 UI 门禁；不要求证据文档或签核
3. 阶段 4 仅运行 manifest、硬编码值和范围检查（`minimal` 下无 GDD/ADR 可追溯性）；阶段 5 注明 "LP-CODE-REVIEW skipped — Solo mode."
4. 判定 COMPLETE；阶段 7 批准后故事标记为 Complete
5. 阶段