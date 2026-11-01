# 技能测试规范：/create-epics

## 技能摘要

`/create-epics` 读取范围内的 GDD、架构文档、已接受的 ADR 和 TR 注册表，将每个系统转换为一个史诗，并映射到架构模块。系统按层级顺序处理（基础层 → 核心层 → 功能层 → 表现层），同一层级内按 `systems-index.md` 顺序处理。每个史诗都会展示（层级、GDD、架构模块、管辖 ADR、引擎风险、ADR 覆盖率、未追溯需求），并在写入任何内容前询问用户"Shall I create Epic: [name]?"。

史诗写入 `production/epics/[epic-slug]/EPIC.md`，每个史诗写入前都会单独询问"May I write `production/epics/[epic-slug]/EPIC.md` and add its row to `production/epics/index.md` (creating it if absent)?"，该询问会指明随后创建或更新的索引。已存在的 EPIC.md 绝不覆盖：用户可选择原地更新（保留其 Stories 表）或跳过。在 `full` 评审模式下，PR-EPIC 门禁（producer）在当前层级所有史诗定义完成后、写入任何文件前运行；在 `lean` 和 `solo` 模式下跳过并注明。评审模式来自 `resolve_config`（`review_mode`），可通过 `--review` 按次覆盖。结论：COMPLETE（已写入 N 个史诗）或 BLOCKED（用户拒绝所有史诗、在门禁处停止，或未找到符合条件的系统）。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含结论关键词：COMPLETE、BLOCKED
- [ ] 包含"May I write"协作协议语言（逐史诗审批）
- [ ] 结尾有下一步交接（`/create-stories [epic-slug]`）
- [ ] 记录了 PR-EPIC 门禁行为：full 模式运行；lean/solo 模式跳过

---

## 导演门禁检查

在 `full` 模式下：当前层级所有史诗定义完成后、写入任何史诗文件前，运行 PR-EPIC（`producer`，`.claude/docs/director-gates/pr-epic.md`）。结论：REALISTIC → 进入写入询问；CONCERNS → `AskUserQuestion` 提供 `[A] Proceed as planned` / `[B] Revise epic boundaries` / `[C] Stop`；UNREALISTIC → 提议拆分或合并史诗，修订后重新展示修订的史诗，并在写入前重新运行门禁。CONCERNS `[B]` 同样在门禁重新运行前重新展示修订的史诗。NOT ASSESSED [缺少输入] → 绝不解读为 REALISTIC：补充输入并重新运行门禁，或继续运行并在输出和 Verdict 行中注明 `PR-EPIC: NOT ASSESSED — [input]`。

在 `lean` 模式下：跳过 PR-EPIC。输出注明："PR-EPIC skipped — Lean mode."

在 `solo` 模式下：跳过 PR-EPIC。输出注明："PR-EPIC skipped — Solo mode."

---

## Fixture 默认值

除非其自身 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture defaults。

---

## 测试用例

### 用例 1：正常路径——两个已批准的 GDD 创建两个史诗

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`modes.workflow: full`、`modes.review_mode: lean`
- `design/gdd/systems-index.md` 列出 2 个基础层系统
- 两个系统在 `design/gdd/` 中都有已批准的 GDD，每个都有 `## Summary`
- `docs/architecture/architecture.md` 为每个系统都有一个模块
- 已接受的 ADR 覆盖 `docs/architecture/tr-registry.yaml` 中两个系统的所有 TR-ID；第一个系统有两个管辖 ADR，其 Engine Compatibility 分别评为 Knowledge Risk LOW 和 HIGH
- `docs/architecture/control-manifest.md` 存在
- 无 `production/epics/` 目录

**输入：** `/create-epics layer: foundation`

