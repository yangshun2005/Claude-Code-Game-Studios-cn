# 技能测试规范：/create-control-manifest

## 技能摘要

`/create-control-manifest` 读取 `docs/architecture/` 中已接受（Accepted）的 ADR，生成控制清单——一份扁平、可执行的规则表，记录每个架构层（Foundation、Core、Feature、Presentation）的必需模式、禁止做法、性能护栏和引擎 API 约束，以及来自项目配置和引擎参考文档的全局规则。故事作者借助它，让故事继承正确的架构规则，无需阅读每一份 ADR。

该技能先统计 ADR 数量，在读取任何 ADR 正文之前用 Grep 解析每份 ADR 的 `## Status`，仅基于 Accepted 集合构建清单。它会预览各层规则数量，在审查模式为 `full` 时运行 **TD-MANIFEST** 导演门禁（`solo` 和 `lean` 模式下跳过并注明），然后在写入 `docs/architecture/control-manifest.md` 之前询问"May I write the Control Manifest?"。判定结果：写入后为 COMPLETE，用户拒绝写入时为 BLOCKED。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"May I write"协作协议用语（针对控制清单）
- [ ] 结尾有下一步交接（先 `/create-epics` 再 `/create-stories`）
- [ ] 说明仅包含 Accepted ADR（不含 Proposed）
- [ ] 指明门禁 TD-MANIFEST 及其 solo/lean 跳过说明

---

## 导演门禁检查

一个门禁：**TD-MANIFEST**（`technical-director`，`.claude/docs/director-gates/td-manifest.md`），在用户接受第 4 阶段规则摘要之后、第 5 阶段写入询问之前启动。

- `full` → 启动；接收 Control Manifest Preview、覆盖的 ADR、引擎版本，以及任何源自偏好设置或引擎参考文档的规则
- `lean` → 跳过；输出注明"TD-MANIFEST skipped — Lean mode."
- `solo` → 跳过；输出注明"TD-MANIFEST skipped — Solo mode."

判定处理：APPROVE → 进入写入询问；CONCERNS → `AskUserQuestion`，选项为 `Revise flagged rules` / `Accept and proceed` / `Discuss further`；REJECT → 不写入清单，修复被标记的规则并重新呈现摘要；NOT ASSESSED [missing input] → 绝不视为 APPROVE：指出缺失的输入，补充后重新运行门禁，或在输出和 Verdict 行中注明 `TD-MANIFEST: NOT ASSESSED — [input]` 后继续运行。

---

## Fixture 默认值

除非某个用例自己的 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § Fixture defaults。

---

## 测试用例

### 用例 1：正常路径——4 份 Accepted ADR 生成正确的清单

**Fixture：**
- `docs/architecture/` 包含 `adr-0001-*.md` … `adr-0004-*.md`，每份都有 `## Status` 节，内容为 `Accepted`
- 每份 ADR 都有 `## Decision`（含"must"/"always"陈述）和 `## Alternatives Considered`（含被拒绝的备选方案）
- 不存在 `docs/architecture/control-manifest.md`
- 审查模式解析为 `solo`

**输入：** `/create-control-manifest`

**预期行为：**
1. Glob `docs/architecture/adr-*.md`（N = 4），然后在读取任何 ADR 正文之前 Grep `^## Status` 解析 Accepted 集合
2. 仅 Grep Accepted ADR 的 Decision / Alternatives Considered / Performance Implications / Engine Compatibility 节
3. 报告"Loaded 4 Accepted ADRs, engine: [name + version]."
4. 显示 Control Manifest Preview（各层规则数量、覆盖的 ADR）并询问"Does this rule summary look complete?"
5. 注明"TD-MANIFEST skipped — Solo mode."
6. 询问"May I write the Control Manifest?"，选项为 `[A] Yes — write to docs/architecture/control-manifest.md`
7. 批准后写入清单；Verdict：COMPLETE

