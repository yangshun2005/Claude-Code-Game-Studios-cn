# 技能测试规范：/create-stories

## 技能摘要

`/create-stories` 将单个史诗拆分为可供开发者使用的故事文件。在 `standard`/`full` 工作流下，它读取 `production/epics/[epic-slug]/EPIC.md`、该史诗的 GDD、管辖 ADR（按章节限定读取，绝不无界读取）、控制清单中本层的规则以及 TR 注册表中本系统的条目。在 `minimal` 下，它改为从 `design/game-brief.md` 合成一个隐式史诗，并且只有在写入询问点名该史诗后才写入其 `EPIC.md`。

每个故事按类型分类（Logic / Integration / Visual/Feel / UI / Config/Data），类型决定其必需的测试证据。受 Proposed ADR 管辖的故事设为 `Status: Blocked`。在 `full` 审查模式下，`qa-lead` 以门禁 QL-STORY-READY 被生成**一次**（判定为 ADEQUATE / GAPS / INADEQUATE），并为 ADEQUATE 的故事返回测试用例规格；GAPS 交给用户（Revise / Accept / Discuss），INADEQUATE 会阻塞该故事直到其被修订；在 `lean`/`solo` 下跳过该门禁并附注说明。所有故事在一个摘要中呈现，技能只询问一次——"May I write these [N] stories to `production/epics/[epic-slug]/`, and update `production/epics/[epic-slug]/EPIC.md` and `production/epics/index.md`?"——然后写入 `story-NNN-[slug].md` 文件并更新史诗的 `EPIC.md` 和 `production/epics/index.md`。判定：COMPLETE 或 BLOCKED（用户拒绝）。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"May I write"协作协议语言（对整个故事集一次批准）
- [ ] 末尾有下一步交接（`/story-readiness`、`/dev-story`）
- [ ] 记录当管辖 ADR 为 Proposed 时故事 Status: Blocked
- [ ] 记录 QL-STORY-READY 门禁：full 模式激活，lean/solo 跳过

---

## 导演门禁检查

在 `full` 模式下：所有故事分解完成后、写入询问之前，`qa-lead` 通过 `Agent` 以门禁 QL-STORY-READY 被生成一次。它为每个故事返回判定（ADEQUATE / GAPS / INADEQUATE），并为 ADEQUATE 的故事返回 Given/When/Then 规格（Logic、Integration）或手动验证步骤（Visual/Feel、UI）。GAPS → `AskUserQuestion`，选项为 `Revise flagged criteria` / `Accept and proceed` / `Discuss further`——用户选择前不做任何修订。INADEQUATE → 阻塞：故事与用户一起修订（或丢弃并点名）后才进行写入询问。NOT ASSESSED [missing input] → 绝不读作 ADEQUATE：补充输入并重新请求判定，或在无 qa-lead 规格的情况下写入故事并在故事列表中标记 `QL-STORY-READY: NOT ASSESSED — [input]`。

在 `lean` 模式下：跳过 QL-STORY-READY。输出注明：
"QL-STORY-READY skipped — Lean mode."

在 `solo` 模式下：跳过 QL-STORY-READY。输出注明：
"QL-STORY-READY skipped — Solo mode."

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——史诗含 3 个故事，所有 ADR 均为 Accepted

**夹具：**
- `project.yaml`：`engine.name: Godot`、`modes.workflow: full`、
  `modes.review_mode: lean`、`modes.story_granularity: fine`（每个验收标准一个故事）
- `production/epics/combat/EPIC.md` 存在，含 3 个 GDD 需求并列出其管辖 ADR
- 该史诗的 GDD 存在，每个需求恰好一个验收标准（其中一个是伤害公式）
- 所有管辖 ADR 文件存在，`## Status` 为 Accepted
- `docs/architecture/control-manifest.md` 存在
- `docs/architecture/tr-registry.yaml` 含全部 3 个需求的 TR-ID
- `production/epics/index.md` 有该史诗的一行，值为 `Not yet created`

**输入：** `/create-stories combat`

**预期行为：**
1. 完整读取 EPIC.md 和 GDD；从清单中 grep 本层的 `## <layer> Layer Rules` 和本系统的 TR 注册表条目；通过标题映射 + 限定读取加载每个 ADR
2. 报告 "Loaded epic [name], GDD [filename], [N] governing ADRs [status], [manifest status]."
3. 按类型分类每个故事（伤害公式故事为 Logic）
4. 注明 "QL-STORY-READY skipped — Lean mode."
5. 呈现故事列表（标题、类型、ADR、覆盖的 TR-ID、测试/证据路径）
6. 询问一次："May I write these 3 stories to `production/epics/combat/`, and
   update `production/epics/combat/EPIC.md` and `production/epics/index.md`?"
7. 写入 `story-001-[slug].md` … `story-003-[slug].md`，更新 EPIC.md 的 Stories 表和索引行为 `3 stories`
8. 询问下一步，推荐 `/story-readiness [first-story-path]`；判定：COMPLETE

