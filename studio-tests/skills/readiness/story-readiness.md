# 技能测试规范：/story-readiness

## 技能摘要

`/story-readiness` 验证故事文件是否已就绪，可供开发者接手实现。它一次性加载参考上下文（系统索引、控制清单的 `Manifest Version:` 头部、TR 注册表，以及通过单次 Grep 获取的所有 ADR 状态），然后按六个检查清单组评估每个故事：设计完整性、架构完整性、范围清晰度、开放问题、资产引用和完成定义。为故事所属系统解析出的工作流层级决定哪些组会阻断（`full`：全部；`standard`：架构项为建议性，关键 ADR 除外；`minimal`：架构不适用）——但每个层级都有一条规则：故事引用的 ADR 若缺失、为 `Proposed`、`Deprecated` 或 `Superseded`，则阻断，因为 `/dev-story` 会因此停止。在 `minimal` 层级，`design/game-brief.md` 代替 GDD。它产出 READY / NEEDS WORK / BLOCKED / NOT ASSESSED 判定。它是只读的——从不写入或编辑文件。在 `full` 审查模式下，它随后运行 QL-STORY-READY 门禁（`qa-lead`）。

未设置 `qa.level` 也未设置 `modes.rigor` 的 fixture 会解析为 `qa.level: minimal`（`rigor: minimal` 默认值）。此时证据项对 Logic、Integration 和 Config/Data 故事自动通过——对 UI 或 Visual/Feel 则从不自动通过，其保留截图在每个 `qa.level` 都是必需的（Case 6）。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题或编号检查章节
- [ ] 包含判定关键词：READY、NEEDS WORK、BLOCKED、NOT ASSESSED
- [ ] 不要求"May I write"措辞——`allowed-tools` 不含 Write 或 Edit
- [ ] 具有下一步交接（判定后该做什么）

---

## Fixture 默认值

除非自身 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### Case 1：正常路径——完全就绪的故事

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full` 和 `modes.review_mode: solo`
- 故事文件存在于 `production/epics/core/story-light-pickup.md`，包含：
  - 引用具体需求的 `GDD: design/gdd/light-system.md` 引用
  - `TR-light-001`，在 `docs/architecture/tr-registry.yaml` 中存在且 `status: active`
  - `ADR: ADR-0003`；`docs/architecture/adr-0003-inventory.md` 的 `## Status` 为 Accepted
  - `Type: Logic`、含 3 个可测试项的 `## Acceptance Criteria` 章节，以及指明测试路径的 `## Test Evidence` 章节
  - 估算、Out of Scope 声明，以及 `Dependencies: None`
  - 引擎说明（或"N/A — 不涉及引擎 API"）、相关清单规则，以及性能说明
  - `Manifest Version:` 与 `docs/architecture/control-manifest.md` 中的一致
  - 无 TBD / UNRESOLVED 标记，无资产路径
- `production/sprints/` 中的冲刺计划列出了其他故事

**输入：** `/story-readiness production/epics/core/story-light-pickup.md`

**预期行为：**
1. 技能报告"Validating 1 story files."
2. 技能一次性加载上下文：从控制清单 grep `Manifest Version`，索引 `tr-registry.yaml`，并通过一次 `Grep pattern="^## Status" glob="docs/architecture/adr-*.md"` 扫描解析所有 ADR 状态
3. 技能评估全部六个检查清单组
4. 技能输出 READY，所有检查通过
5. 阶段 8 注明"QL-STORY-READY skipped — Solo mode."
6. 第 7 节展示冲刺中的其他就绪故事

