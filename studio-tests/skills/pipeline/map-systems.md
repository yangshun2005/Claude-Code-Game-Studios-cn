# 技能测试规范：/map-systems

## 技能摘要

`/map-systems` 将游戏概念拆解为系统索引。它读取
`design/gdd/game-concept.md`（在 `minimal` 模式下读取 `design/game-brief.md`）和支柱，
枚举显式和隐式系统，将依赖关系映射到分层
（基础 → 核心 → 功能 → 表现 → 打磨），分配优先级层级
（MVP / 垂直切片 / Alpha / 完整愿景），推导设计顺序，并在
批准后基于 `.claude/docs/templates/systems-index.md` 写入
`design/gdd/systems-index.md`。每个协作阶段以 `AskUserQuestion` 结束。

三个总监门禁按顺序运行，各自处于独立阶段：
TD-SYSTEM-BOUNDARY（technical-director）在依赖图批准后运行，
PR-SCOPE（producer）在优先级批准后、写入前运行，CD-SYSTEMS（creative-director）
在索引写入后运行。在 `lean` 和 `solo` 模式下，每个门禁均跳过并附具名说明。
在 `standard`/`full` 模式下，它是 `/design-system` 的前置要求；在 `minimal` 模式下为可选。
判定结果：COMPLETE（索引已写入）或 BLOCKED（用户拒绝写入）。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"May I write"协作协议语言（针对 `design/gdd/systems-index.md`）
- [ ] 末尾有下一步交接（`/design-system`）
- [ ] 在各自阶段记录三个门禁：TD-SYSTEM-BOUNDARY（阶段 3）、PR-SCOPE（阶段 4）、CD-SYSTEMS（阶段 5，写入后）

---

## 总监门禁检查

在 `full` 模式下，按顺序：
1. **TD-SYSTEM-BOUNDARY**（`technical-director`）——在依赖排序
   批准后、优先级分配前。REJECT → 与用户修订边界；
   CONCERNS → 在索引中内联记录。
2. **PR-SCOPE**（`producer`）——在优先级批准后、索引写入前。
   UNREALISTIC → 在写入前提议修订层级分配；
   OPTIMISTIC → 展示建议调整并询问是否应用。
3. **CD-SYSTEMS**（`creative-director`）——在索引写入后；它接收
   索引路径、支柱和核心幻想（在 `minimal` 模式下为简报的推介语和"他们的
   感受"行）、每个层级的分配以及高风险和瓶颈系统。REJECT → 在 GDD 编写前
   修订系统集；CONCERNS → `AskUserQuestion`（`修订系统集` / `接受——记录到
   索引中` / `进一步讨论`）；选择接受时，它们被记录为
   `> **Creative Director Note** (<tier>): …`，直接位于
   `## Priority Tiers` 表格下方。

三者中任何一个都可能回答 NOT ASSESSED [missing input]；这绝不被视为
批准——需补充输入并重新运行门禁，或继续运行并在索引中记录
`[GATE-ID]: NOT ASSESSED — [input]`。

在 `lean` 模式下，每个门禁均跳过并附说明："TD-SYSTEM-BOUNDARY skipped — Lean
mode."、"PR-SCOPE skipped — Lean mode."、"CD-SYSTEMS skipped — Lean mode."

在 `solo` 模式下，出现同样的三条说明，但标注"Solo mode."

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 均设置 `modes.rigor: minimal` 和
`modes.automation: collaborative`，且无 `project.local.yaml`。规则：
`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——概念拆解并在完整审查模式下写入索引

**夹具：**
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: full`
- `design/gdd/game-concept.md` 存在，核心机制包含战斗和
  物品栏，并有 MVP 定义章节
- `design/gdd/game-pillars.md` 存在，含 2 个支柱
- 尚无 `design/gdd/systems-index.md`
- TD-SYSTEM-BOUNDARY 返回 APPROVE，PR-SCOPE 返回 REALISTIC，CD-SYSTEMS
  返回 APPROVE

**输入：** `/map-systems`

**预期行为：**
1. 读取概念和支柱；glob `design/gdd/*.md`
2. 按类别呈现系统——名称、类别、一句话描述、
   显式或隐式（例如战斗隐含生命值、伤害计算、命中
   检测、战斗 UI）——并询问缺失 / 合并 / 不需要的系统
3. 呈现分层依赖图（瓶颈、叶节点、任何循环）并
   询问"这个依赖排序看起来对吗？…"
4. 批准后启动 TD-SYSTEM-BOUNDARY，传入依赖图（每个
   系统 → 它依赖什么）、层级分配、瓶颈系统以及
   任何循环及其建议解决方案
5. 呈现优先级层级，推理引用本项目支柱；批准后
   启动 PR-SCOPE