**断言：**
- [ ] 恰好起草 3 个故事——`fine` 下每个验收标准一个
- [ ] 每个故事文件具有头部字段 Epic、Status、Layer、Type、Manifest Version，以及包含 GDD、`Requirement: TR-[system]-NNN`、ADR Governing Implementation、ADR Version、Engine/Risk 的 Context 块
- [ ] 每个故事具有 Acceptance Criteria、QA Test Cases、Test Evidence 和 Dependencies 章节
- [ ] 公式故事类型为 Logic，证据路径为 `tests/unit/[system]/…`（Godot 测试根）
- [ ] 出现 "QL-STORY-READY skipped — Lean mode."，且每个 QA Test Cases 章节读作 "N/A — no qa-lead specs at this tier…"（无即兴测试用例）
- [ ] "May I write" 在完整列表展示后询问一次，并点名 3 个故事、`EPIC.md` 和 `production/epics/index.md`
- [ ] 文件命名为 `production/epics/combat/story-NNN-[slug].md`
- [ ] 技能不启动实现

---

### 用例 2：失败路径——点名的史诗不存在

**夹具：**
- `project.yaml`：`modes.workflow: full`
- `production/epics/` 包含其他史诗，但没有 `nonexistent-epic/` 目录

**输入：** `/create-stories nonexistent-epic`

**预期行为：**
1. 技能查找 `production/epics/nonexistent-epic/EPIC.md`——未找到
2. 技能输出清晰错误，点名该路径
3. 技能推荐 `/create-epics`（上一步）或对照 `production/epics/` 下的史诗检查 slug
4. 不创建任何故事文件，也不从简报合成史诗

**断言：**
- [ ] 技能输出清晰错误，点名 `production/epics/nonexistent-epic/EPIC.md`
- [ ] 不写入任何故事文件、EPIC.md 或索引行
- [ ] 技能推荐 `/create-epics`
- [ ] 技能在 `full` 下不落入 `minimal` 简报合成分支

---

### 用例 3：阻塞故事——ADR 为 Proposed

**夹具：**
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: solo`、
  `modes.story_granularity: fine`
- EPIC.md 存在，含 2 个需求，各一个验收标准；两个管辖 ADR 文件均存在
- 需求 1 由 `## Status` 为 Accepted 的 ADR 管辖
- 需求 2 由 ADR-0007 管辖，其 `## Status` 为 Proposed

**输入：** `/create-stories [epic-slug]`

**预期行为：**
1. 两个 ADR 文件均存在，因此 `full` 层存在性检查通过
2. 技能读取每个管辖 ADR 的状态，发现 ADR-0007 为 Proposed
3. 故事 2 起草为 `Status: Blocked`，附注
   "BLOCKED: ADR-0007 is Proposed — accept it with `/architecture-decision accept ADR-0007` once decided"
4. 故事 1 起草为 `Status: Ready`
5. 阻塞故事在单次 "May I write" 询问前于故事列表中标记

**断言：**
- [ ] ADR 状态取自 ADR 的 `## Status` 章节，而非假设
- [ ] 故事 2 具有 `Status: Blocked`，附注点名 ADR-0007 和 `/architecture-decision accept`——绝不是裸的 `/architecture-decision`，那会启动新 ADR
- [ ] 变体——ADR-0007 的 `## Status` 读作 `Superseded by ADR-0009`：故事 2 仍为 `Status: Blocked`，其附注点名 ADR-0009 作为应指向的 ADR
- [ ] 故事 1 具有 `Status: Ready`——被阻塞的 ADR 不影响它
- [ ] 阻塞故事在写入批准前标记，而非写入后才发现
- [ ] 两个故事文件均被写入（阻塞故事仍写入——只是标记）

---

### 用例 4：边界情况——未提供参数

**夹具：**
- `project.yaml`：`modes.workflow: standard`
- `production/epics/` 包含 2 个史诗子目录，各有一个 `EPIC.md`

**输入：** `/create-stories`（无参数）

**预期行为：**
1. 技能在 `standard` 下检测到无参数
2. Glob `production/epics/*/EPIC.md`
3. 通过 `AskUserQuestion` 询问："Which epic would you like to break into stories?"
   以 2 个史诗（及其状态）为选项
4. 仅分解用户选择的史诗

**断言：**
- [ ] 技能询问 "Which epic would you like to break into stories?" 而非报错
- [ ] glob 得到的两个史诗均作为选项提供
- [ ] 用户选择前不写入任何故事文件
- [ ] 技能不静默选择史诗而无用户输入
- [ ] 技能在 `standard` 下不采用 `minimal` 简报合成分支

**夹具（无史诗）：**
- `project.yaml`：`modes.workflow: standard`
- `production/epics/` 不含 `EPIC.md`

**预期行为（无史诗）：**
1. glob 返回空，技能停止："No epics found under
   `production/epics/`. Run `/create-epics layer: foundation` first — an epic is
   what this skill decomposes."

**断言（无史诗）：**
- [ ] 不以空选项列表询问 "Which epic…" 问题
- [ ] 停止信息点名 `/create-epics`，且不写入任何内容

---

### 用例 5：导演门禁——Full 模式，一个故事从 QL-STORY-READY 返回 GAPS

**夹具：**
- `project.yaml`：`modes.workflow: full`、`modes.review_mode: full`、
  `modes.story_granularity: fine`
