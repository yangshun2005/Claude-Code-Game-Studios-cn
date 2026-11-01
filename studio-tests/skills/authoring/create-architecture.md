# 技能测试规范：/create-architecture

## 技能摘要

`/create-architecture` 生成主架构文档 `docs/architecture/architecture.md`。Phase 0 加载上下文，并在缺少前置条件时提前停止：引擎参考文档（无引擎 → `/setup-engine`）、`design/gdd/systems-index.md`（在 `standard`/`full` 下缺失 → `/map-systems`）、`design/gdd/game-concept.md`（缺失 → `/brainstorm`；在 `minimal` 下由单页的 `design/game-brief.md` 代替）。它从每个 GDD 的需求承载章节中提取技术需求基线（`TR-[system]-[NNN]`），从 ADR 头部清点现有 ADR，并在撰写前展示引擎知识缺口清单，附带一次 `AskUserQuestion`。

Phase 1–4（分层图、模块归属、数据流、API 边界）各自展示并获批。Phase 5 审计现有 ADR，并将每条基线需求映射到 ADR 覆盖情况；未覆盖的需求成为必需的新 ADR。Phase 7 询问"所有章节已获批。我可以写入主架构文档吗？"，提供 `[A]/[B]/[C]` 选项。Phase 7b 收集 TD-ARCHITECTURE 和 LP-FEASIBILITY 评审，并排展示，在将签核记录到文档状态章节前先询问。Phase 8 更新 `production/session-state/active.md` 并打印固定的交接模板。

已存在的 `docs/architecture/architecture.md` 绝不会未经询问就被替换：Phase 0e 提供就地更新选定章节（提升文档版本）、重写全文或停止。聚焦领域参数（`layers`、`data-flow`、`api-boundaries`、`adr-audit`）即选定一个章节的更新；若尚无文档，则改为提供完整走查。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 Phase 标题
- [ ] 以各门禁自身的措辞包含签核结论关键词：`APPROVE`、`CONCERNS`、`FEASIBLE`，以及 `REVISED after …` 记录
- [ ] 包含写前询问语言：通过 `AskUserQuestion` 请求写入批准（"May I write the master architecture document?"、"May I update the Document Status section…"）
- [ ] 末尾有下一步交接（`/architecture-decision`、`/architecture-review`、`/create-control-manifest`）
- [ ] 为 `docs/architecture/architecture.md` 记录一致的写入模型——文件何时首次创建、每个获批章节何时写入
- [ ] 记录门禁行为：完整模式下 TD-ARCHITECTURE + LP-FEASIBILITY；在 lean/solo 下各自跳过并附具名说明
- [ ] 记录缺少引擎、系统索引和游戏概念时的停止分支
- [ ] 记录改造：检测已存在的 `docs/architecture/architecture.md`，并在任何重写前提供更新选定章节

---

## 总监门禁检查

评审模式来自解析后的 `review_mode`（`project.yaml` 中的 `modes.review_mode`）；`--review full|lean|solo` 对单次运行覆盖它。

在 `full` 模式下：`docs/architecture/architecture.md` 写入后，TD-ARCHITECTURE（`technical-director`）和 LP-FEASIBILITY（`lead-programmer`）对其进行评审。各自是生成的代理，读取自己的门禁文件（`.claude/docs/director-gates/td-architecture.md`、`lp-feasibility.md`）；其输入相互独立，因此同时生成，两个结论都在 Step 3 展示前收集完毕。父会话从不读取这些文件：它向 TD-ARCHITECTURE 传递文档路径、技术需求基线（TR-ID 及数量）、带状态的 ADR 列表和引擎知识缺口清单。

在 `lean` 模式下：两者都不是 PHASE-GATE，因此都跳过。输出注明"TD-ARCHITECTURE and LP-FEASIBILITY skipped — Lean mode."

在 `solo` 模式下：两者都跳过，注明"TD-ARCHITECTURE and LP-FEASIBILITY skipped — Solo mode."

---

## 夹具默认值

