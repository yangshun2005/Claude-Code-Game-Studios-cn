# 技能测试规范：/review-all-gdds

## 技能摘要

`/review-all-gdds` 是一个 Opus 级技能，对 `design/gdd/` 中的系统 GDD 执行整体跨 GDD 审查。加载全部内容后（阶段 1），它以 `game-designer` 子代理身份并行运行两项独立审查：阶段 2，跨 GDD 一致性（依赖双向性、规则矛盾、过期引用、归属冲突、公式兼容性、验收标准冲突）；阶段 3，游戏设计整体性（相互竞争的进度循环、注意力预算、主导策略、经济循环、难度曲线、支柱对齐、玩家幻想一致性）。阶段 4 走查多系统场景。它产出 PASS / NOT ASSESSED / CONCERNS / FAIL 判定，并且仅在用户批准后才写入 `design/gdd/gdd-cross-review-[YYYY-MM-DD].md` 和 systems-index 的"Needs Revision"状态。

每个阶段都是 `references/N-*.md` 文件，技能在执行该阶段前读取它，若在该阶段期间对话被压缩则再次读取。参数决定焦点：`full`（默认）、`consistency`、`design-theory` 或 `since-last-review`，后者从 `.claude/scripts/review-scope.sh` 获取范围。

它在单个 GDD 获批之后、架构工作之前被调用。它不生成任何总监门禁代理。在 `workflow: minimal` 下不适用（无 GDD），因此以下每个 fixture 都设置 `modes.workflow: full`。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥5 个阶段标题（复杂多阶段技能）
- [ ] 包含判定关键词：PASS、NOT ASSESSED、CONCERNS、FAIL
- [ ] `allowed-tools` 包含 Write，因此存在"May I write"措辞（阶段 6 报告写入）
- [ ] 末尾有下一步交接
- [ ] 记录了阶段 2（一致性）和阶段 3（设计理论）的并行生成

---

## 总监门禁检查

无总监门禁——此技能不生成任何总监门禁代理，也不解析审查模式。它本身就是整体审查；其唯一的代理是阶段 2 和阶段 3 的两个 `game-designer` 子代理。

---

## Fixture 默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身的 Fixture 另有说明。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——无冲突的干净 GDD 集

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含 `game-concept.md`、`game-pillars.md`（支柱与反支柱）、`systems-index.md` 以及三个系统 GDD——`combat.md`、`inventory.md`、`crafting.md`——每个都包含阶段 1c 加载的所有章节，以及一个已填写的 `## Cross-References` 表格，其所有行均可解析
- 所有 GDD 一致：无公式矛盾、无竞争归属、无过期引用、依赖互惠；每个系统都服务于一个支柱
- 设计理论检查也不会警告任何内容：一个主进度循环、最多同时活跃 4 个系统、每个资源都有来源和去向、缩放曲线兼容、玩家幻想兼容

**输入：** `/review-all-gdds`

**预期行为：**
1. 技能在 `design/gdd/*.md` 中 grep `## Summary` 并显示清单（"Found [N] GDDs. Summaries: …"）
2. 技能读取实体注册表，或注明其为空并建议 `/consistency-check`
3. 技能加载审查章节并报告"Loaded [N] system GDDs covering [M] systems. Pillars: […]. Anti-pillars: […]."
4. 阶段 2 和阶段 3 同时作为 `game-designer` 子代理生成，每个都获得其切片对应的粘贴 GDD 内容（而非文件路径）及其检查清单（2a–2f / 3a–3g）
5. 阶段 4 走查 3–5 个多系统场景
6. 报告显示无阻塞问题且无警告；判定 PASS
7. `AskUserQuestion`："May I write this review to `design/gdd/gdd-cross-review-[date].md`?"
8. 根据项目状态构建的收尾组件

**断言：**
- [ ] 阶段 2 和阶段 3 并行生成（两个 `Agent` 调用都在等待任一结果之前发出）
- [ ] 子代理提示携带 GDD 章节内容，而不仅仅是文件路径
- [ ] 报告说明覆盖集：`GDDs reviewed: [N] of [M] present` 和 `Not read: none`，并列出走查的场景
- [ ] 判定为 PASS 仅因为无阻塞问题**且无警告**，且覆盖三个可读 GDD——单个警告就会使其变为 CONCERNS
- [ ] 报告仅在用户批准后写入，使用 ISO 日期名称（`gdd-cross-review-YYYY-MM-DD.md`）
- [ ] 收尾组件提供 `/create-architecture` 和 `/gate-check`（PASS 下两者均允许）

---

### 用例 2：失败路径——两个 GDD 之间的规则冲突

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `combat.md` 定义了一个下限："Minimum damage after armour reduction is 1"
- `status-effects.md` 陈述了一个绕过它的机制："Poison ignores armour and can reduce damage to 0"
- 两个 GDD 在其他方面完整且有效；`game-pillars.md` 定义了支柱，`systems-index.md` 列出了两个系统

**输入：** `/review-all-gdds`