**预期行为：**
1. Glob `design/gdd/*.md`（排除 systems-index 和其他非系统文档）以确定 N，然后 Grep `^## Summary` 以确定基础层系统范围
2. 读取 2 个范围内的 GDD、`architecture.md`、范围内 ADR 的 Decision / GDD Requirements Addressed / Engine Compatibility 部分、TR 注册表和清单头部；报告"Loaded 2 GDDs, [M] ADRs, engine: …"
3. 对每个系统：展示史诗块并询问"Shall I create Epic: [name]?"
4. 注明"PR-EPIC skipped — Lean mode."
5. 对每个史诗：询问"May I write `production/epics/[epic-slug]/EPIC.md` and add its row to `production/epics/index.md` (creating it if absent)?"，批准后写入两者
6. 创建或更新 `production/epics/index.md`
7. 结论：COMPLETE — 已写入 2 个史诗；下一步 `/create-stories [epic-slug]`

**断言：**
- [ ] 在任何写入询问前展示每个史诗块并询问"Shall I create Epic: [name]?"
- [ ] 逐史诗询问"May I write"（而非两个一起问一次）
- [ ] 文件落在 `production/epics/[epic-slug]/EPIC.md`
- [ ] 每个 EPIC.md 包含 Layer、GDD path、Architecture Module、Governing ADRs 表、GDD Requirements 表、Definition of Done
- [ ] 第一个史诗的 Engine Risk 为 HIGH——取其管辖 ADR 中的最高值，而非第一个或平均值
- [ ] Definition of Done 要求 Logic 和 Integration 故事在 `tests/`（Godot 测试根目录）下通过测试，Visual/Feel 和 UI 故事在 `production/qa/evidence/` 中保留截图——UI 涉及的每个屏幕，外加 Visual/Feel 的负责人签核
- [ ] 出现"PR-EPIC skipped — Lean mode."；未启动 producer
- [ ] 写入后创建或更新 `production/epics/index.md`，且仅在指明它的询问之后
- [ ] 结论为 COMPLETE 并指明 `/create-stories [epic-slug]`

---

### 用例 2：失败路径——未找到系统 GDD

**Fixture：**
- `project.yaml`：`modes.workflow: full`
- `design/gdd/` 仅包含 `game-concept.md` 和 `systems-index.md`（无系统 GDD）

**输入：** `/create-epics all`

**预期行为：**
1. Glob `design/gdd/*.md` 排除非系统文档——N = 0
2. 报告"No system GDDs found in `design/gdd/` — run `/design-system` first"并停止
3. 不展示史诗、不运行门禁、不写入文件

**断言：**
- [ ] `game-concept.md` 和 `systems-index.md` 不计为系统 GDD
- [ ] 打印"No system GDDs found … run `/design-system` first"消息
- [ ] 不显示"Shall I create Epic"或"May I write"询问，不写入 EPIC.md 或 index.md
- [ ] 结论为 BLOCKED（未找到符合条件的系统），而非 COMPLETE

---

### 用例 3：导演门禁——Full 模式在写入前启动 PR-EPIC

**Fixture：**
- 2 个已批准的 GDD、架构、ADR、TR 注册表和清单同用例 1
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: full`——固定工作流，因为未设置时会解析为 `minimal`，此时技能读取简报而非 GDD，门禁不会介入
- PR-EPIC 返回 REALISTIC

**输入：** `/create-epics layer: foundation`

**Full 模式预期行为：**
1. 两个史诗都通过"Shall I create Epic: [name]?"展示并接受
2. 通过 `Agent` 启动 `producer`，门禁 PR-EPIC，接收史诗结构摘要、层级、里程碑时间线和团队容量
3. REALISTIC 时：逐史诗"May I write"询问继续
4. 批准后写入史诗文件

**断言（full 模式）：**
- [ ] 两个史诗定义后 PR-EPIC 启动一次
- [ ] PR-EPIC 在任何"May I write"询问前运行
- [ ] PR-EPIC 解决前不写入任何 EPIC.md

**Fixture（按次覆盖）：**
- 同一项目，`modes.workflow: full`、`modes.review_mode: full`
- 输入：`/create-epics layer: foundation --review lean`

**覆盖预期行为：**
1. `--review lean` 覆盖解析出的 `full`
2. 跳过 PR-EPIC——在输出中注明
3. "May I write"询问直接继续

**断言（覆盖）：**
- [ ] 输出中出现"PR-EPIC skipped — Lean mode."
- [ ] 未启动 producer agent，写入询问直接跟在史诗定义之后

---

### 用例 4：边界情况——full 工作流下的未追溯需求

**Fixture：**
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: solo`
- 一个已批准的基础层 GDD，其系统在 `tr-registry.yaml` 中有 5 个 TR-ID；已接受的 ADR 覆盖其中 3 个（2 个未追溯）