除非各用例自己的夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——新架构文档，完整模式，评审通过

**夹具：**
- `project.yaml` 具有 `engine.name: Godot`、`modes.review_mode: full`、`modes.workflow: full`
- `docs/engine-reference/godot/` 具有 `VERSION.md`、`breaking-changes.md`、`deprecated-apis.md`、`current-best-practices.md` 和 `modules/`
- `design/gdd/game-concept.md` 和 `design/gdd/systems-index.md` 存在且有真实内容
- `design/gdd/combat.md` 和 `design/gdd/inventory.md` 遵循 GDD 模板
- `docs/architecture/adr-0001-event-bus.md` 存在（`Status: Accepted`）
- 无 `docs/architecture/architecture.md`
- TD-ARCHITECTURE 返回 APPROVE；LP-FEASIBILITY 返回 FEASIBLE

**输入：** `/create-architecture`

**预期行为：**
1. Phase 0a 读取四个引擎文档，以及仅与系统索引中领域匹配的 `modules/` 文档
2. Phase 0b 对每个 GDD 的需求承载章节（Detailed Rules/Design、Formulas、Dependencies、Tuning Knobs、Acceptance Criteria）执行 Grep，并展示技术需求基线，ID 形如 `TR-combat-001 (provisional)`（尚无 TR 注册表）
3. Phase 0c 从 `## Status`/`## Summary` 和 `**Domain**` 的 Grep 构建 ADR 清单，而非完整读取
4. Phase 0d 展示引擎知识缺口清单，并询问 `[A] Proceed — flag HIGH RISK domains…` / `[B] Let me check the engine reference first…` / `[C] Show me which domains are HIGH RISK and why`
5. Phase 1–4：每个提案（分层图、归属图、每个数据流场景、API 边界）在下一个之前展示并获批，每次批准记录到 `production/session-state/active.md`——Phase 7 之前不向架构文档写入任何内容
6. Phase 7：一段摘要，然后"All sections approved. May I write the master architecture document?"，提供 `[A] Yes — write…` / `[B] Show me the full draft inline first…` / `[C] Not yet…`
7. Phase 7b：TD-ARCHITECTURE（接收文档路径、TR 基线、带状态的 ADR 列表和知识缺口清单）和 LP-FEASIBILITY 评审已写入的文档；两份评估并排展示；用户选择 `Accept — proceed to handoff`
8. 展示拟议的文档状态块，然后"May I update the Document Status section with the sign-off results?"
9. Phase 8 向 `production/session-state/active.md` 写入摘要，并打印交接，标题为"Architecture Complete"、"Run These ADRs Next"、"Gate-Check Readiness"；其 **Then:** 行先写 `/architecture-review`，评审通过后写 `/create-control-manifest`，就绪块写 `/gate-check pre-production`

**断言：**
- [ ] 基线使用 `TR-[system]-[NNN]` ID，且从扫描的 GDD 章节构建，而非整文件读取
- [ ] 知识缺口清单及其三选项 `AskUserQuestion` 在 Phase 1 之前
- [ ] 每个章节在请求批准前先展示
- [ ] 主文档仅在 Phase 7 的 `AskUserQuestion` 批准后写入
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 在文档写入后各自作为代理（`technical-director`、`lead-programmer`）生成，两个结论都在 Step 3 前收集
- [ ] TD-ARCHITECTURE 按名称接收其四项上下文；父会话不读取门禁文件
- [ ] 文档状态签核仅在"May I update"批准后写入
- [ ] 交接使用固定模板标题，无尾随评论
- [ ] 交接写 `/gate-check pre-production`——绝不用 `[stage]` 占位符，因为本技能运行期间它会解析为已通过的门禁
- [ ] `docs/architecture/architecture.md` 在 Phase 7 询问前不写入，哪怕部分；Phase 2–4 的批准进入会话状态

---

### 用例 2：评审有疑虑——LP-FEASIBILITY 返回 CONCERNS