6. 呈现摘要（按类别计数、MVP 数量、设计顺序前 3 个、
   高风险项）并询问"我可以将系统索引写入
   `design/gdd/systems-index.md` 吗？"
7. 写入索引，然后启动 CD-SYSTEMS，传入索引路径、支柱/核心
   幻想、每个层级的系统分配以及高风险和瓶颈
   系统
8. 更新 `production/session-state/active.md`；判定：COMPLETE；提供
   `/design-system [first-system-in-order]`

**断言：**
- [ ] 隐式系统被推断并标记为隐式，且给出原因
- [ ] TD-SYSTEM-BOUNDARY 在依赖图批准后、优先级呈现前启动，并传入完整依赖图——而非摘要
- [ ] PR-SCOPE 在优先级批准后、写入询问前启动
- [ ] 询问"我可以将系统索引写入 `design/gdd/systems-index.md` 吗？"且批准前不写入任何内容
- [ ] CD-SYSTEMS 仅在索引写入后启动，并接收全部四个层级和高风险系统——而非仅 MVP 列表
- [ ] 系统枚举表保留模板的列（`# / System Name / Category / Priority / Status / Design Doc / Depends On`）——不添加列
- [ ] `active.md` 已更新，判定为 COMPLETE

---

### 用例 2：失败路径——未找到游戏概念

**夹具：**
- `project.yaml`：`modes.workflow: standard`
- `design/gdd/game-concept.md` 和 `design/game-brief.md` 均不存在

**输入：** `/map-systems`

**预期行为：**
1. 阶段 1 未找到概念
2. 技能输出："No game concept found. Run `/brainstorm` first to create one,
   then come back to decompose it into systems."
3. 技能停止——无枚举、无门禁、无索引

**断言：**
- [ ] 打印"No game concept found"消息
- [ ] 技能推荐 `/brainstorm` 作为下一步操作
- [ ] 不呈现系统枚举，不启动任何门禁
- [ ] 不创建 `design/gdd/systems-index.md`，不输出判定 COMPLETE

---

### 用例 3：总监门禁——CD-SYSTEMS 在写入后返回 CONCERNS

**夹具：**
- 游戏概念和支柱存在；尚无索引
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: full`
- TD-SYSTEM-BOUNDARY 返回 APPROVE，PR-SCOPE 返回 REALISTIC
- CD-SYSTEMS 返回 CONCERNS：核心幻想隐含的一个 MVP 系统缺失
- 在 CONCERNS 问题处用户选择 `接受——记录到索引中`

**输入：** `/map-systems`

**预期行为：**
1. 在"May I write"批准后写入索引
2. 启动 CD-SYSTEMS 并返回 CONCERNS
3. 通过 `AskUserQuestion` 展示关注点：`修订系统集` /
   `接受——记录到索引中` / `进一步讨论`
4. 选择接受时，关注点被记录到索引中为
   `> **Creative Director Note** (MVP): …`，直接位于 `## Priority Tiers`
   表格下方
5. 会话状态已更新；判定：COMPLETE

**断言：**
- [ ] CD-SYSTEMS 仅在索引写入后启动，而非在写入询问前
- [ ] CONCERNS 作为选择呈现给用户——用户选择前不编辑索引
- [ ] 仅接受选项（其命名了编辑）导致注释被写入；注释直接位于 `## Priority Tiers` 表格下方并命名 MVP 层级
- [ ] 若用户选择 `修订系统集`，受影响的系统将与用户一起重做，并再次询问索引重写
- [ ] CONCERNS 不触发完整重新拆解（该路径属于 REJECT）
- [ ] 判定为 COMPLETE

---

### 用例 4：边缘情况——系统索引已存在

**夹具：**
- `project.yaml`：`modes.workflow: standard`
- `design/gdd/game-concept.md` 存在
- `design/gdd/systems-index.md` 已存在，含 8 个系统，其中 3 个已设计

**输入：** `/map-systems`

**预期行为：**
1. 技能读取现有索引并呈现其当前状态
2. 技能询问："The systems index already exists with 8 systems (3 designed, 5 not
   started). What would you like to do?"，选项为"用新系统更新索引"、
   "设计下一个未设计的系统"、"审查并修订优先级"
3. 用户选择操作
4. 技能从现有索引继续，而非重新创建

**断言：**
- [ ] 技能在继续前检测并读取现有索引
- [ ] 呈现系统数量及已设计 / 未开始的分项
- [ ] 提供全部三个选项——索引不会被自动覆盖
- [ ] 技能不会在用户未选择的情况下从头重新运行完整拆解

---

### 用例 5：总监门禁——Lean 和 solo 模式跳过全部三个门禁并附说明