**预期行为：**
1. 阶段 2 检测到下限/上限矛盾
2. 报告打印一条 `🔴 Rule Contradiction` 条目，引用两条规则并询问哪个 GDD 具有权威性
3. 该问题位于一致性 Issues → 阻塞下；两个 GDD 都出现在"GDDs Flagged for Revision"中
4. 判定：FAIL，并附"If FAIL — required actions before re-running"
5. 在报告写入问题之后，第二个 `AskUserQuestion` 提议在 `systems-index.md` 中将标记的 GDD 标记为 `Needs Revision`

**断言：**
- [ ] 判定为 FAIL（不是 PASS、CONCERNS 或 NOT ASSESSED）
- [ ] 冲突条目中命名了两个 GDD 文件名
- [ ] 引用了具体的矛盾规则（而非含糊的"发现冲突"）
- [ ] 该问题被归类为阻塞（🔴），且两个 GDD 都在标记表中，优先级为 Blocking
- [ ] 技能不决定哪个 GDD 正确，也不编辑任一 GDD
- [ ] 如果用户批准 systems-index 更新，状态写入为精确的 `Needs Revision`，不带括号
- [ ] 收尾组件既不提供 `/create-architecture` 也不提供 `/gate-check`（判定为 FAIL）

---

### 用例 3：部分路径——交叉引用不存在的 GDD

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含 `game-pillars.md`、`systems-index.md` 和两个一致的系统 GDD，`crafting.md` 和 `inventory.md`
- `crafting.md` 的 `## Cross-References` 表格有一行的 Target GDD 为 `system-b.md`；不存在 `design/gdd/system-b.md`
- `systems-index.md` 将 system-b 列为下一个系统，Status: Not Started
- 其他内容不会触发警告

**输入：** `/review-all-gdds`

**预期行为：**
1. 阶段 2 的过期引用检查（2c）确认每个 Cross-References 行的 Target GDD 存在，并发现 `system-b.md` 缺失
2. 由于 `systems-index.md` 列出了 system-b，该引用是对计划工作的依赖：一条 `⚠️ Stale Reference` 警告，命名 `crafting.md` 和缺失目标，待该 GDD 编写时重新检查
3. 未发现其他问题；判定 CONCERNS（有警告，无阻塞）
4. 收尾组件包含 `/design-system system-b`（设计顺序中的下一个）

**断言：**
- [ ] 发现项命名 `crafting.md` 和缺失的 `system-b.md`
- [ ] 它报告在警告（⚠️）下，而非阻塞——因为 systems index 将 system-b 列为计划中
- [ ] 判定为 CONCERNS——对计划工作的引用不是 FAIL，也不是无条件的 PASS
- [ ] 收尾组件提供 `/design-system system-b`，并提供 `/create-architecture` 但不提供 `/gate-check`（后者需要 PASS）
- [ ] 技能不跳过也不静默忽略缺失目标

**用例 3b——目标两处都不存在：** 相同的 fixture，但 `systems-index.md` 没有 system-b 的条目。永远不会有任何东西定义它。

**断言（3b）：**
- [ ] 该引用报告为 🔴 阻塞，且 `crafting.md` 出现在"GDDs Flagged for Revision"中，优先级为 Blocking
- [ ] 判定为 FAIL，且收尾组件既不提供 `/create-architecture` 也不提供 `/gate-check`

---

### 用例 4：边界情况——少于两个系统 GDD

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含 `systems-index.md` 和单个系统 GDD，`combat.md`

**输入：** `/review-all-gdds`

**预期行为：**
1. 技能加载现有内容并统计系统 GDD
2. 少于 2 个 → 技能停止并输出："Cross-GDD review requires at least 2 system GDDs. Write more GDDs first, then re-run `/review-all-gdds`."
3. 不运行任何审查阶段，也不提供报告

**断言：**
- [ ] 技能输出"requires at least 2 system GDDs"停止消息
- [ ] 阶段 2 和阶段 3 子代理均未生成
- [ ] 不为单个文档产出 PASS 判定
- [ ] 不提供报告写入，也不写入任何文件

---

### 用例 5：总监门禁——无论审查模式如何，均不生成

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含用例 1 的集合：概念、支柱、系统索引和三个一致的系统 GDD
- 5a：`modes.review_mode: full`；5b：`modes.review_mode: solo`

**输入：** `/review-all-gdds`

**预期行为：**
1. 解析后的配置块仅请求 `automation`、`workflow` 和 `system_overrides`——不参考审查模式
2. 在 5a 和 5b 中，相同的两个 `game-designer` 子代理运行阶段 2 和阶段 3
3. 不生成任何总监门禁代理；判定正常产出

**断言：**
- [ ] 两种模式下均不生成总监门禁代理（CD-、TD-、PR-、AD-、LP- 前缀的门禁）
- [ ] 审查模式不改变任何内容：在 `solo` 下两个 `game-designer` 子代理仍然运行
- [ ] 输出不包含"Gate: [GATE-ID]"或"[GATE-ID] skipped"条目
- [ ] 技能在两种模式下都产出判定
- [ ] 运行中唯一的 `Agent` 生成是阶段 2 和阶段 3 的 `game-designer` 子代理