**断言：**
- [ ] 全部 4 份 Accepted ADR 都列在清单头部的 `ADRs Covered` 下
- [ ] 每个层节都有 `### Required Patterns` 和 `### Forbidden Approaches`
- [ ] 每条规则都带有来源——层规则为 `— source: [ADR-NNNN]`；全局规则注明其 `project.yaml` / `technical-preferences.md` 键或引擎参考文件（如 `deprecated-apis.md`）
- [ ] 在任何写入询问之前显示预览并询问"Does this rule summary look complete?"
- [ ] 出现"TD-MANIFEST skipped — Solo mode."；未启动 technical-director
- [ ] 技能在"May I write the Control Manifest?"批准之前不写入
- [ ] 写入后 Verdict 为 COMPLETE

---

### 用例 2：失败路径——未找到 ADR

**Fixture：**
- `docs/architecture/` 目录存在但不含 `adr-*.md` 文件

**输入：** `/create-control-manifest`

**预期行为：**
1. Glob 返回 N = 0
2. 技能输出："No ADRs found — run `/architecture-decision` before building a control manifest."
3. 技能停止——不执行 Status Grep、不预览、不询问写入、不写文件

**断言：**
- [ ] 打印"No ADRs found"消息
- [ ] 技能推荐 `/architecture-decision` 作为下一步操作
- [ ] 不写入控制清单文件，也不显示"May I write"询问
- [ ] 不输出 Verdict COMPLETE（技能在第 1 阶段停止）

---

### 用例 3：混合 ADR 状态——仅包含 Accepted ADR

**Fixture：**
- `docs/architecture/` 包含 5 份 ADR：3 份 `## Status` 为 Accepted，2 份 `## Status` 为 Proposed（ADR-0004、ADR-0005）

**输入：** `/create-control-manifest`

**预期行为：**
1. N = 5；`^## Status` Grep 返回 5 个匹配，其中 3 个为 Accepted——适用"some Accepted"行，Accepted 集合 A = 这 3 份 ADR
2. 节提取过滤到集合 A
3. 预览报告"Loaded 3 Accepted ADRs"，`ADRs covered` 仅列出这 3 份，`ADRs excluded` 注明 ADR-0004（Proposed）和 ADR-0005（Proposed）
4. 继续执行，不询问 Proposed ADR（该询问仅用于 none-Accepted 情况）

**断言：**
- [ ] 在读取任何 ADR 节之前用 `^## Status` Grep 解析状态
- [ ] 预览的 `ADRs covered` 和清单的 `ADRs Covered` 恰好列出这 3 份 Accepted ADR
- [ ] 两份被排除的 ADR 都在用户批准前看到的预览中注明名称和状态——技能不会静默省略它们
- [ ] 清单中没有任何规则源自这两份 Proposed ADR
- [ ] 此 fixture 不显示"none Accepted — proceed with Proposed or stop?"询问

---

### 用例 4：边界情况——重新生成现有清单

**Fixture：**
- `docs/architecture/control-manifest.md` 已存在，`Manifest Version` 和 `Last Updated` 均为上周日期
- `docs/architecture/` 包含 Accepted ADR，其中一份是此后才接受的
- `production/epics/` 已包含带故事的 epic，因此首次运行的 `/create-epics` 下一步不适用

**输入：** `/create-control-manifest update`

**预期行为：**
1. 技能基于当前 Accepted ADR 重建清单（同样的第 1–4 阶段）
2. 覆盖前询问"May I write the Control Manifest?"
3. 新清单的 `Manifest Version` 和 `Last Updated` 均为当天日期
4. 写入后打印"Updated. Recommend notifying the team of changed rules — especially any new Forbidden entries."

**断言：**
- [ ] 在"May I write the Control Manifest?"批准之前不覆盖现有文件
- [ ] 新接受的 ADR 出现在 `ADRs Covered` 中
- [ ] `Manifest Version` 等于 `Last Updated`，且为生成日期（不是 v1→v2 计数器）
- [ ] 显示重新生成的下一步（"Updated. Recommend notifying the team…"）而非首次运行的 `/create-epics` 交接