**夹具（lean 模式）：**
- 游戏概念存在
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: lean`

**Lean 模式预期行为：**
1. 系统、依赖和优先级如用例 1 一样呈现并批准
2. 在各自阶段注明"TD-SYSTEM-BOUNDARY skipped — Lean mode."和"PR-SCOPE skipped — Lean
   mode."
3. 询问"May I write"；批准后写入索引
4. 注明"CD-SYSTEMS skipped — Lean mode."
5. 会话状态已更新，打印判定：COMPLETE

**断言（lean 模式）：**
- [ ] 出现全部三条跳过说明，各自处于其阶段
- [ ] 不启动任何总监代理
- [ ] "May I write"询问仍在写入前
- [ ] 即使 CD-SYSTEMS 被跳过，`active.md` 仍更新并打印判定：COMPLETE

**夹具（solo 模式）：**
- 相同概念，`project.yaml`：`modes.workflow: standard`、`modes.review_mode: full`
- 输入：`/map-systems --review solo`

**Solo 模式预期行为：**
1. `--review solo` 覆盖已解析的 `full`
2. 同样三个门禁被跳过，标注"Solo mode."

**断言（solo 模式）：**
- [ ] 全部三条跳过说明均带"Solo mode."标签
- [ ] 除此之外行为与本技能的 lean 模式相同

---

### 用例 6：引导模式——新索引仍需询问；更新则不需要

**夹具（新索引）：**
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: solo`、
  `modes.automation: guided`
- 游戏概念存在；无 `design/gdd/systems-index.md`

**夹具（更新）：**
- 同上，但 `design/gdd/systems-index.md` 存在且用户选择了"用新系统更新
  索引"

**输入（两个夹具）：** `/map-systems`

**预期行为：**
1. 新索引：在摘要后询问"我可以将系统索引写入
   `design/gdd/systems-index.md` 吗？"，仅在同意后写入——`guided` 对新文件
   询问"May I write?"
2. 更新：呈现摘要，命名 `design/gdd/systems-index.md`，不等待明确同意即写入
   更新，并说明写入了什么

**断言：**
- [ ] `guided` 下的新索引：显示写入询问，回答前不写入任何内容
- [ ] `guided` 下的更新：无写入询问；写入前显示摘要和目标，写入后报告结果

---

### 用例 7：总监门禁——TD-SYSTEM-BOUNDARY 返回 REJECT

**夹具：**
- `project.yaml`：`modes.workflow: standard`、`modes.review_mode: full`
- 游戏概念和支柱存在；尚无索引
- TD-SYSTEM-BOUNDARY 返回 REJECT：两个系统共享同一职责，
  必须合并

**输入：** `/map-systems`

**预期行为：**
1. 依赖图批准后启动 TD-SYSTEM-BOUNDARY
2. 呈现 REJECT，命名两个系统
3. 与用户修订边界；展示修订后的依赖图
4. 仅在此之后才呈现优先级并启动 PR-SCOPE

**断言：**
- [ ] REJECT 未解决时优先级分配不开始
- [ ] 在呈现优先级前向用户展示修订后的边界
- [ ] 修订前不启动 PR-SCOPE，不写入任何内容
- [ ] REJECT 不被记录为注释并跳过——该处理属于 CONCERNS

---

## 协议合规性

- [ ] 在任何拆解前读取游戏概念（或 `minimal` 模式下的简报）和支柱
- [ ] 在枚举、依赖、优先级和写入处使用 `AskUserQuestion`
- [ ] 写入前询问"我可以将系统索引写入 `design/gdd/systems-index.md` 吗？"——在 `collaborative` 下，以及 `guided` 下对新索引
- [ ] CD-SYSTEMS 的 CONCERNS 仅通过用户选择进入已写入的索引；TD-SYSTEM-BOUNDARY 的 CONCERNS 在索引中内联记录
- [ ] 在 full 模式下门禁按顺序在各自阶段运行，绝不在其输入存在前运行
- [ ] 在 lean/solo 输出中按名称和模式注明跳过的门禁
- [ ] 以下一步交接结束：`/design-system [next-system]`

---

## 覆盖说明

- 循环依赖检测及其建议解决方案属于阶段 3——
  此处未独立进行夹具测试。
- PR-SCOPE UNREALISTIC、CD-SYSTEMS REJECT 路径以及三个门禁中任何一个的
  NOT ASSESSED 回答均未提供夹具。
- `autonomous`（通过 `log_decision` 记录枚举并在不询问的情况下写入）未测试。
- `next` 和 `[system-name]` 参数模式（交接至 `/design-system`）此处
  未测试——它们是索引后的便利功能。
- `docs.density` 深度（每个系统的叙述放在哪里）未锁定夹具。