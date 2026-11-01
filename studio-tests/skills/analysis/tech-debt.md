# 技能测试规范：/tech-debt

## 技能摘要

`/tech-debt` 通过四个子命令跟踪技术债：`scan`、`add`、`prioritize` 和 `report`。不带子命令时打印用法并以 Verdict FAIL 停止。`scan` 搜索代码根目录（按引擎解析：`src/`、`Assets/` 或 `Source/`），查找 `TODO`、`FIXME`、`HACK` 和 `@deprecated` 标记、重复代码块、超过 500 行的文件和超过 50 行的函数，将发现按六类债务分类，说明覆盖了多少文件，并在追加（绝不覆盖）条目之前询问"May I write these findings to `docs/tech-debt-register.md`?"——更新匹配的 `Open` 行，并将模式已消失的 `Open` 行提议为 `Resolved [date]`。新行遵循技能的 Debt Register Format（下一个 `TD-NNN`、十列、`Added` 日期）。扫描零个源文件为 `NOT ASSESSED — no source files to scan`，代码根目录无法解析为 `NOT ASSESSED — code root unresolved`，两者都不写入任何内容。`add` 通过提示和三个 `AskUserQuestion` 选择收集条目，然后询问"May I append this entry to `docs/tech-debt-register.md`?"。`prioritize` 根据 Impact 和 Effort 列将每个登记项评分为 `impact ÷ effort`，重新排序并在写回之前询问；它推荐下一个冲刺的内容，或在 `workflow: minimal`（无冲刺）时推荐接下来构建什么，结尾的后续步骤将高优先级项发送到 `/sprint-plan` 或简报的构建顺序。技能解析 `automation` 和 `workflow`。`report` 为只读。不调用任何主管门。Verdict：COMPLETE、BLOCKED（用户拒绝写入）、FAIL（缺少子命令）、NOT ASSESSED。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含 verdict 关键词：COMPLETE、BLOCKED、FAIL、NOT ASSESSED
- [ ] 包含"May I write"措辞（写入 `docs/tech-debt-register.md`）
- [ ] 具有下一步交接（阶段 3：`/sprint-plan`、`/tech-debt report`）

---

## 主管门检查

无。技术债跟踪是内部代码库分析技能；不调用任何门。

---

## Fixture 默认值

除非其自身的 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——扫描将新发现追加到现有登记表

**Fixture：**
- `project.yaml`：`engine.name: godot`；`modes.automation` 未设置（collaborative）
- `docs/tech-debt-register.md` 采用技能的 Debt Register Format，有 2 个 `Open` 行：
  - TD-001——Code Quality，文件 `src/core/save_system.gd`：针对保存路径 bug 的 `# HACK` 变通方案。该 `HACK` 注释已不在文件中
  - TD-002——Code Quality，文件 `src/gameplay/combat.gd`：该文件中的 `# TODO` 注释
- `src/gameplay/combat.gd` 有 2 个 `# TODO` 注释和 1 个 `# FIXME` 注释
- `src/ui/hud.gd` 长 620 行；`src/` 中共有 12 个源文件

**输入：** `/tech-debt scan`

**预期行为：**
1. 技能将代码根目录解析为 `src/` 并扫描债务指标
2. 发现包括 2 个 TODO、FIXME（视为伪装成债务的 bug）以及 `hud.gd` 作为超过 500 行的文件
3. 每个发现被归入六类之一（Architecture、Code Quality、Test、Documentation、Dependency、Performance）
4. 输出说明分母——扫描了 12 个源文件
5. 与登记表匹配：2 个 TODO 与 TD-002 的文件和类型相同，因此更新 TD-002；TD-001 的 `HACK` 已消失，因此 TD-001 被提议为 `Resolved [date]`；FIXME 和 `hud.gd` 成为新行 TD-003 和 TD-004
6. 技能展示发现和 TD-001 的解决，然后询问"May I write these findings to `docs/tech-debt-register.md`?"——一次批准涵盖两者
7. 批准后，TD-001 的 Status 变为 `Resolved YYYY-MM-DD` 且该行保留；TD-002 被更新；TD-003 和 TD-004 被追加，包含全部十列，Status 为 `Open`，`Added` 设为今天（`YYYY-MM-DD`），Priority 为 `—`
8. Verdict：**COMPLETE**——扫描发现已写入登记表

