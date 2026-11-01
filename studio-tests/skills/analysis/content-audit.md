# 技能测试规范：/content-audit

## 技能摘要

`/content-audit [system-name | --summary]` 将设计规定的内容数量与已构建的内容进行对比。它读取 `design/gdd/systems-index.md` 获取系统及其 MVP/优先级层级；当注册表有条目时，从 `design/registry/entities.yaml` 的 `entities:` 和 `items:` 部分获取计划中的具名内容；并用两次 grep 预扫描 GDD（检查 Summary 是否存在，以及数字加内容名词的计数），然后只完整读取它需要的 GDD（阶段 1）。接着它对每种内容类型 glob 实现目录——`assets/data/**/enemies/**`、`assets/data/**/items/**`、已解析代码根下的场景文件等（阶段 2）——并生成缺口表 `System | Content Type | Specified | Found | Gap | Status`，每行状态为 COMPLETE / IN PROGRESS / EARLY / NOT STARTED，带 HIGH PRIORITY 标记，以及一行摘要（阶段 3）。完整报告仅在"May I write"之后写入 `docs/content-audit-[YYYY-MM-DD].md`；`--summary` 不写入任何内容。`modes.workflow` 设定范围：`full` 审计每个 GDD 计数，`standard` 只统计必需 GDD 章节中的计数（并列出被排除的章节），`minimal` 没有可对照审计的系统索引。运行以 COMPLETE 结束，或在缺少必需输入、范围内无任何计数、或无法取得 Found 计数（代码根未解析）时以 `NOT ASSESSED — NO DATA` 结束——无法计算缺口的运行绝不标记为 COMPLETE。不调用任何主管门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含状态和判定关键词：COMPLETE、IN PROGRESS、EARLY、NOT STARTED、NOT ASSESSED
- [ ] 在可选报告写入 `docs/content-audit-[YYYY-MM-DD].md` 之前包含"May I write"
- [ ] 具有下一步交接（缺口表审查后该做什么）

---

## 主管门禁检查