**断言：**
- [ ] ADR 状态通过对 `docs/architecture/adr-*.md` 的一次 `^## Status` Grep 扫描解析，而非完整读取每个 ADR，且 ADR-0003 被发现为 Accepted
- [ ] 技能在 `tr-registry.yaml` 中查找 `TR-light-001` 并发现其为 active
- [ ] 当前清单版本取自清单头部的 Grep，而非完整读取
- [ ] "Passing Checks (N/[total])"章节列出全部六个组——设计完整性、架构完整性、范围清晰度、开放问题、资产引用（"no asset references"）、完成定义——并在每组下列出通过项
- [ ] 所有检查通过时判定为 READY
- [ ] 技能不写入任何文件
- [ ] 第 7 节列出最多 3 个其他就绪的冲刺故事——或声明"Next ready stories: no sprint file found" / "none ready in [sprint]"，而非省略该节

---

### Case 2：阻断路径——引用的 ADR 为 Proposed（未 Accepted）

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- 故事 `production/epics/core/story-light-system.md` 引用 `ADR-0005`
- `docs/architecture/adr-0005-light-system.md` 存在，`## Status` 为 Proposed
- 故事其他内容完整

**输入：** `/story-readiness production/epics/core/story-light-system.md`

**预期行为：**
1. 状态扫描发现 ADR-0005 为 `Proposed`
2. 技能标记为 BLOCKED，修复文本为"BLOCKED: ADR-0005 is Proposed — wait for acceptance before implementing."
3. 技能输出 BLOCKED，在 Blockers 下列出该 ADR

**断言：**
- [ ] 在 `full` 下 ADR 为 Proposed 时判定为 BLOCKED（而非 NEEDS WORK 或 READY）
- [ ] 输出明确将 ADR-0005 命名为阻断项
- [ ] 输出说明在实现前等待 ADR 被接受
- [ ] 无论其他检查是否通过，技能都不输出 READY
- [ ] 变体——同一故事在 `modes.workflow: standard` 下，ADR-0005 的 `## Engine Compatibility` 表中有 `| **Layer** | Feature |`（非关键）：故事引用了 Proposed ADR，因此判定仍为 BLOCKED——被引用的 Proposed ADR 在每个层级都阻断，因为 `/dev-story` 会因此停止
- [ ] 变体——同一非关键 ADR 在 `standard` 下，现为 `Accepted`，故事的 `Manifest Version:` 早于控制清单：过期版本作为建议性列在 Gaps 下，附其 `Fix:` 行，判定为 READY——建议性缺口不会降级判定

---

### Case 3：需要改进——缺少验收标准（每个层级都检查）

**Fixture：**
- `project.yaml` 设置 `modes.workflow: minimal`
- 故事 `production/epics/core/story-oxygen-drain.md` 有 `Type: Logic` 但无 `## Acceptance Criteria` 章节
- 故事不引用任何 ADR、TR-ID 和清单版本

**输入：** `/story-readiness production/epics/core/story-oxygen-drain.md`

**预期行为：**
1. 在 `minimal` 下，技能评估设计完整性和范围清晰度，并将架构完整性视为不适用
2. 技能发现无验收标准
3. 技能输出 NEEDS WORK，命名该缺口并附 `Fix:` 行，建议具体、可观察的标准

**断言：**
- [ ] 缺少 Acceptance Criteria 章节时判定为 NEEDS WORK（而非 BLOCKED 或 READY）
- [ ] 输出具体指出缺失的 Acceptance Criteria，附 `Fix:` 行提出可度量的标准
- [ ] 不标记 ADR、TR-ID 或清单缺口——架构完整性在 `minimal` 下不适用
- [ ] 故事不是 BLOCKED：无需外部操作（无缺失/DRAFT 依赖，无无主的 UNRESOLVED 问题）
- [ ] 变体——同一 `minimal` 故事，验收标准追溯到 `design/game-brief.md` 的 MVP 功能（无 `design/gdd/` 路径）："GDD requirement referenced"通过——在 `minimal` 下 brief 代替 GDD
- [ ] 变体——故事还命名了 `ADR-0005`，其 `## Status` 为 `Proposed`：在 `minimal` 下判定也为 BLOCKED——被引用 ADR 规则是唯一不适用的架构完整性项

---