**断言：**
- [ ] 扫描覆盖已解析的代码根目录 `src/`，查找 TODO、FIXME、HACK、`@deprecated`、超过 500 行的文件和超过 50 行的函数
- [ ] 每个发现被分配六类债务之一
- [ ] 扫描的文件数与发现一同说明
- [ ] "May I write"指明 `docs/tech-debt-register.md`，且在展示发现之后出现
- [ ] 两个 TODO 发现更新 TD-002，而不是添加重复行
- [ ] TD-001 在同一次批准中被提议为 `Resolved [date]`，并保留在登记表中而非删除
- [ ] 新行采用下一个 ID（TD-003、TD-004）并带有 `Added` 日期；登记表就地更新，绝不替换
- [ ] Verdict 为 COMPLETE

---

### 用例 2：无子命令——用法与 FAIL

**Fixture：**
- `docs/tech-debt-register.md` 存在
- `src/` 包含带 TODO 注释的源文件

**输入：** `/tech-debt`

**预期行为：**
1. 阶段 1 未找到子命令
2. 技能输出用法，列出 `scan`、`add`、`prioritize` 和 `report`
3. 技能停止——不运行扫描，不读取或写入登记表
4. Verdict：**FAIL**——缺少必需子命令

**断言：**
- [ ] 用法列出四个子命令
- [ ] 不执行扫描，不写入任何文件
- [ ] Verdict 为 FAIL，说明缺少子命令

---

### 用例 3：空代码根目录——NOT ASSESSED，而非干净扫描

**Fixture：**
- `project.yaml`：`engine.name: godot`
- `src/` 存在但不包含源文件
- `docs/tech-debt-register.md` 存在，有 3 个条目

**输入：** `/tech-debt scan`

**预期行为：**
1. 技能将代码根目录解析为 `src/` 并统计扫描覆盖的文件数：零
2. 技能报告 **NOT ASSESSED — no source files to scan**，解释"未发现债务指标"描述的是空搜索，而非代码库
3. 技能停止：无"May I write"提示，无登记表写入，无 COMPLETE verdict

**断言：**
- [ ] 零文件扫描报告为 NOT ASSESSED — no source files to scan
- [ ] 输出不声称"未发现债务"或代码库干净
- [ ] 不写入登记表，也不提供写入
- [ ] 不发出 COMPLETE verdict

---

### 用例 3b：代码根目录无法解析——NOT ASSESSED，不凭猜测扫描

**Fixture：**
- `project.yaml` 没有 `engine.name`；`.claude/docs/technical-preferences.md` 内容为 `[TO BE CONFIGURED]`
- `src/` 和 `Source/` 都存在且包含源文件，因此目录树无法决定根目录
- `docs/tech-debt-register.md` 存在，有 3 个条目

**输入：** `/tech-debt scan`

**预期行为：**
1. 无法按 `.claude/docs/code-root-resolution.md` 解析代码根目录
2. 技能报告 `NOT ASSESSED — code root unresolved` 并停止
3. 不凭猜测扫描任何目录；无"May I write"提示，无登记表写入，无 COMPLETE verdict

**断言：**
- [ ] Verdict 为 `NOT ASSESSED — code root unresolved`
- [ ] 不从猜测的根目录报告任何债务发现
- [ ] 不写入登记表，也不提供写入

---

### 用例 4：添加模式——用户拒绝写入

