# 技能测试规范：/architecture-review

## 技能摘要

`/architecture-review` 是一个 Opus 级技能，用于检查全部架构决策是否覆盖了游戏的设计。它从每个 GDD 中提取技术需求（TR-ID），在追溯矩阵中将每项需求映射到覆盖它的 ADR，检测 ADR 之间的冲突，按依赖关系对 ADR 排序（通过 `.claude/scripts/adr-dep-graph.sh`），对照 `docs/engine-reference/` 审计引擎兼容性，咨询主引擎专家，并检查 `docs/architecture/architecture.md` 对系统索引的覆盖情况。它产出 PASS / CONCERNS / NOT ASSESSED / FAIL 判定。

它接受一个聚焦参数（默认 `full`，或 `coverage`、`consistency`、`engine`、`single-gdd [path]`、`rtm`）。报告生成后，它通过 `AskUserQuestion` 询问是否写入审查报告（`docs/architecture/architecture-review-[date].md`）、追溯索引和 `docs/architecture/tr-registry.yaml`。它是技术设置与预制作之间的质量关卡。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：PASS、CONCERNS、NOT ASSESSED、FAIL
- [ ] `allowed-tools` 包含 Write，因此存在写入审批措辞（"May I update the systems index?"、"May I update `docs/architecture/tr-registry.yaml`…"）
- [ ] 末尾有下一步交接（阶段 9）
- [ ] 记录了参数模式：`full`、`coverage`、`consistency`、`engine`、`single-gdd [path]`、`rtm`

---

## 总监关卡检查

无。此技能不解析 `review_mode`（其配置块只请求 `automation` 和 `workflow`），也不生成总监关卡。它唯一生成的 agent 是阶段 5 的主引擎专家。架构签核关卡（TD-ARCHITECTURE、LP-FEASIBILITY）属于 `/create-architecture`，不属于此技能。

---

## Fixture 默认值

除非其自身的 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——每项需求都由一个 Accepted ADR 覆盖

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`、`engine.name: Godot`、`engine.version: 4.6`
- `design/gdd/systems-index.md` 加两个系统 GDD（`combat.md`、`inventory.md`），每个都有 `## Summary`，并在其 Detailed Rules 中包含技术需求
- `docs/architecture/adr-0001-*.md` … `adr-0003-*.md`，每个都有 `## Status: Accepted`、`## GDD Requirements Addressed`、`## Engine Compatibility` 和 `## ADR Dependencies`，其 `**Depends On**` 行已填写——`adr-0001` 为 `None`，`adr-0002` 和 `adr-0003` 为 `ADR-0001`——因此 `adr-dep-graph.sh` 能找到真实边，且 `NO_DEPS_SECTION` 下不列出任何 ADR；它们合起来覆盖每项需求；没有两个相互矛盾
- `docs/architecture/architecture.md` 存在，并列出索引中的每个系统
- 没有先前的 `docs/architecture/architecture-review-*.md` 报告
- 所有预关卡项都存在：`tests/unit/`、`tests/integration/`（Godot 测试根目录）、`.github/workflows/tests.yml`、`design/accessibility-requirements.md`、`design/ux/interaction-patterns.md`

**输入：** `/architecture-review`

**预期行为：**
1. 新鲜度检查打印 `RECEIPT: NONE` → 完整审查
2. 技能统计分母（N_gdd、N_adr），用定向 Grep 而非整文件加载所需章节，并报告 "Loaded [N] GDDs, [M] ADRs, engine: Godot 4.6"
3. 技能提取 TR-ID（复用任何现有的 `tr-registry.yaml` ID）并构建矩阵
4. 技能运行 `bash .claude/scripts/adr-dep-graph.sh`，并从其 `ORDER:` 行打印推荐的实现顺序
5. 技能运行引擎审计并咨询 `godot-specialist`
6. 报告以 `### Verdict: PASS` 结尾
7. 阶段 8 询问要写入什么；阶段 9 提供 `/gate-check pre-production`