**输入：** `/create-epics [system-name]`

**预期行为：**
1. 史诗块显示"GDD Requirements Covered by ADRs: 3 / 5"，并在"Untraced Requirements"下列出 2 个未追溯的 TR-ID
2. 警告："⚠️ 2 requirements in [system] have no ADR. The epic can be created, but `/create-stories` will write their stories with no governing ADR (`ADR: N/A`) and `Status: Ready` — nothing downstream blocks them, so they would be implemented without architectural guidance. Run `/architecture-decision` first if they need a decision, or proceed and accept the gap."
3. 询问"Shall I create Epic: [name]?"，提供 `[A] Yes, create it` / `[B] Skip this epic` / `[C] Pause — I need to write ADRs first`
4. 选 [A]：注明"PR-EPIC skipped — Solo mode."，询问"May I write"，EPIC.md 的 GDD Requirements 表将 2 行标记为 `❌ No ADR`

**断言：**
- [ ] 在创建询问前显示覆盖计数和 2 个未追溯的 TR-ID
- [ ] 未追溯需求警告指明 `/architecture-decision`，并说明下游实际发生的情况：故事以 `Ready` 写入且无管辖 ADR——不声称它们会被 Blocked
- [ ] 提供选项 `[C] Pause — I need to write ADRs first`
- [ ] 写入的 EPIC.md 将未追溯行标记为 `❌ No ADR`（不静默丢弃）

---

### 用例 5：导演门禁——PR-EPIC 返回 CONCERNS

**Fixture：**
- 2 个已批准的 GDD、架构、ADR、TR 注册表和清单同用例 1
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: full`
- PR-EPIC 返回 CONCERNS（一个史诗的范围过大）

**输入：** `/create-epics layer: foundation`

**预期行为：**
1. 启动 PR-EPIC 并返回 CONCERNS 及具体反馈
2. 技能在任何写入询问前展示 producer 的评估
3. `AskUserQuestion`："Producer raised concerns about the epic structure. How do you want to proceed?"，提供 `[A] Proceed as planned — I accept the producer's concerns` / `[B] Revise epic boundaries — split or merge as recommended` / `[C] Stop — I want to reconsider the scope`
4. 选 [B]：从步骤 4 修订史诗定义并重新运行 PR-EPIC
5. 选 [C]：停止，结论：BLOCKED — 用户想重新考虑史诗范围

**断言：**
- [ ] 在任何"May I write"询问前向用户展示 producer 的 CONCERNS
- [ ] 门禁未解决时技能不写入史诗
- [ ] 提供全部三个选项（[A] 继续 / [B] 修订 / [C] 停止）
- [ ] 选 [B] 重新展示修订的史诗并在任何写入询问前重新运行 PR-EPIC
- [ ] 选 [C] 以结论：BLOCKED 结束且不写入文件

---

### 用例 6：边界情况——某系统已存在 EPIC.md

**Fixture：**
- 同用例 1（`modes.workflow: full`、`modes.review_mode: lean`）
- `production/epics/[first-slug]/EPIC.md` 已存在，头部有 `**Stories**: 3 stories` 和一个 3 行的 `## Stories` 表；`production/epics/index.md` 有其行，为 `3 stories`
- 第二个系统无 EPIC.md