无。内容审计是分析技能；不调用任何门禁。

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有规定内容均存在

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/gdd/systems-index.md` 列出 Enemies（MVP）和 Items（MVP）
- `design/registry/entities.yaml` 无条目
- `design/gdd/enemies.md` 规定"4 种敌人类型：Grunt、Sniper、Tank、Boss"
- `design/gdd/items.md` 规定"3 个物品"
- `assets/data/enemies/` 包含 `grunt.json`、`sniper.json`、`tank.json`、`boss.json`
- `assets/data/items/` 包含 3 个物品 `.json` 文件

**输入：** `/content-audit`

**预期行为：**
1. 技能读取系统索引；空注册表被跳过，改用 GDD 扫描
2. 预扫描 grep 发现两个 GDD 都声明了内容计数；两者都被完整读取
3. 技能 glob `assets/data/**/enemies/**` 和 `assets/data/**/items/**`
4. 缺口表：Enemies——Specified 4，Found 4，Gap 0，COMPLETE；Items——Specified 3，Found 3，Gap 0，COMPLETE
5. 摘要行：规定 7，找到 7，总体缺口 0%
6. 技能呈现表格并询问"May I write the full report to `docs/content-audit-[YYYY-MM-DD].md`?"
7. 输出以 `Verdict: COMPLETE — content audit finished.` 结束

**断言：**
- [ ] 缺口表使用 System、Content Type、Specified、Found、Gap、Status 列
- [ ] 两行均显示 Specified = Found，Gap 0，Status COMPLETE
- [ ] 摘要行报告规定 7、找到 7、缺口 0%
- [ ] 未触发 HIGH PRIORITY 标记
- [ ] 在"May I write"询问得到肯定答复之前不写入任何文件

---

### 用例 2：发现缺口——MVP 敌人内容几乎未开始

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/gdd/systems-index.md` 将 Enemies 标记为 MVP
- `design/gdd/enemies.md` 规定"3 种敌人类型：Grunt、Sniper、Boss"
- `assets/data/enemies/` 仅包含 `grunt.json`

**输入：** `/content-audit enemies`

**预期行为：**
1. 单系统审计：技能跳过预扫描，完整读取 `design/gdd/enemies.md`
2. 技能找到 1 个敌人定义文件
3. 缺口表行：Enemies——Specified 3，Found 1，Gap 2，Status EARLY（33%）
4. Enemies 被标记 HIGH PRIORITY：状态 EARLY 且标记为 MVP
5. 技能询问"May I write the full report to `docs/content-audit-[YYYY-MM-DD].md`?"；若同意，报告的 HIGH PRIORITY Gaps 部分列出 Enemies 及其理由
6. 下一步包括"为每个 HIGH PRIORITY 缺口运行 `/create-stories [epic-slug]`。"

**断言：**
- [ ] Specified（3）、Found（1）和 Gap（2）均显示
- [ ] 3 个中 1 个（33%）的状态为 EARLY，而非 IN PROGRESS 或 NOT STARTED
- [ ] Enemies 因处于 EARLY 且标记为 MVP 而被标记 HIGH PRIORITY
- [ ] 技能现在标记缺口；不假设内容稍后会添加
- [ ] 下一步指向 `/create-stories` 处理 HIGH PRIORITY 缺口

---

### 用例 3：无设计输入——无可对照审计的内容

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/gdd/systems-index.md` 不存在
- `design/gdd/` 不含任何 GDD
- `design/registry/entities.yaml` 无条目

**输入：** `/content-audit`

**预期行为：**
1. 技能将输入记录为 FOUND / ABSENT；系统索引、GDD 和注册表条目均为 ABSENT
2. 所有必需输入均为 ABSENT，因此技能停止
3. 输出报告 `NOT ASSESSED — NO DATA`，指明缺失的系统索引和 GDD，以及生成它们的技能（`/map-systems`、`/design-system`）
4. 不生成缺口表和缺口百分比（无除以零）
5. 不提供报告写入，不写入任何文件

**断言：**
- [ ] 判定为 `NOT ASSESSED — NO DATA`，而非 COMPLETE
- [ ] 不生成缺口表或总体缺口百分比
- [ ] 输出指明缺失的输入及生成它们的技能
- [ ] 不出现"May I write"提示，不写入任何文件

---

### 用例 3b：处处无计数——GDD 存在但均未给出计数

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/gdd/systems-index.md` 列出 Core Loop
- `design/gdd/core-loop.md` 有 `## Summary`，仅描述机制——无内容计数，无具名列表
- `design/registry/entities.yaml` 无条目

**输入：** `/content-audit`

**预期行为：**
1. 系统索引和 GDD 均为 FOUND，因此技能不会在输入不足检查处停止
2. 扫描 (b) 无匹配；`core-loop.md` 有 Summary，因此标记为"无可审计的内容计数"，不完整读取
3. 无行带有 Specified 计数，因此不计算 Gap、Status 或总体缺口百分比（Specified 为 0）
4. 运行以 `Verdict: NOT ASSESSED — NO DATA` 结束，说明没有 GDD 给出计数，并指明用 `/design-system` 添加计数

**断言：**
- [ ] 不计算总体缺口百分比，无行标记为 COMPLETE
- [ ] 判定为 `NOT ASSESSED — NO DATA`，绝不在空缺口表上标记 COMPLETE
- [ ] 输出说明没有 GDD 给出计数，并指明 `/design-system`

---

### 用例 4：边界情况——Summary 之前的 GDD 描述内容但无计数

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/gdd/systems-index.md` 列出 Enemies 和 Quests
- `design/gdd/enemies.md` 有 `## Summary`，规定"3 种敌人类型：Grunt、Sniper、Boss"；`assets/data/enemies/` 有全部 3 个
- `design/gdd/quests.md` 无 `## Summary` 章节，称游戏有"少量支线任务"，无数字或具名列表
- 用户对报告写入回答是

**输入：** `/content-audit`

**预期行为：**
1. 技能 glob `design/gdd/*.md` 作为分母：N = 3，因为 glob 也计入 `systems-index.md`（在步骤 1 读取；它无 `## Summary`）；Summary grep 仅匹配 `enemies.md`；内容计数 grep 仅匹配 `enemies.md`
2. 对缺失的 Summary 采用失败开放，技能完整读取 `quests.md`，而非将其视为超出范围——失败开放规则优先于内容计数收窄
3. 技能将 Quests 内容类型记录为"Unspecified"并标记为设计缺口；不虚构计数
4. Enemies 行为 COMPLETE（3 个中 3 个）
5. 写入的报告 `docs/content-audit-[YYYY-MM-DD].md` 在"Unspecified Content Counts"下列出 `quests.md`
6. 报告注明计数是基于文件扫描的近似值

**断言：**
- [ ] `quests.md` 因缺少 `## Summary` 而被完整读取，即使内容计数 grep 未匹配它
- [ ] Quests 内容记录为"Unspecified"，绝不给出估算计数
- [ ] 报告的 Unspecified Content Counts 部分指明 `quests.md`
- [ ] Enemies 行仍正常审计（Specified 3，Found 3，COMPLETE）
- [ ] 报告带有近似值说明

---

### 用例 5：门禁合规——无门禁；可选报告需批准

**夹具：**
- `project.yaml`：`modes.workflow: full`，`modes.review_mode: full`
- GDD 规定 10 个内容项；实现目录中找到 9 个

**输入：** `/content-audit`

**预期行为：**
1. 技能读取设计输入并扫描实现；生成缺口表
2. 无论审查模式如何，不调用任何主管门禁
3. 技能在任何写入之前呈现缺口表和摘要
4. 技能询问"May I write the full report to `docs/content-audit-[YYYY-MM-DD].md`?"
5. 若用户拒绝，不写入任何内容；绝不修改任何资产文件

**断言：**
- [ ] 任何审查模式下均不调用主管门禁
- [ ] 呈现缺口表而不自动写入任何文件
- [ ] 提供写入 `docs/content-audit-[YYYY-MM-DD].md` 的报告选项但不强制
- [ ] 技能不修改任何资产文件

---

### 用例 6：标准工作流——仅审计必需章节中的计数

**夹具：**
- `modes.workflow` 解析为 `standard`
- `design/gdd/systems-index.md` 列出 Enemies（MVP）
- `design/gdd/enemies.md` 的 `## Detailed Design` 规定"3 种敌人类型：Grunt、Sniper、Boss"；其 `## Tuning Knobs` 章节提到"2 个精英变体"
- `assets/data/enemies/` 包含 `grunt.json`、`sniper.json`、`boss.json`

**输入：** `/content-audit`

**预期行为：**
1. `## Detailed Design`（`standard` 下的必需章节）中的计数被审计：Enemies——Specified 3，Found 3，Gap 0，COMPLETE
2. "2 个精英变体"计数位于 `## Tuning Knobs`，`standard` 不要求该章节；它不获得缺口行，并被列为在该层级未审计
3. 输出以 `Verdict: COMPLETE — content audit finished.` 结束

**断言：**
- [ ] Detailed Design 计数被审计，Enemies 行为 COMPLETE
- [ ] Tuning Knobs 计数不在缺口表或总计中
- [ ] 输出指明 Tuning Knobs 计数在 `standard` 下未审计，而非静默丢弃

---

### 用例 7：最小工作流——无可对照审计的系统索引

**夹具：**
- `project.yaml`：`modes.rigor: minimal`（`workflow` 解析为 `minimal`）
- `design/game-brief.md` 存在；`design/gdd/systems-index.md` 不存在

**输入：** `/content-audit`

**预期行为：**
1. 系统索引为 ABSENT，这在 `minimal` 下是正常的
2. 输出报告 `NOT ASSESSED — NO DATA`，指明缺失的系统索引和创建它的 `/map-systems`
3. 无缺口表、无缺口百分比、无报告写入

**断言：**
- [ ] 判定为 `NOT ASSESSED — NO DATA`，而非 COMPLETE
- [ ] 输出指明 `design/gdd/systems-index.md` 缺失，`/map-systems` 是生成它的技能
- [ ] 不计算缺口百分比，不写入任何文件

---

### 用例 8：代码根未解析——关卡计数未取得

**夹具：**
- `modes.workflow` 解析为 `full`
- `project.yaml` 无 `engine.name`；`.claude/docs/technical-preferences.md` 内容为 `[TO BE CONFIGURED]`；`src/` 和 `Source/` 均存在，因此目录树无法决定根
- `design/gdd/systems-index.md` 列出 Levels（MVP）和 Enemies（MVP）
- `design/gdd/levels.md` 规定"6 个关卡"；`design/gdd/enemies.md` 规定"3 种敌人类型：Grunt、Sniper、Boss"
- `assets/data/enemies/` 包含 3 个敌人文件；`assets/` 下无场景文件

**输入：** `/content-audit`

**预期行为：**
1. Enemies 从 `assets/data/` 审计：Specified 3，Found 3，COMPLETE
2. 关卡扫描需要代码根，而代码根未解析：Levels 行的 Found 为 `NOT ASSESSED — code root unresolved`——不是 0，其 Status 也不是 NOT STARTED
3. Levels 不获得 HIGH PRIORITY 标记，且不进入两个摘要总计；摘要中指明它
4. 运行以 `Verdict: NOT ASSESSED — NO DATA` 结束，指明未解析的代码根

**断言：**
- [ ] Levels 行显示 `NOT ASSESSED — code root unresolved`，绝不显示 Found 0 或 NOT STARTED
- [ ] 摘要总计排除 Levels 并说明
- [ ] 判定为 NOT ASSESSED，而非 COMPLETE

---

## 协议合规

- [ ] 在扫描实现之前读取系统索引、注册表（当有条目时）和范围内的 GDD
- [ ] 无 `## Summary` 的 GDD 即使内容计数 grep 未匹配也被完整读取
- [ ] 应用已解析的 `modes.workflow`：`standard` 仅审计必需 GDD 章节中的计数，并列出被排除的章节
- [ ] 缺口表显示 System、Content Type、Specified、Found、Gap、Status
- [ ] 行 Status 为以下之一：COMPLETE、IN PROGRESS、EARLY、NOT STARTED；无法取得 Found 的行标记为 `NOT ASSESSED` 且不进入总计
- [ ] 仅在"May I write"之后写入，且仅写入 `docs/content-audit-[YYYY-MM-DD].md`；`--summary` 不写入任何内容
- [ ] 缺少必需输入、范围内无计数、或未取得 Found 计数，均产生 `NOT ASSESSED — NO DATA`，绝不产生计算出的缺口百分比或 COMPLETE
- [ ] 不调用任何主管门禁
- [ ] 在 Unity 上，每个内容 glob 也在 `Assets/` 下运行且不含 `data/` 段（例如 `Assets/**/Items/**`、`*Item*.asset`）——Unity 项目的物品、能力、任务和对话绝不因仅扫描 `assets/data/` 而被报告为 NOT STARTED

---

## 覆盖说明

- `--summary` 模式（打印表格、不写入任何内容、以"运行不带 `--summary` 的 `/content-audit` 以写入完整报告。"结束）没有单独用例。
- Unity