### Case 4：边界情况——过期的清单版本

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- 故事头部有 `Manifest Version: 2026-01-15`
- `docs/architecture/control-manifest.md` 有 `Manifest Version: 2026-03-10`
- 故事其他一切通过

**输入：** `/story-readiness production/epics/core/story-mirror-rotation.md`

**预期行为：**
1. 技能从故事中提取 `2026-01-15`
2. 技能 grep 清单头部并提取 `2026-03-10`
3. 故事版本较旧 → "Manifest version is current"项失败
4. 判定为 NEEDS WORK，修复为审查变更的规则并更新故事的 `Manifest Version:`

**断言：**
- [ ] 技能通过 grep `docs/architecture/control-manifest.md` 中的 `Manifest Version` 获取当前版本
- [ ] 技能将故事内嵌版本与当前版本比较
- [ ] 在 `full` 下，过期清单版本导致 NEEDS WORK（非 BLOCKED，非 READY）
- [ ] 输出解释新清单规则可能适用，`Fix:` 说明审查变更的规则、必要时更新故事，然后将其 `Manifest Version:` 设为当前

---

### Case 5：主管门禁——QL-STORY-READY 在各审查模式下的行为

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- 故事文件存在且通过每个检查清单项
- 审查模式来自 `modes.review_mode`（旧版镜像 `production/review-mode.txt`）或 `--review`

**Case 5a——full 模式：** `/story-readiness production/epics/core/story-light-pickup.md --review full`

**预期行为：**
1. 技能先完成自身检查清单和判定
2. 技能通过 `Agent` 生成 `qa-lead`，门禁为 QL-STORY-READY，传递故事文件路径、故事类型、验收标准（逐字）、GDD 需求（TR-ID 和文本）、依赖状态和阶段 4 判定
3. ADEQUATE → 故事通过
4. GAPS → `AskUserQuestion`：`Update story with suggested gaps` / `Accept and proceed anyway` / `Discuss further`
5. INADEQUATE → 门禁表示故事必须在纳入冲刺前修订：检查清单的 READY 降级为 NEEDS WORK，最终判定行连同缺口重述；询问用户是更新故事还是继续，继续被记录为覆盖，而非 READY
6. NOT ASSESSED（缺少门禁输入）→ 命名缺失输入，绝不读作 ADEQUATE：补充输入并重跑门禁，或 READY 变为 NOT ASSESSED

**断言（5a）：**
- [ ] 技能在决定是否生成 QL-STORY-READY 前使用解析出的审查模式
- [ ] QL-STORY-READY 仅在检查清单判定存在后生成
- [ ] GAPS 结果产生三选项 `AskUserQuestion`
- [ ] INADEQUATE 结果覆盖检查清单的 READY：最终判定为 NEEDS WORK，连同门禁的具体缺口重述
- [ ] 若用户仍继续，判定保持 NEEDS WORK，输出声明覆盖（"proceeding despite QL-STORY-READY: INADEQUATE (user override)"）——绝不静默 READY
- [ ] 门禁返回 NOT ASSESSED 时命名缺失输入，绝不让 READY 保留：除非补充输入并重跑门禁，判定为 NOT ASSESSED

**Case 5b——lean 或 solo 模式：** `--review lean` 或 `--review solo`

**预期行为：**
1. 跳过 QL-STORY-READY
2. 输出注明"QL-STORY-READY skipped — Lean mode."或"QL-STORY-READY skipped — Solo mode."
3. 判定仅基于检查清单

**断言（5b）：**
- [ ] QL-STORY-READY 在 lean 或 solo 模式下不生成
- [ ] 输出中注明跳过并命名模式
- [ ] 判定仅基于检查清单

---

### Case 6：`qa.level: minimal` 下的 UI 故事——截图计划从不豁免