**夹具：**
- 与用例 1 相同，直至 Phase 7b
- LP-FEASIBILITY 返回 CONCERNS："Save/load path has no owning module for serialisation"
- 场景 (a)：TD-ARCHITECTURE 返回 APPROVE
- 场景 (b)：TD-ARCHITECTURE 返回 NOT ASSESSED，指明一项无法读取的上下文

**输入：** `/create-architecture`

**预期行为：**
1. Step 3 并排展示技术总监评估和首席程序员结论，包括具体疑虑
2. `AskUserQuestion`：`Accept — proceed to handoff` / `Revise flagged items first` / `Discuss specific concerns`
3. 若 Accept：拟议的文档状态为 `LP-FEASIBILITY: [date] — CONCERNS (accepted)`，仅在"May I update the Document Status section…"回答是后应用
4. 若 Revise：被标记章节重新起草并按 Phase 1–4 方式获批，修订后的文档通过 Phase 7 的询问一次性写入，状态随后为 `LP-FEASIBILITY: [date] — REVISED after CONCERNS`
5. 技能不为返回 CONCERNS 的评审记录 `FEASIBLE`
6. (b) Step 4 提议 `TD-ARCHITECTURE: [date] — NOT ASSESSED — [missing input]`，绝不写 `APPROVE`，交接首行写 `NOT ASSESSED`

**断言：**
- [ ] 疑虑文本与 TD 评估一同展示给用户
- [ ] 向用户提供 Accept / Revise flagged items first / Discuss specific concerns
- [ ] 接受时记录 `CONCERNS (accepted)`，而非 `FEASIBLE`
- [ ] 修订时记录 `REVISED after CONCERNS`，修订后的文档在其章节获批后一次性写入
- [ ] 未经"May I update"批准不写入任何文档状态变更
- [ ] (b) NOT ASSESSED 评审连同其缺失输入记录，绝不记为批准

---

### 用例 3：Lean 和 Solo 模式——两项评审均跳过，仅凭用户批准写入

**夹具：**
- 与用例 1 相同项目，但场景 (a)：`project.yaml` 具有 `modes.review_mode: lean`
- 场景 (b)：相同项目，以 `--review solo` 调用

**输入：** (a) `/create-architecture` (b) `/create-architecture --review solo`

**预期行为：**
1. Phase 0–7 如用例 1 运行；文档在 Phase 7 批准后写入
2. 在 Phase 7b 两项评审都不运行；输出注明 (a) "TD-ARCHITECTURE and LP-FEASIBILITY skipped — Lean mode." (b) "TD-ARCHITECTURE and LP-FEASIBILITY skipped — Solo mode."
3. Step 4 展示文档状态块，两行均为 `skipped — [mode] mode`，并在应用前询问；技能随后进入 Phase 8 交接，不因跳过的评审而阻塞

**断言：**
- [ ] 不生成 `technical-director` 或 `lead-programmer`，lean 或 solo 模式下不应用任何 TD-ARCHITECTURE 评审
- [ ] 跳过说明指明两个门禁和模式
- [ ] (b) `--review solo` 参数对本次运行覆盖解析后的 `review_mode`
- [ ] 文档仅凭用户批准写入，完成不被阻塞
- [ ] 文档状态记录两项评审在该模式下跳过（在其询问后），使文档说明为何无签核
- [ ] Phase 8 交接仍打印

---

### 用例 3b：评审拒绝——不提供 Accept

**夹具：**
- 与用例 1 相同项目，`full` 模式；`technical-director` 返回 `TD-ARCHITECTURE: REJECT`（一个核心系统依赖表现层模块）；`lead-programmer` 返回 `LP-FEASIBILITY: FEASIBLE`

**输入：** `/create-architecture`

**预期行为：**
1. 两项评审在 Phase 7b 并行生成
2. Step 3 并排展示 `TD-ARCHITECTURE: REJECT` 和 `LP-FEASIBILITY: FEASIBLE`
3. `AskUserQuestion` 提供 `Revise flagged items first` 和 `Discuss specific concerns`——**不**提供 `Accept`
4. 选择 `Revise flagged items first` 时，被标记章节重新起草并按 Phase 1–4 方式获批，修订后的文档通过 Phase 7 的询问一次性写入
5. 不记录任何批准；Step 4 记录 `REVISED after REJECT`，交接首行携带相同值

