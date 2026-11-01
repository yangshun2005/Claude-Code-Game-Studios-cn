# 技能测试规范：/perf-profile

## 技能摘要

`/perf-profile` 是静态分析性能剖析器。它以系统名称或 `full` 作为范围，在产出任何内容前将所需输入逐一记录为 `FOUND` 或 `ABSENT`，从顶部注入的配置块解析 `performance.enforce`（Phase 0；该块还携带 `automation` 和 `workflow`），并用 `get_effective_yaml_key` 读取已提交的预算：`performance.target_framerate`、`performance.frame_budget_ms`、`performance.draw_call_limit` 和 `performance.memory_ceiling_mb`，之后才回退到设计文档或 `CLAUDE.md`（Phase 2）。它分析源代码中的 CPU、内存、渲染和 I/O 热点候选（Phase 3）——不读取运行时剖析器导出文件。报告为每个有预算的行评级 `OK` / `WARNING` / `OVER`；没有已提交预算的指标为 `NOT ASSESSED`，摘要仅针对已设置的预算给出余量（否则为 `no budget set — headroom not assessed`）。技能在写入前询问"May I write this profiling report to `production/polish/[scope]-report-[date].md`?"（Phase 4），为 M 或 L 工作量的热点提供实现 / 削减范围 / 推迟 / 上报选项（Phase 5），并移交给 `/architecture-decision`、`/scope-check` 或 `/sprint-plan update`（Phase 6）。不调用任何主管门禁。结论：COMPLETE，或当所有必需输入均缺失时为 NOT ASSESSED — NO DATA。在 `workflow: minimal`（无冲刺）下，已接受的优化进入简报的构建顺序，Phase 6 不指向 `/sprint-plan update`。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含结论关键词：COMPLETE、NOT ASSESSED — NO DATA，以及每指标状态 OK / WARNING / OVER
- [ ] 包含"May I write"措辞并指明 `production/polish/[scope]-report-[date].md`
- [ ] 具有下一步移交（Phase 6：`/architecture-decision`、`/scope-check`、`/sprint-plan update`）

---

## 主管门禁检查

无。性能剖析是建议性分析技能；任何审查模式下都不调用门禁。

---

## 夹具默认值

除非自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——预算已提交，发现战斗热点

**夹具：**
- `project.yaml`：`engine.name: godot`、`performance.target_framerate: 60`、`performance.frame_budget_ms: 16.6`、`performance.draw_call_limit: 200`、`performance.memory_ceiling_mb: 2048`；`performance.enforce` 未设置（解析为 `warn`）；`modes.automation` 未设置（collaborative）
- 无配置键、设计文档或 `CLAUDE.md` 章节声明加载时间目标
- `src/gameplay/combat/enemy_manager.gd` 有一个 `_process()`，遍历每个敌人，并在该循环内遍历每个活跃投射物，且每帧为每个敌人投射一条射线
- 将嵌套循环重构为空间分区是数天的工作量

**输入：** `/perf-profile combat`

**预期行为：**
1. 技能记录其输入：源代码 FOUND，已提交预算 FOUND
2. Phase 0：`performance.enforce` 解析为 `warn`——违规是发现项，不是阻塞项
3. Phase 2：在任何回退之前，用 `get_effective_yaml_key` 读取四个 `performance.*` 键
4. Phase 3：列出战斗 `_process()` 函数；识别出嵌套循环和每帧射线投射
5. Phase 4：报告的 Performance Budgets 表以 16.6 ms、2048 MB 和 200 次绘制调用作为预算，这三行各带 OK / WARNING / OVER 状态；Load time 行无已提交预算，为 `NOT ASSESSED`；热点和建议引用 `enemy_manager.gd` 并带行号
6. 摘要给出前 3 个热点、针对三个已设置预算的估计余量、加载时间的 `no budget set — headroom not assessed`，以及推荐的下一步操作
7. 技能询问"May I write this profiling report to `production/polish/combat-report-[date].md`?"
8. 嵌套循环热点被评为 Fix Effort M 或 L，因此 Phase 5 提供：实现、通过 `/scope-check` 削减范围、推迟到 Polish，或通过 `/architecture-decision` 上报
9. 结论：COMPLETE

**断言：**
- [ ] 预算来自用 `get_effective_yaml_key` 读取的四个 `performance.*` 键，而非设计文档或硬编码的 16.67 ms
- [ ] 嵌套循环和每帧射线投射作为热点报告，并带 `file:line` 位置
- [ ] 每条优化建议说明位置、预期收益、风险和方案
- [ ] 帧时间、内存和绘制调用行带 OK / WARNING / OVER 状态；Load time 行为 `NOT ASSESSED`，而非 OK
- [ ] 不为加载时间声称余量；摘要说 `no budget set — headroom not assessed`
- [ ] "May I write"指明 `production/polish/combat-report-[date].md` 且先于任何写入
- [ ] M/L 工作量热点触发 Phase 5 选择（实现 / `/scope-check` / 推迟到 Polish / `/architecture-decision`）
- [ ] 结论为 COMPLETE