**Fixture：**
- `docs/tech-debt-register.md` 存在，有 4 个条目
- `modes.automation` 未设置（collaborative）

**输入：** `/tech-debt add`

**预期行为：**
1. 技能以纯文本询问描述和受影响文件
2. 技能使用 `AskUserQuestion` 询问类别——六个选项，从 Architecture 到 Performance
3. 技能使用 `AskUserQuestion` 询问修复工作量——S、M、L、XL
4. 技能使用 `AskUserQuestion` 询问若不修复的影响——Low、Med、High、Critical
5. 技能展示完整的新条目
6. 技能询问"May I append this entry to `docs/tech-debt-register.md`?"
7. 用户拒绝——不写入任何内容
8. Verdict：**BLOCKED**——用户拒绝写入

**断言：**
- [ ] 类别通过 `AskUserQuestion` 收集，提供六类债务
- [ ] 工作量通过 `AskUserQuestion` 收集，提供 S / M / L / XL
- [ ] 影响通过 `AskUserQuestion` 收集，提供 Low / Med / High / Critical，以便 `prioritize` 能为该行评分
- [ ] 在追加提示之前展示完整条目
- [ ] 拒绝后登记表不变，verdict 为 BLOCKED

---

### 用例 4a：添加模式——匹配现有 Open 条目

**Fixture：**
- `docs/tech-debt-register.md` 存在一个 `Open` 条目，其文件和类别与用户即将添加的条目相同

**输入：** `/tech-debt add`

**预期行为：**
1. 用例 4 的步骤 1–5 照常运行
2. **追加前匹配：** 在追加询问之前，技能展示现有的 `Open` 条目（相同文件和类别），并询问是否更新它而非添加重复项
3. 选择"更新"时，匹配的条目就地更新——不添加重复行
4. 选择"仍然添加"（用户拒绝更新）时，技能仍询问"May I append this entry to `docs/tech-debt-register.md`?"，若同意，则在匹配项旁追加新条目

**断言：**
- [ ] 与现有 `Open` 条目文件和类别相同的新条目，绝不未经先展示匹配并询问就追加
- [ ] "更新"编辑匹配行而非添加新行
- [ ] 拒绝更新不会结束运行：仍提供追加，且在写入前询问

---

### 用例 4b：无登记表的报告模式——NOT ASSESSED

**Fixture：**
- `docs/tech-debt-register.md` 不存在

**输入：** `/tech-debt report`

**预期行为：**
1. 技能查找登记表但未找到
2. Verdict：**NOT ASSESSED**——无登记表可报告；先运行 `/tech-debt scan`
3. 不打印趋势或总计，不写入任何内容

**断言：**
- [ ] Verdict 为 NOT ASSESSED，而非零项报告或"稳定"趋势
- [ ] 输出指明 `/tech-debt scan` 作为创建登记表的方式
- [ ] 不写入任何文件

---

### 用例 5：完整审查模式下的优先级排序——评分、重新排序、批准

**Fixture：**
- `project.yaml`：`modes.review_mode: full`；`modes.workflow: standard`（固定：未设置解析为 minimal，推荐下一个构建而非冲刺）
- `docs/tech-debt-register.md` 有 4 个条目：TD-001 High/S、TD-002 Low/L、TD-003 Critical/M、TD-004 Med/—（Effort 尚未估算）

**输入：** `/tech-debt prioritize`

**预期行为：**
1. 技能读取 `docs/tech-debt-register.md`
2. 每项评分为 `impact ÷ effort`（Impact Low 1 … Critical 4；Effort S 1 … XL 4）：TD-001 3.0、TD-003 2.0、TD-002 0.3；TD-004 列在其后为"未评分——Effort 未判定"
3. 登记表按分数重新排序，技能推荐哪些条目属于下一个冲刺
4. 无论审查模式如何，不调用主管门
5. 技能展示重新排序的登记表，然后询问"May I write the re-prioritized register back to `docs/tech-debt-register.md`?"
6. 批准后写入文件
7. Verdict：**COMPLETE**——登记表已重新排序并保存