**Fixture：**
- `project.yaml` 设置 `modes.workflow: standard` 和 `qa.level: minimal`（单独设置，因此仅测试证据轴为 `minimal`，每个检查清单组都评估）
- 尚无 `docs/architecture/control-manifest.md` 和 `tr-registry.yaml`
- 故事 `production/epics/ui/story-003-shop-panel.md` 有 `Type: UI`、引用其所实现需求的 `GDD: design/gdd/shop.md` 引用、两个命名商店面板显示内容的可测试验收标准、"No ADR applies — pure layout"、引擎说明、估算、Out of Scope 声明、`Dependencies: None` 和性能说明——且无 `## Test Evidence` 章节，也无其他提及截图将放置何处
- 无 TBD / UNRESOLVED 标记，无资产路径

**输入：** `/story-readiness production/epics/ui/story-003-shop-panel.md`

**预期行为：**
1. 架构项通过或自动通过（明确"No ADR applies"；无 TR-ID、清单版本或清单可检查）
2. "Test evidence requirement is clear"项仍适用——UI 是 `qa.level: minimal` 自动通过的例外
3. `testing.strict.ui` 未设置，因此该项默认严格：缺失章节使故事为 NEEDS WORK
4. `Fix:` 要求 `## Test Evidence` 章节，命名每个触及屏幕的保留截图将放置于 `production/qa/evidence/` 下的位置

**断言：**
- [ ] 判定为 NEEDS WORK，而非 READY——在 `qa.level: minimal` 下证据项对 UI 故事不自动通过
- [ ] 缺口命名缺失的截图计划，其 `Fix:` 指向 `production/qa/evidence/`
- [ ] 缺失的截图计划是唯一缺口——不标记 ADR、TR-ID 或清单缺口
- [ ] 变体——`Type: Logic` 故事，三个可测试标准且无 `## Test Evidence` 章节，相同配置：证据项在 `qa.level: minimal` 下自动通过，因此仅缺失章节不会使其为 NEEDS WORK

---

### Case 7：NOT ASSESSED——无可评估内容，或无人能读的 ADR

**Case 7a——范围内零故事：**
- `production/epics/` 仅含 `EPIC.md` 索引文件，无故事文件
- **输入：** `/story-readiness all`

**断言（7a）：**
- [ ] 输出为 `NOT ASSESSED — no stories in scope`，命名 `production/epics/**/*.md` 为搜索路径
- [ ] 路由到 `/create-epics [layer]` 然后 `/create-stories [epic-slug]`
- [ ] 不在空列表上打印 `Ready: 0 / Needs Work: 0 / Blocked: 0` 摘要

**Case 7b——引用的 ADR 无可读状态：**
- `project.yaml` 设置 `modes.workflow: full`
- 故事 `production/epics/core/story-save-slots.md` 通过其他所有项并引用 `ADR-0007`；`docs/architecture/adr-0007-save-format.md` 存在但无 `## Status` 章节
- **输入：** `/story-readiness production/epics/core/story-save-slots.md`

**断言（7b）：**
- [ ] ADR 检查为 NOT ASSESSED——状态未知，而非失败——判定为 NOT ASSESSED：非 BLOCKED，非 READY
- [ ] 输出命名 `docs/architecture/adr-0007-save-format.md` 并路由到 `/architecture-decision retrofit docs/architecture/adr-0007-save-format.md`
- [ ] 变体——故事命名 `ADR-0007` 但无此文件：这是关于故事的发现，因此判定为 BLOCKED（"referenced ADR is missing"），而非 NOT ASSESSED

---

### Case 8：Deprecated 或 Superseded 的 ADR 阻断，修复指向故事的 ADR 字段

**Fixture：**
- `project.yaml` 设置 `modes.workflow: standard`（被引用的失效 ADR 在每个层级都阻断；`standard` 表明它不受建议性规则约束）
- 故事 A `production/epics/core/story-save-format.md` 引用 `ADR-0006`；`docs/architecture/adr-0006-save-format.md` 的 `## Status` 为 Superseded by ADR-0009
- 故事 B `production/epics/core/story-input-map.md` 引用 `ADR-