---

### 用例 2：无已提交预算——预算行为 NOT ASSESSED

**夹具：**
- `project.yaml` 有 `engine.name: godot` 且无 `performance` 块
- 无设计文档且无 `CLAUDE.md` 章节声明帧率、内存、加载时间或绘制调用目标
- `src/` 包含带 `_process()` 函数的游戏脚本

**输入：** `/perf-profile full`

**预期行为：**
1. `get_effective_yaml_key` 对四个预算键返回空；设计文档和 `CLAUDE.md` 回退也未找到目标
2. 源代码为 FOUND，因此 Phase 3 分析仍运行并报告热点
3. Performance Budgets 表的每一行都是 `NOT ASSESSED`——没有一行对照模板的 `[16.67ms]` 占位符测量
4. 摘要不针对无人设置的预算声称余量：它说 `no budget set — headroom not assessed`
5. 技能询问"May I write this profiling report to `production/polish/full-report-[date].md`?"；`NOT ASSESSED` 行保留在提供写入的文件中

**断言：**
- [ ] 无预算行显示 OK、WARNING 或 OVER；每行都是 `NOT ASSESSED`
- [ ] `[16.67ms]` 占位符从不作为预算使用，且不声称任何余量数字
- [ ] 热点分析仍运行并报告
- [ ] `NOT ASSESSED` 行保留在提供写入的报告中，不被删除

---

### 用例 3：无可剖析内容——整体结论 NOT ASSESSED — NO DATA

**夹具：**
- 全新项目：`project.yaml` 有 `engine.name: godot` 且无 `performance` 块，代码根目录 `src/` 不存在
- 无设计文档
- 任何位置都无剖析器输出

**输入：** `/perf-profile full`

**预期行为：**
1. 技能列出其输入并逐一记录：源代码 ABSENT、预算 ABSENT、剖析器输出 ABSENT
2. 每个必需输入均为 ABSENT，因此技能在 Phase 3 前停止
3. 技能报告整体结论为 `NOT ASSESSED — NO DATA`，指明缺失内容及由哪个技能产出
4. 不填写报告模板，也不提供任何写入内容

**断言：**
- [ ] 每个输入记录为 FOUND 或 ABSENT，而非假定存在
- [ ] 整体结论为 `NOT ASSESSED — NO DATA`，而非 COMPLETE
- [ ] 输出指明每个缺失输入及由哪个技能产出
- [ ] 无填写好的报告、无余量数字、无"May I write"提示

---

### 用例 4：本地强制覆盖为 `block`——违规是阻塞项

**夹具：**
- `project.yaml`：`engine.name: godot`、`performance.enforce: warn`、`performance.draw_call_limit: 200`，其他三个预算已设置
- `project.local.yaml`：`performance.enforce: block`，因此技能顶部解析后的块显示 `block`
- `src/levels/forest/forest_builder.gd` 实例化 600 个独立植被 `MeshInstance3D` 节点，每个有自己的材质且无实例化
- `modes.automation` 未设置（collaborative）

**输入：** `/perf-profile forest`

**预期行为：**
1. Phase 0 使用技能顶部解析配置块中的 `block`，而非重新读取 `project.yaml` 会得到的 `warn`
2. forest 范围的绘制调用估计超过 200；Draw calls 行为 OVER
3. 报告明确说明该违规是阻塞项——阻止发布——且 `/gate-check` 将因此使 Polish 门禁 FAIL
4. 给出建议（例如批处理或实例化植被）及其预期收益
5. 技能在写入 `production/polish/forest-report-[date].md` 前询问"May I write"

**断言：**
- [ ] 本地 `block` 生效，而非已提交的 `warn`——`performance.enforce` 来自解析后的块
- [ ] Draw calls 行对照 200 预算为 OVER
- [ ] 报告说明该违规是阻塞项且 `/gate-check` 将因此使 Polish 门禁 FAIL
- [ ] "May I write"先于任何写入

---

### 用例 5：完整审查模式下强制级别为 `off`——仅信息性，无门禁

**夹具：**
- `project.yaml`：`engine.name: godot`、`modes.review_mode: full`、`modes.workflow: standard`（固定：未设置解析为 `minimal`，无冲刺，因此 Phase 6 会指明简报的构建顺序而非 `/sprint-plan update`）、`performance.enforce: off`，四个预算均已设置
- `src/` 包含一个 `_process()`，其估计帧成本超过 `performance.frame_budget_ms`

**输入：** `/perf-profile full`

**预期行为：**
1. 无论审查模式如何，都不调用主管门禁
2. 估计帧时间仍报告在 Performance Budgets 表中
3. 由于 `off` 下预算仅为信息性，超预算帧时间不作为违规发现或修复建议提出
4. 技能在写入报告前询问"May I write"
5. 输出以 Phase 6 下一步结束：`/architecture-decision`、`/scope-check [feature]`、`/sprint-plan update`