**断言：**
- [ ] 条目根据自身列评分 `impact ÷ effort` 并重新排序：TD-001、TD-003、TD-002
- [ ] TD-004 不获得虚构分数；列为未评分，并指明缺失的 Effort
- [ ] 给出下一个冲刺推荐
- [ ] 任何审查模式下都不调用主管门
- [ ] 仅在"May I write"提示被批准后才写入
- [ ] Verdict 为 COMPLETE
- [ ] 后续步骤将高优先级项发送到 `/sprint-plan`

---

### 用例 5b：Workflow Minimal 下的优先级排序——接下来构建，而非冲刺

**Fixture：**
- `project.yaml`：`modes.rigor: minimal`（workflow 解析为 `minimal`，无冲刺）；`modes.automation` 未设置（collaborative）
- `docs/tech-debt-register.md` 有 3 个 `Open` 条目：TD-001 High/S、TD-002 Low/L、TD-003 Critical/M
- `design/game-brief.md` 存在，构建顺序为三个编号项；无 `production/sprints/` 文件夹

**输入：** `/tech-debt prioritize`

**预期行为：**
1. 技能解析 `workflow: minimal` 并读取登记表
2. 条目按用例 5 评分并重新排序：TD-001 3.0、TD-003 2.0、TD-002 0.3
3. 推荐说明接下来构建哪些条目，而非哪些属于下一个冲刺
4. 技能询问"May I write the re-prioritized register back to `docs/tech-debt-register.md`?"，批准后写入
5. 后续步骤告诉用户将高优先级项添加到简报的构建顺序，而非运行 `/sprint-plan`；仍包含 `/tech-debt report`
6. Verdict：**COMPLETE**——登记表已重新排序并保存

**断言：**
- [ ] 推荐为"接下来构建什么"；不使用"sprint"一词作为放置条目的位置
- [ ] 后续步骤指明简报的构建顺序（`design/game-brief.md`）作为高优先级项的放置处
- [ ] 不推荐 `/sprint-plan`
- [ ] 评分、重新排序和批准后写入规则与用例 5 相同
- [ ] Verdict 为 COMPLETE

---

## 协议合规性

- [ ] 需要子命令；无子命令时打印用法并以 FAIL 停止
- [ ] `scan` 按引擎解析代码根目录（`src/`、`Assets/`、`Source/`）并说明覆盖的文件数
- [ ] 零文件或根目录无法解析的扫描为 NOT ASSESSED，不写入任何内容
- [ ] 每次登记表写入之前都有指明 `docs/tech-debt-register.md` 的"May I write" / "May I append"提示，且在展示内容之后
- [ ] `scan` 追加且绝不覆盖现有条目；匹配 `Open` 行的发现更新它，模式已消失的 `Open` 行被提议为 `Resolved [date]`，绝不删除
- [ ] 新行遵循 Debt Register Format：下一个 `TD-NNN` ID、全部十列、`YYYY-MM-DD` 格式的 `Added` 日期
- [ ] 拒绝写入以 BLOCKED 结束
- [ ] 不调用任何主管门
- [ ] 以阶段 3 后续步骤结尾（`/sprint-plan`、`/tech-debt report`）；在 workflow `minimal` 时，冲刺步骤变为"将它们添加到简报的构建顺序"
- [ ] 通过 `resolve_config` 解析 `automation` 和 `workflow`

---

## 覆盖说明

- 带登记表的 `report` 模式（按类别和工作量的只读摘要，从每行的 `Added` 和 `Resolved` 日期统计新增和已解决数量，标记超过 3 个冲刺的条目，Verdict COMPLETE——债务报告已生成）此处未测试。
- Unity 和 Unreal 扫描仅在解析的根目录（`Assets/`、`Source/`）上不同，未单独测试。