**断言：**
- [ ] `technical-director` 和 `lead-programmer` 调用在等待任一结果前发出
- [ ] REJECT 存在时不提供 `Accept — proceed to handoff` 选项
- [ ] 修订后的文档在重新起草的章节获批后一次性写入——而非逐章节写入
- [ ] 本次运行的文档状态绝不将 TD-ARCHITECTURE 显示为 APPROVE；交接行写 `REVISED after REJECT`

---

### 用例 4：边界情况——系统索引缺失或为空

**夹具：**
- 引擎已配置；`design/gdd/game-concept.md` 存在
- 场景 (a)：`modes.workflow: standard`，无 `design/gdd/systems-index.md`
- 场景 (b)：`modes.workflow: standard`，`design/gdd/systems-index.md` 存在但仅含模板占位符
- 场景 (c)：`modes.workflow: minimal`，无系统索引，无游戏概念，`design/game-brief.md` 存在

**输入：** `/create-architecture`

**预期行为：**
1. 技能在读取任一设计文件前先检查两者是否存在
2. (a) 停止并提示："No systems index found. Run `/map-systems` first…"——不构建基线，不写入文件
3. (b) 将仅含占位符的索引视为缺失，同样停止
4. (c) 说明 `minimal` 下不需要系统索引，读取 `design/game-brief.md` 代替概念，并继续

**断言：**
- [ ] (a) 技能以 `/map-systems` 消息停止，不写入任何内容
- [ ] (b) 存在但仅含占位符的系统索引被视为缺失
- [ ] (c) 在 `minimal` 下技能说明索引非必需，并从游戏简报继续
- [ ] 在 `standard`（a、b）下，设计文件缺失时无运行越过 Phase 0b；在 `minimal`（c）下简报代替两者，仅当它也缺失时才停止

---

### 用例 5：ADR 审计——拟议 ADR 与未覆盖需求

**夹具：**
- 与用例 1 相同，另加 `docs/architecture/adr-0002-save-format.md`，`Status: Proposed`
- 另有 `design/gdd/crafting.md`，写于 GDD 模板之前，不含任何被扫描的章节标题
- 基线需求 `TR-combat-002`（连招状态机）未被任何 ADR 的 GDD Requirements Addressed 章节或决策文本覆盖

**输入：** `/create-architecture`

**预期行为：**
1. Phase 0b 的章节 Grep 在 `crafting.md` 中无匹配，因此技能完整读取它，说明这一点，其需求以 `TR-crafting-NNN` 进入基线——零匹配的 GDD 绝不视为无需求
2. Phase 0c 列出 ADR-0001（Accepted）和 ADR-0002（Proposed）及其领域
3. Phase 5 展示 ADR 质量表和可追溯性表，将 `TR-combat-002` 标记为 `❌ GAP`，附覆盖/缺口计数
4. 缺口列在必需的新 ADR 下，按层分组，Foundation 优先，形如 `/architecture-decision [title] → covers: TR-combat-002`
5. Phase 8 的"Gate-Check Readiness"将 ADR-0002 列在"Accept ADRs"下，未写入的 ADR 列在"Write ADRs"下
6. "Run These ADRs Next"按优先级列出 Phase 6 中至多三个 ADR

**断言：**
- [ ] `crafting.md` 未匹配任何被扫描章节，被完整读取并报告，且向基线贡献 TR-ID
- [ ] ADR 状态来自头部扫描，ADR-0002 报告为 Proposed
- [ ] `TR-combat-002` 标记为 GAP，并成为指明其所覆盖 TR-ID 的必需新 ADR
- [ ] 必需的新 ADR 按层分组，Foundation 优先
- [ ] "Gate-Check Readiness