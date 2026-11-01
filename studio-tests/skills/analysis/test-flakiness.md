# 技能测试规范：/test-flakiness

## 技能摘要

`/test-flakiness` 从多次 CI 或本地测试运行的结果中找出不稳定测试。它接受日志路径、`scan`（所有可用的结果文件——`reports/` 中或保存到 `test-results/` 的 gdUnit4 JUnit XML、`test-results/` 中的 Unity NUnit XML、`Saved/Logs/` 中的 Unreal 自动化日志）或 `registry`（针对已在 `tests/regression-suite.md` 隔离区中的测试的修复指导）。没有结果数据时，它会列出三种获取数据的方式并停下来询问。它跨运行构建每个测试的历史记录，按失败率对既有通过又有失败的测试分类——>25% 为 High（立即隔离）、5–25% 为 Moderate（直接修复，暂不隔离）、1–5% 为 Low（监控）——并从原因表中给出可能原因和修复方向，使用 Grep 搜索测试文件。运行次数少于 3 次时，无论失败率多少，所有发现均为"疑似"而非"已确认"，且不隔离任何测试；摘要表的 Confidence 列会注明这一点。它分别询问"我可以更新 `tests/regression-suite.md` 的隔离区吗……？"和"我可以将完整的 flakiness 报告写入 `production/qa/flakiness-report-[date].md` 吗？"，追加隔离条目而不删除现有条目，且从不删除测试文件。不调用任何 director 门禁。判定：COMPLETE（已写入报告）或 BLOCKED（用户拒绝写入）。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED，以及 flakiness 等级 High / Moderate / Low
- [ ] 包含针对 `tests/regression-suite.md` 和 `production/qa/flakiness-report-[date].md` 的"May I"写入措辞
- [ ] 具有下一步交接（第 7 节：添加跳过注解，在发布门禁前安排修复）

---

## Director 门禁检查

无。Flakiness 检测是建议性质量技能；不调用任何门禁。

---

## Fixture 默认值

除非其自身的 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——从 JUnit 历史记录发现 High flakiness，经批准后隔离

**Fixture：**
- `project.yaml`：`engine.name: godot`；`modes.automation` 未设置（collaborative）
- `test-results/` 中有 6 个 gdUnit4 JUnit XML 文件，来自同一提交的 6 次运行
- `test_loot_drop_rolls_rare_item` 在 6 次运行中失败 2 次（33%）；其余 40 个测试在全部 6 次运行中通过
- `tests/unit/loot/loot_drop_test.gd` 调用 `randf()` 且未设种子
- `tests/regression-suite.md` 有一个 Quarantined Tests 表，含 1 个现有条目

**输入：** `/test-flakiness scan`

**预期行为：**
1. 技能在 `test-results/` 中找到 XML 文件，并解析 `<testcase name=` 和 `<failure` / `<error` 条目
2. 技能构建 `test_id → [run1 … run6]`，找到一个既有通过又有失败的测试
3. 6 次运行中 33% 高于 25% → **High** flakiness，Confidence 已确认 → 立即隔离
4. Grep `loot_drop_test.gd` 找到 `randf` → 可能原因 Random seed；修复方向：传入显式种子
5. 隔离建议指明 gdUnit4 的 GDScript 跳过参数（`_do_skip := true`、`_skip_reason := "flaky: ..."`）作为跳过方式
6. 摘要显示：已分析运行 6 次、跟踪测试 41 个、该不稳定测试失败率 33%、40 个测试结果一致
7. 技能询问"我可以将发现的 flaky 测试更新到 `tests/regression-suite.md` 的隔离区吗？"，并另行询问"我可以将完整的 flakiness 报告写入 `production/qa/flakiness-report-[date].md` 吗？"
8. 批准后，用 `Edit` 追加新条目；现有条目保留
9. 判定：**COMPLETE**——已写入 flakiness 报告

**断言：**
- [ ] `test-results/` 中的 JUnit XML 结果按测试跨全部 6 次运行解析
- [ ] 该不稳定测试被点名，附失败率、Confidence 已确认，并归类为 High → 隔离
- [ ] 指明的跳过机制是 gdUnit4 的 `_do_skip` / `_skip_reason` 参数对，而非虚构的标志
- [ ] 可能原因为 Random seed，修复方向为显式种子
- [ ] 两次写入分别询问，各自在发生前询问
- [ ] 隔离条目被追加，现有条目未被删除
- [ ] 判定为 COMPLETE

---

### 用例 2：Moderate Flakiness——直接修复，不隔离

**Fixture：**
- `project.yaml`：`engine.name: godot`
- `test-results/` 中有 20 个 JUnit XML 文件，来自同一提交的 20 次运行
- `test_physics_bounce_height_matches_spec` 在 20 次运行中失败 3 次（15%）
- 该测试断言 `bounce_height == 0.5`

**输入：** `/test-flakiness scan`

**预期行为：**
1. 技能计算失败率 15% → **Moderate** flakiness
2. Grep 测试文件找到浮点相等比较 → 可能原因 Floating point
3. 建议："该测试间歇性不可靠……暂不隔离——直接修复测试"，修复方向为 epsilon 比较（`is_equal_approx`）