---

### 用例 6：NOT ASSESSED——少于两个可读 GDD

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含 `game-pillars.md`、`systems-index.md` 和两个系统 GDD：`combat.md`（完整）和 `inventory.md`（有模板标题但标题下只有占位文本）

**输入：** `/review-all-gdds`

**预期行为：**
1. 存在两个系统 GDD，因此"requires at least 2 system GDDs"停止不会触发
2. `inventory.md` 存在但为空——不可审查——因此只能比较一个 GDD，干净的结果只意味着没有比较任何内容
3. 判定 NOT ASSESSED；"If NOT ASSESSED — what could not be reviewed, and why"命名 `inventory.md` 为仅占位，并说明交叉审查需要两个可读 GDD
4. 报告说明 `GDDs reviewed: 1 of 2 present`

**断言：**
- [ ] 判定为 NOT ASSESSED——不是 PASS，尽管未发现矛盾
- [ ] 报告命名 `inventory.md` 为存在但不可审查，并说明 `GDDs reviewed: 1 of 2 present`
- [ ] 完全无法读取的 GDD 会在每个判定下都列在报告的 `Not read:` 行，并且与空 GDD 一样是 NOT ASSESSED 触发条件
- [ ] 收尾组件既不提供 `/create-architecture` 也不提供 `/gate-check`

---

### 用例 7：阶段文件按需读取，压缩后再次读取

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含用例 1 的集合：概念、支柱、系统索引和三个一致的系统 GDD
- 技能文件夹包含 `references/1-load.md` 到 `references/7-handoff.md`
- 在阶段 4（场景走查）进行期间对话被压缩，然后运行继续

**输入：** `/review-all-gdds`

**预期行为：**
1. SKILL.md 不包含阶段步骤；阶段 1 之前运行读取 `references/1-load.md`，且在该读取之前不扫描 `## Summary` 行或加载 GDD 章节
2. 阶段 2 和阶段 3 作为 `game-designer` 子代理并行生成，主运行在向每个子代理传递检查清单（2a–2f、3a–3g）之前读取 `references/2-consistency.md` 和 `references/3-design-holism.md`
3. `references/4-scenarios.md`、`5-report.md`、`6-write.md` 和 `7-handoff.md` 各自在该阶段工作之前读取：第一个之前无场景，`5-report.md` 之前无判定或报告，`6-write.md` 之前无写入问题，`7-handoff.md` 之前无收尾组件
4. 压缩后，运行在继续走查之前再次读取 `references/4-scenarios.md`，如技能所述："Read it again if the conversation was compacted or resumed during this phase."

**断言：**
- [ ] 转录显示读取了七个 `references/N-*.md` 文件中的每一个，且都在该阶段工作开始之前
- [ ] 任何阶段工作（摘要扫描、检查清单交接、场景、判定、写入问题、收尾组件）都不在其阶段文件读取之前发生
- [ ] 压缩后，在走查恢复之前再次读取 `references/4-scenarios.md`
- [ ] 阶段 2 和阶段 3 子代理仍使用阶段文件中的检查清单项并行生成

---

### 用例 8：`since-last-review`——范围来自脚本，而非对 git 的推理

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/` 包含 `game-pillars.md`、`systems-index.md`、一个已提交的 `gdd-cross-review-2025-01-15.md` 以及四个系统 GDD：`combat.md`、`inventory.md`、`crafting.md`、`status-effects.md`
- 自该报告提交以来只有 `combat.md` 被修改；其 `## Dependencies` 章节命名 `inventory.md`
- `crafting.md` 和 `status-effects.md` 未更改，且没有更改的 GDD 命名它们
- `.claude/scripts/review-scope.sh` 打印 `PRIOR_REVIEW: design/gdd/gdd-cross-review-2025-01-15.md`、`CHANGED:` 含 `design/gdd/combat.md`，以及 `DEPS` 行 `combat.md -> inventory.md`

**输入：** `/review-all-gdds since-last-review`

**预期行为：**
1. 在清单之后，运行执行 `bash .claude/scripts/review-scope.sh`，而非自行从 `git log` 或 `git diff` 计算更改集
2. 范围是 `CHANGED` 集加上 `DEPS` 行命名的 GDD：`combat.md` 和 `inventory.md`；运行在任何完整读取之前向用户显示哪些 GDD 在范围内
3. 只有这两个 GDD 获得阶段 1c 章节加载；`crafting.md` 和 `status-effects.md` 不被加载，也不进行第二次依赖扩展
4. 报告说明 `GDDs reviewed: 2 of 4 present`
5. 阶段 2 和阶段 3 照常对两个范围内的 GDD 运行

**断言：**
- [ ] 运行 `bash .claude/scripts/review-scope.sh`，其 `CHANGED` 和 `DEPS` 列表按打印内容用作范围
- [ ] 运行不通过自己的 git 历史搜索推导更改集
- [ ] `inventory.md` 在范围内，尽管它未更改，因为 `