**输入：** `/create-epics layer: foundation`

**预期行为：**
1. 两个史诗都通过"Shall I create Epic: [name]?"展示并接受
2. 对第一个系统，技能不覆盖：询问"An EPIC.md already exists for [name]. Update it in place, or skip it?"，提供 `[A] Update in place — keep its Stories table` / `[B] Skip this epic`；用户选 [A]
3. 更新重写除 `Stories` 外的头部字段、Overview、Governing ADRs、GDD Requirements 和 Definition of Done，保留 `**Stories**` 行和 `## Stories` 表不变
4. 对第二个系统，正常的"May I write `…/EPIC.md` and add its row to `production/epics/index.md`…?"询问随后进行
5. `production/epics/index.md` 保留第一个史诗的一行，仍为 `3 stories`，并为第二个新增一行

**断言：**
- [ ] 在任何写入前检测到已存在文件，并提供更新/跳过——不覆盖
- [ ] 更新后，`**Stories**` 头部行和全部 3 行 `## Stories` 不变
- [ ] 索引中第一个史诗无重复行，且不重置为 `Not yet created`
- [ ] 第二个系统的 EPIC.md 仅在其自身的"May I write"询问后写入

---

### 用例 7：Standard 层级——GDD 无 `## Summary`，一个未追溯的非关键需求

**Fixture：**
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: solo`
- `design/gdd/systems-index.md` 列出一个基础层和一个核心层系统；两个 GDD 都早于 `## Summary`（Summary 扫描均不匹配）
- `docs/architecture/architecture.md` 为每个系统都有一个模块，`docs/architecture/tr-registry.yaml` 包含两个系统的 TR-ID
- 基础层系统的 TR-ID 由已接受的基础层 ADR 覆盖；核心层系统的一个 TR-ID 无 ADR
- `docs/architecture/control-manifest.md` 不存在

**输入：** `/create-epics all`

**预期行为：**
1. Glob `design/gdd/*.md` 得 N = 2；`^## Summary` 扫描返回 0 个匹配
2. 注明一次："No `## Summary` sections found across 2 GDDs — scoping from `systems-index.md` instead of Summary."，并完整读取两个范围内的 GDD
3. 无控制清单继续（存在则读取，`standard` 下非必需）
4. 核心层史诗在"Untraced Requirements"下列出其未追溯的 TR-ID，仅作信息——无 ⚠️ 未追溯需求警告，无阻塞
5. 注明"PR-EPIC skipped — Solo mode."；每个史诗在其自身询问后写入

**断言：**
- [ ] 零匹配的 Summary 扫描如实报告，两个系统仍在范围内——不解读为"范围内无内容"
- [ ] 缺失控制清单在 `standard` 下不停止运行
- [ ] 列出未追溯的非关键需求，但不发出 ⚠️ 警告
- [ ] 两个史诗都写入，各自在其自身的"May I write"询问后

---

## 协议合规

- [ ] 在任何"May I write"询问前向用户展示史诗定义
- [ ] 逐史诗询问"May I write"，而非整批一次
- [ ] 已存在的 EPIC.md 按用户选择原地更新或跳过——绝不覆盖
- [ ] PR-EPIC 门禁（若启用）在写入询问前运行——而非之后
- [ ] 跳过的门禁在输出中按名称和模式注明
- [ ] EPIC.md 内容仅来自 GDD、ADR 和架构文档——不虚构
- [ ] 绝不创建故事；每个创建的史诗以下一步交接 `/create-stories [epic-slug]` 结束

---

## 覆盖说明

- 功能层和表现层遵循与基础层和核心层相同的逐史诗模式——层级特定顺序未独立测试。
- `minimal` 层级（可选；从 `design/game-brief.md` 分解，无则停止）未提供 fixture。
- `