---

### 用例 5：导演门禁——full 审查模式下 TD-MANIFEST 返回 REJECT

**Fixture：**
- 存在 4 份 Accepted ADR
- 审查模式解析为 `full`
- TD-MANIFEST 返回 REJECT，指明一条没有来源 ADR 的规则

**输入：** `/create-control-manifest`

**预期行为：**
1. 技能显示第 4 阶段预览；用户选择 `[A] Yes — looks good, run the director review and write the manifest`
2. 通过 `Agent` 启动 `technical-director`，门禁 TD-MANIFEST，传入预览（完整规则列表）、覆盖的 ADR 和引擎版本
3. REJECT 时：不写入清单；修复被标记的规则并重新呈现摘要
4. REJECT 未解除期间不询问"May I write the Control Manifest?"

**断言：**
- [ ] TD-MANIFEST 仅在第 4 阶段摘要被接受后启动
- [ ] 门禁接收规则列表、覆盖的 ADR 和引擎版本
- [ ] REJECT 时不写入清单文件
- [ ] 在任何写入询问之前修订被标记的规则并重新呈现摘要
- [ ] `full` 模式下不出现"TD-MANIFEST skipped"说明

---

### 用例 6：边界情况——无 Accepted ADR，以及 ADR 无 `## Status`

**Fixture（none Accepted）：**
- `docs/architecture/` 包含 3 份 ADR，每份 `## Status` 均为 Proposed

**Fixture（malformed）：**
- `docs/architecture/` 包含 3 份 ADR，均无 `## Status` 节

**输入（两个 fixture）：** `/create-control-manifest`

**预期行为（none Accepted）：**
1. N = 3；`^## Status` Grep 匹配 3 个，均非 Accepted
2. 报告"3 ADRs found, none Accepted. A manifest built from Proposed ADRs would encode decisions that may still change."并询问是继续使用 Proposed ADR 还是停止

**预期行为（malformed）：**
1. N = 3；`^## Status` Grep 无匹配
2. 报告"3 ADRs found, none has a `## Status` section — acceptance cannot be determined. Run `/architecture-decision retrofit [file]` on each."并停止

**断言：**
- [ ] None Accepted：显示询问，不写入空清单
- [ ] Malformed：运行停止——ADR 不被视为 Accepted，不写入清单
- [ ] Malformed：retrofit 命令为 `/architecture-decision retrofit [file]`——即该技能解析的形式
- [ ] 两个 fixture 中，在用户继续之前都不预览规则摘要，也不显示"May I write"询问

---

## 协议合规

- [ ] 在读取 ADR 节之前统计 ADR 并解析 `## Status`
- [ ] 仅包含 Accepted ADR，每份被排除的 ADR 都在预览中注明名称和状态；none-Accepted 和 no-`## Status` 情况停止或询问，而非输出空清单
- [ ] 在门禁之前和"May I write"之前向用户显示规则摘要
- [ ] 写入 `docs/architecture/control-manifest.md` 之前询问"May I write the Control Manifest?"
- [ ] TD-MANIFEST 在 `full` 下运行，在 `lean`/`solo` 下跳过并注明
- [ ] 以下一步交接结尾：先 `/create-epics layer: foundation` 再 `/create-stories [epic-slug]`

---

## 覆盖说明

- 生成清单的确切节结构（约束表、模式列表）由技能正文定义，测试断言中不再重复列举。
- 用户继续使用 Proposed ADR 之后发生什么（用例 6）未做 fixture 测试。
- TD-MANIFEST 的 APPROVE、CONCERNS 和 NOT ASSESSED 路径，以及 `lean` 跳过说明，未单独做 fixture 测试。
- `deprecated-apis.md` 条目的相关性过滤（2D vs 3D 范围）未覆盖。