**断言：**
- [ ] 输出报告 "Loaded [N] GDDs, [M] ADRs, engine: [name + version]"
- [ ] 追溯矩阵列出每个 TR-ID 及其 GDD、系统、需求、ADR 覆盖和状态；每一行都是 ✅
- [ ] ADR 依赖顺序来自 `adr-dep-graph.sh` 的 `ORDER:` 行，而非手工追踪，并将 `adr-0001` 排在首位；`NO_DEPS_SECTION` 和 `UNFILLED_DEPS` 为空，因此无环结果是干净的图，而非结构性缺口
- [ ] 判定为 PASS 仅因为所有需求都由 **Accepted** ADR 覆盖、不存在冲突、且引擎参考一致
- [ ] 报告头部有 `Mode: full` 和 `Workflow: full` 行
- [ ] 报告包含每个审查输入的 `review-receipts.sh hash` 行（ADR、GDD、TR 注册表、引擎参考、`project.yaml`、技能自身文件），在 TR 注册表写入后获取
- [ ] 在阶段 8 的 `AskUserQuestion` 之前不写入任何文件（`[A]` 全部三个文件 / `[B]` 仅报告写入 `docs/architecture/architecture-review-[date].md` / `[C]` 暂不写入）
- [ ] 结尾组件提供 `/gate-check pre-production`，因为每个预关卡清单项都是 ✅

---

### 用例 2：失败路径——基础层缺口加跨 ADR 所有权冲突

**Fixture：**
- 同用例 1，但：
  - `save-system.md`（基础层系统）要求 "Player progress persists between sessions"，且没有 ADR 涉及它
  - `adr-0002` 和 `adr-0004` 都是 Accepted，且都声称对玩家生命值拥有权威

**输入：** `/architecture-review`

**预期行为：**
1. 持久化需求显示为 `❌ GAP` 行
2. 缺口出现在 "Coverage Gaps" 下，附建议的 `/architecture-decision [title]`、Domain 和 Engine Risk
3. 打印冲突块 `## Conflict: [ADR-0002] vs [ADR-0004]`
4. 判定为 FAIL；Blocking Issues 至少列出基础层缺口
5. 两个 ADR 都不被编辑；由用户决定解决方案

**断言：**
- [ ] 判定为 FAIL（基础层需求未被覆盖）
- [ ] 缺口行列出 TR-ID、GDD 和需求，并附建议的 `/architecture-decision` 命令
- [ ] 冲突块列出两个 ADR 编号、Type（State / Data ownership）、每个 ADR 声称的内容、Impact，以及至少两个解决方案选项
- [ ] 技能不自动解决冲突，也不编辑任一 ADR
- [ ] 如果 `docs/consistency-failures.md` 存在，阶段 8 的 `[A]` 选项会列出它（"…and append 1 conflict entry to `docs/consistency-failures.md`"），且仅当用户选择 `[A]` 时才追加条目；`[B]`（仅报告）只写报告，不写其他；如果文件不存在，则不创建
- [ ] 结尾组件不提供 `/gate-check`；它提供通过 `/architecture-decision [system]` 编写缺失的 ADR

---

### 用例 3：部分路径——覆盖依赖 Proposed ADR

**Fixture：**
- 同用例 1，但 `adr-0003`（唯一覆盖 `TR-inventory-001` 的 ADR）具有 `## Status: Proposed`
- `adr-0004`（Accepted）在其 `## ADR Dependencies` 表中具有 `**Depends On**: ADR-0003`

**输入：** `/architecture-review`

**预期行为：**
1. `TR-inventory-001` 标记为 `🟡 Covered (Proposed)`，而非 ✅
2. 依赖分析标记 "ADR-0004 depends on ADR-0003 — but ADR-0003 is still Proposed"
3. 判定上限为 CONCERNS，并指出出路：`/architecture-decision accept ADR-0003`

**断言：**
- [ ] 技能在标记覆盖前读取每个 ADR 的 `## Status`；Proposed ADR 绝不产出 ✅
- [ ] 当覆盖依赖 Proposed ADR 时，判定为 CONCERNS（而非 PASS）
- [ ] 输出指出 `/architecture-decision accept ADR-0003` 作为清除途径
- [ ] 对 Proposed ADR 的未解决依赖被标记

---

### 用例 4：边界情况——缺少架构文档；ADR 格式错误

**用例 4a——无主架构文档：**
- 同用例 1，但 `docs/architecture/architecture.md` 不存在

**断言（4a）：**
- [ ] 报告将 `Architecture document coverage: NOT ASSESSED — no docs/architecture/architecture.md` 列为具名行项，而非显示无阶段 6 发现
- [ ] 总体判定为 NOT ASSESSED，而非 PASS：阶段 6 是 `full` 范围的一部分且未运行，即使每项需求都被覆盖（其他地方的 CONCERNS 或 FAIL 发现仍会优先于它）