- EPIC.md 存在，含 2 个需求，各一个验收标准；两个管辖 ADR 均为 Accepted
- QL-STORY-READY 对故事 1 返回 ADEQUATE，对故事 2 返回 GAPS（一个模糊的验收标准）
- 在 GAPS 问题处用户选择 `Revise flagged criteria`

**输入：** `/create-stories [epic-slug]`

**预期行为：**
1. 两个故事均被分解
2. `qa-lead` 以 QL-STORY-READY 对整个故事列表生成一次
3. 呈现评估：故事 1 为 ADEQUATE 及其测试用例规格；故事 2 为 GAPS 并点名模糊标准
4. 对故事 2 的 `AskUserQuestion`：`Revise flagged criteria` / `Accept and proceed`
   / `Discuss further`
5. 选择 Revise 时：起草并展示故事 2 的修订标准；仅对故事 2 的规格在一次后续调用中重新请求
6. 两者均为 ADEQUATE 后，呈现故事列表并进行单次写入询问；每个故事的 QA Test Cases 章节包含 qa-lead 规格

**断言：**
- [ ] `qa-lead` 对所有故事生成一次（不是每故事一次，也不为故事 1 的规格第二次生成）；唯一的后续调用仅覆盖故事 2
- [ ] 每故事判定使用 ADEQUATE / GAPS / INADEQUATE 并点名失败标准
- [ ] GAPS 判定连同三个选项交给用户——用户选择前不修订故事 2 的标准
- [ ] 故事 2 的修订标准在写入询问前展示；在 ADEQUATE 前不携带规格
- [ ] 若用户选择 `Accept and proceed`，故事 2 将以未改动的标准写入，其 QA Test Cases 读作 `*Test cases not yet defined — run /qa-plan to generate them.*`
- [ ] 故事 1 的 QA Test Cases 章节包含来自门禁的 Given/When/Then（Logic/Integration）或 Setup/Verify/Pass condition（Visual/Feel、UI）块
- [ ] `full` 模式下不出现 "QL-STORY-READY skipped" 注记

---

### 用例 6：Minimal 层——从简报合成史诗，然后回访

**夹具：**
- `project.yaml`：`engine.name: Godot`、`engine.version: "4.6"`，且无 `modes`
  键——`modes.rigor` 默认为 `minimal`，因此 `workflow`、`qa.level` 和审查模式解析为 `minimal` / `minimal` / `solo`
- `design/game-brief.md` 含工作标题 "Lantern Keeper"、玩家目标与失败状态、2 个 MVP 特性和构建顺序
- `docs/engine-reference/godot/VERSION.md` 缺失
- 无 `production/epics/` 目录

**输入：** `/create-stories`

**预期行为：**
1. `minimal` 下无参数采用步骤 2 的 `minimal` 分支并读取简报；不因缺失 GDD、ADR、控制清单或 TR 注册表而停止
2. 起草 `production/epics/lantern-keeper/EPIC.md` 但不写入
3. 注明 "QL-STORY-READY skipped — Solo mode."
4. 按构建顺序为每个 MVP 特性呈现 2 个故事，并询问一次以写入它们并创建 `production/epics/lantern-keeper/EPIC.md`（无 `index.md`——它不存在）
5. 写入 `EPIC.md` 和两个故事；打印 `Epics index not updated:
   production/epics/index.md absent`
6. 推荐 `/dev-story [first-story-path]`——无 `/story-readiness`，无 `/sprint-plan` 选项

**断言：**
- [ ] 单次写入询问前不写入任何内容，包括 `EPIC.md`，且该询问点名 `EPIC.md`
- [ ] 每个故事读作 `Requirement: Brief MVP feature N`，ADR 字段为 `N/A (minimal — no ADRs)`，Manifest Version 为 `N/A (minimal — no control manifest)`
- [ ] 每个故事的 Risk 为 `NOT ASSESSED (no VERSION.md risk rating)`——绝不是猜测的等级
- [ ] 每个 QA Test Cases 章节读作 "N/A — no qa-lead specs at this tier…" 行
- [ ] 索引行点名 `production/epics/index.md`，而非系统索引

**夹具（回访）：**
- 如上，在该次运行后：`production/epics/lantern-keeper/` 含 `EPIC.md`（2 行 Stories 表）、`story-001-…` 和 `story-002-…`；简报现列出第 3 个 MVP 特性

**预期行为（回访）：**
1. 读取现有故事和 `EPIC.md`；为新特性起草一个故事
2. 询问一次，点名新故事和 `EPIC.md` 更新；写入 `story-003-[slug].md` 并向 Stories 表追加一行

**断言（回访）：**
- [ ] `story-001` 和 `story-002` 不变
- [ ] 新故事编号为 003，接续现有最高编号
- [ ] `EPIC.md` 保留其两行及所有其他章节；恰好追加一行

**夹具（尚无故事的 `EPIC.md`）：**
- 如第一个夹具，但已运行 `/create-epics`，因此 `production/epics/lantern-keeper/EPIC.md` 存在（范围来自简报，Stories 为 "Not yet created"），