**断言：**
- [ ] 15% 失败率归类为 Moderate
- [ ] 建议为直接修复，而非隔离
- [ ] 原因为 Floating point，修复指明 epsilon 比较，如 `is_equal_approx`

---

### 用例 3：无结果数据——列出选项并停止

**Fixture：**
- 无 `test-results/` 目录、无 `.github/` 目录、无 `Saved/Logs/`
- 未给出日志路径

**输入：** `/test-flakiness scan`

**预期行为：**
1. 技能未找到 CI 或本地结果数据
2. 技能打印"No CI log data found…"，并给出三个选项：至少运行测试套件 3 次并收集日志；将 CI 日志保存到 `test-results/`；运行 `/test-flakiness registry`
3. 技能停止并询问用户选择哪个选项
4. 不生成 flakiness 表，不写入任何内容

**断言：**
- [ ] 输出说明未找到 CI 日志数据
- [ ] 列出全部三个选项，包括 `/test-flakiness registry`
- [ ] 技能停下来询问，而非报告干净结果
- [ ] 不写入任何文件

---

### 用例 4：运行次数过少——疑似，未确认

**Fixture：**
- `project.yaml`：`engine.name: godot`
- `test-results/` 中仅有 2 个 JUnit XML 文件，来自同一提交的 2 次运行
- `test_save_roundtrip_preserves_inventory` 在一次运行中通过，在另一次中失败

**输入：** `/test-flakiness scan`

**预期行为：**
1. 技能解析两个文件，找到既有通过又有失败的测试——失败率 50%
2. 运行次数少于 3 次，发现被标记为"疑似"而非"已确认"：摘要表的 Confidence 显示为 suspected，High 等级的立即隔离不适用
3. 该测试不被隔离；建议在采取行动前收集更多运行数据
4. 技能询问是否有更多运行数据可用
5. Data Limitations 部分注明可用运行少于 5 次，因此置信度低

**断言：**
- [ ] 发现被标记为 suspected 而非 confirmed，位于摘要表的 Confidence 列
- [ ] 50% 失败率不触发隔离——不建议跳过，任何回归套件备注都将该测试标记为疑似
- [ ] 技能询问是否有更多运行数据可用
- [ ] Data Limitations 说明分析的运行少于 5 次

---

### 用例 5：完整审查模式下的 Registry 模式——为已知隔离测试提供指导，无门禁

**Fixture：**
- `project.yaml`：`modes.review_mode: full`
- `tests/regression-suite.md` 的 Quarantined Tests 表列出 `test_ai_path_recalc_avoids_blocked_tile`（原因：timing）和 `test_scene_load_spawns_player`（原因：scene not ready）

**输入：** `/test-flakiness registry`

**预期行为：**
1. 技能读取 `tests/regression-suite.md` 的隔离区
2. 对每个隔离测试，从原因表给出修复指导：Timing / async → 显式 await 或同步，而非基于时间的延迟；Scene/prefab load race → 实例化后等待一帧（`await get_tree().process_frame`）
3. 无论审查模式如何，不调用任何 director 门禁
4. 现有隔离条目不被删除，技能不删除或编辑任何测试文件

**断言：**
- [ ] 输入为 `tests/regression-suite.md` 的隔离区
- [ ] 每个隔离测试获得与其原因匹配的修复方向
- [ ] 任何审查模式下都不调用 director 门禁
- [ ] 不删除任何隔离条目，不删除任何测试文件

---

## 协议合规性

- [ ] 分析前定位结果数据（`test-results/` XML、`Saved/Logs/` 或给定日志路径）；没有则列出三个选项并停下来询问
- [ ] 按失败率对既有通过又有失败的测试分类：>25% High（隔离）、5–25% Moderate（修复，不隔离）、1–5% Low（监控）
- [ ] 从原因表给出可能原因和修复方向，使用 Grep 搜索测试文件
- [ ] 仅当测试在多次运行间既有通过又有失败且其间无代码变更时，才计为不稳定
- [ ] 运行次数少于 3 次时，在摘要的 Confidence 列将发现标记为疑似而非已确认，且不隔离任何测试
- [ ] 更新 `tests/regression-suite.md` 前和写入 `production/qa/flakiness-report-[date].md` 前分别询问
- [ ] 追加隔离条目，从不删除现有条目，从不删除测试文件
- [ ] 不调用任何 director 门禁
- [ ] 写入时以 COMPLETE 结束，拒绝时以 BLOCKED 结束；无结果数据时列出三个选项并停下来询问（用例 3），从不报告干净结果

---

## 覆盖说明

- 无参数路径（CI 日志可访问时运行 `scan`，否则 `registry`）此处未测试。
- Unity NUnit XML 遵循与用例 1 相同的流程，从 `<test-case` 元素及其 `result` 属性读取；未单独测试。
- Unreal 不同，此处未测试：其结果在 `Saved/Logs/` 中是纯文本 `Result={Success}` / `Result={Fail}` 行（每个测试打印 `Test Completed. Result={<status>}`），其跳过机制为 NOT SOURCEABLE，因此被隔离的 Unreal 测试会被记录，并询问用户其 CI 如何排除它。gdUnit4 C# 测试同理。
- Low 等级（1–5%）需要 20 次或更多运行才能达到，未测试。