**断言：**
- [ ] 任何审查模式下都不调用主管门禁
- [ ] 估计帧时间仍出现在预算表中
- [ ] 超预算指标不作为违规发现或修复建议提出
- [ ] "May I write"先于任何写入
- [ ] 下一步指明 `/architecture-decision`、`/scope-check` 和 `/sprint-plan update`

---

### 用例 6：`workflow`——已接受优化的去向

**夹具：**
- 同用例 1：预算已提交，`enemy_manager.gd` 有嵌套循环和每帧射线投射，嵌套循环热点被评为 Fix Effort M 或 L
- `modes.rigor: minimal`，因此 `modes.workflow` 解析为 `minimal`（无冲刺）
- 用户为嵌套循环热点选择 A
- 变体：`modes.workflow: standard`（固定：这是有冲刺的层级）

**输入：** `/perf-profile combat`

**预期行为：**
1. Phase 5 运行，因为热点为 M 或 L，选项 A 读作"Implement the optimization"，在 `minimal` 下注明无冲刺，工作加入简报的构建顺序
2. 用户在 `minimal` 下选择 A 后，优化被导向简报的构建顺序，而非冲刺
3. Phase 6 的调度行说要加入简报的构建顺序，而非运行 `/sprint-plan update`；`/sprint-plan` 不被指明为调度方式
4. `standard` 变体：选项 A 提供立即修复或安排日程，Phase 6 指明 `/sprint-plan update`
5. 两次运行结论：COMPLETE

**断言：**
- [ ] 在 `minimal` 下，选项 A 和 Phase 6 都指向简报的构建顺序
- [ ] 在 `minimal` 下，不推荐 `/sprint-plan update`
- [ ] `standard` 变体：Phase 6 指明 `/sprint-plan update`，选项 A 不提及构建顺序
- [ ] 其他 Phase 5 选项（`/scope-check`、推迟到 Polish、`/architecture-decision`）以及 Phase 6 中 `/architecture-decision` 和 `/scope-check [feature]` 的行在两次运行中相同
- [ ] 结论为 COMPLETE

---

### 用例 7：会话在子文件夹中启动——预算仍从项目根读取

**夹具：**
- 项目根目录的 `project.yaml`：`engine.name: godot`、`performance.target_framerate: 60`、`performance.frame_budget_ms: 16.6`、`performance.draw_call_limit: 200`、`performance.memory_ceiling_mb: 2048`
- 会话的工作目录是 `src/gameplay/`（子文件夹），而非根目录
- `src/gameplay/combat/enemy_manager.gd` 有一个带嵌套循环的 `_process()`

**输入：** `/perf-profile combat`

**预期行为：**
1. Phase 2 将四个 `get_effective_yaml_key` 读取作为单条命令运行，先切换到项目根目录并从那里加载 `.claude/hooks/yaml-helper.sh`
2. 四个键都返回其已提交值；没有因从子文件夹找不到辅助脚本或 `project.yaml` 而返回空
3. Performance Budgets 表使用 16.6 ms、2048 MB 和 200 次绘制调用；这些行不是 `NOT ASSESSED`

**断言：**
- [ ] 预算读取是单条命令，在 source 辅助脚本前切换到项目根目录，而非从当前文件夹运行四次读取
- [ ] 四个值是已提交值，而非空
- [ ] 无预算行因工作目录而为 `NOT ASSESSED`：`project.yaml` 中设置的预算从不报告为未设置
- [ ] 嵌套循环仍被发现并带 `file:line` 位置报告

---

## 协议合规

- [ ] 在产出任何报告前将每个输入记录为 FOUND 或 ABSENT
- [ ] 在回退到设计文档或 `CLAUDE.md` 前，用 `get_effective_yaml_key` 读取四个 `performance.*` 预算键
- [ ] 无已提交预算的指标为 `NOT ASSESSED`——从不对照 `[16.67ms]` 占位符测量——且摘要仅针对已设置的预算声称余量
- [ ] 应用解析块中的 `performance.enforce`：`warn` → 发现项，`block` → 阻塞项，`off` → 信息性
- [ ] 在写入前询问"May I write"并指明 `production/polish/[scope]-report-[date].md`，且将 `NOT ASSESSED` 章节保留在写入的文件中
- [ ] 不调用任何主管门禁
- [ ] 以 Phase 6 下一步移交结束；调度行取决于 `workflow`（`standard`/`full` 下为 `/sprint-plan update`，`minimal` 下为简报的构建顺序）

---

## 覆盖说明

- 未识别的 `performance.enforce` 值（向用户呈现后按 `warn` 处理）在此不测试。
- 技能仅为静态分析；用运行时剖析确认热点列在报告的"Requires Investigation"章节下，这些用例不涉及。
- 在 Phase 5 选项 C 下推迟 M/L 热点会将其记录在 `### Deferred to Polish` 下；该记录不单独断言。