**用例 4b——ADR 存在但无一具有可扫描章节：**
- 同用例 1（`modes.workflow: full`——在默认 `minimal` 层级下，没有 `modes` 块的 `project.yaml`，技能不适用），但三个 `docs/architecture/adr-*.md` 文件都不包含阶段 1b 扫描所寻找的六个标题：`## Status`、`## Decision`、`## GDD Requirements Addressed`、`## Engine Compatibility`、`## ADR Dependencies`、`## Performance Implications`

**断言（4b）：**
- [ ] 技能报告 "[N] ADRs found, none carries a scannable section — run `/architecture-decision retrofit [file]` on each"（子命令在前——若路径在前，`/architecture-decision` 会开始编写新 ADR）
- [ ] 技能不报告零覆盖或设计失败——这是格式失败
- [ ] ADR 没有 `## Status` 的行标记为 `❓ Not assessed`，判定不是 PASS——也不是基于覆盖理由的 FAIL，因为覆盖是未知而非缺失

---

### 用例 5：引擎专家咨询——已配置 vs 未配置引擎

**用例 5a——引擎已配置：** 用例 1 fixture（`engine.name: Godot`）

**预期行为：**
1. 阶段 5 引擎审计后，技能通过 `Agent` 生成 `godot-specialist`，传入引擎相关 ADR、引擎参考文档和审计发现
2. 专家的发现出现在 `### Engine Specialist Findings` 下，并计入判定

**断言（5a）：**
- [ ] 专家由 `engine.name` 解析（Godot → `godot-specialist`）
- [ ] 在引擎审计后生成，其提示中包含审计发现
- [ ] 输出包含 `### Engine Specialist Findings` 章节
- [ ] 不生成或提及任何总监关卡（TD-ARCHITECTURE、LP-FEASIBILITY 或任何其他关卡 ID）

**用例 5b——未配置引擎：** 同用例 1，但 `project.yaml` 中 `engine.name` 未设置，且 `technical-preferences.md` 仍为 `[TO BE CONFIGURED]`

**断言（5b）：**
- [ ] 不生成引擎专家
- [ ] 输出记录 `Engine validation: NOT ASSESSED — no engine configured`，而非省略该步骤
- [ ] 判定为 NOT ASSESSED，而非 PASS——引擎审计是 `full` 范围的一部分，且没有固定引擎可供对照检查，即使每项需求都被覆盖

---

## 协议合规

- [ ] 在任何写入审批前显示完整追溯矩阵（阶段 1a 的新鲜度问题和阶段 2 的模糊需求问题可能更早出现）
- [ ] 每次写入审批都使用带标签选项的 `AskUserQuestion`，而非纯文本 "May I?"
- [ ] 在请求写入前显示要写入的内容（报告、systems-index 行）
- [ ] 绝不重新编号或删除现有 `tr-registry.yaml` 条目；将某条目标记为已弃用需与用户确认
- [ ] 判定恰好为以下之一：PASS、CONCERNS、NOT ASSESSED、FAIL
- [ ] 以阶段 9 通过 `AskUserQuestion` 交接结束；任何预关卡项为 ❌ 时不提供 `/gate-check`

---

## 覆盖说明

- `coverage`、`consistency`、`engine`、`single-gdd` 和 `rtm` 模式未单独进行 fixture 测试；用例 1–5 演练默认 `full` 模式。
- 阶段 5b（将 GDD 修订标记作为 `Needs Revision` 写入 `systems-index.md`）未测试；它需要一个与 GDD 矛盾的 HIGH RISK 引擎发现。
- 新鲜度回执路径（全部 `UNCHANGED` 且报告覆盖或不覆盖本次运行、部分 `CHANGED`/`NEW`、`GONE`、`UNRESOLVED`）未进行 fixture 测试。
- `standard` 层级的缩减范围（仅架构文档 + 关键 ADR）未测试。
- 默认 `minimal` 层级（没有 `modes` 块的 `project.yaml`），技能的范围表称审查不适用，未进行 fixture 测试——每个用例都固定 `modes.workflow: full`。`workflow_overrides.system_overrides` 也未测试：此技能不请求该键，因此按系统的层级对它没有影响。