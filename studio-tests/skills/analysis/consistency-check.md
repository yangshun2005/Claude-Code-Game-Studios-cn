# 技能测试规范：/consistency-check

## 技能摘要

`/consistency-check [full | since-last-review | entity:<name> | item:<name>]`
将 `design/gdd/` 中每个在范围内的 GDD 与实体注册表
（`design/registry/entities.yaml`）进行比对。它读取一次注册表，构建实体、
物品、公式和常量的查找表，然后对每个 GDD 用 grep 搜索每个已注册名称
（`-C 3`），而不是读取整个文档；只有冲突才做针对性的
章节读取。注册表条目的 `source:` GDD 是权威归属方。
在有效层级为 `standard` 的 GDD 中——即项目层级，或该
系统的 `system_overrides` 行——只比对必需的 GDD 章节
（当 `workflow_overrides.tuning_knobs` 为 true 时再加上 Tuning Knobs），
报告会说明这一点。发现的问题分类为 🔴 CONFLICT、⚠️ STALE REGISTRY 或 ℹ️ UNVERIFIABLE，
报告结论为 PASS 或 CONFLICTS FOUND——或当注册表为空、
指定实体不在其中、或没有 GDD 在范围内时为 NOT ASSESSED，
因为对空内容做扫描不算 PASS。注册表修正只在
"May I update / May I add" 询问之后才写入；之后结论为 COMPLETE，或
在冲突仍存在时为 BLOCKED。每个 🔴 CONFLICT 在"May I append"询问后
追加到 `docs/consistency-failures.md`，面包屑追加到
`production/session-state/active.md`，技能以
`AskUserQuestion` 组件收尾（协作和引导模式）。无总监门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含结论关键词：PASS、CONFLICTS FOUND、NOT ASSESSED、COMPLETE、BLOCKED
- [ ] 分析阶段（1–5）为只读
- [ ] 结尾有下一步交接
- [ ] 注册表写入前有"May I update" / "May I add"措辞，并指明 `design/registry/entities.yaml`
- [ ] 冲突日志写入前有"May I append"措辞，并指明 `docs/consistency-failures.md`

---

## 总监门禁检查

无总监门禁——此技能不生成任何总监门禁代理。一致性
检查是机械扫描；扫描本身不需要创意或技术总监
审查。

---

## 夹具默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和
`modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身
夹具另有说明。规则：`studio-tests/CLAUDE.md` §
夹具默认值。

---

## 测试用例

### 用例 1：正常路径——注册表与 4 个 GDD 一致

**夹具：**
- `modes.workflow` 解析为 `full`；`modes.automation` 解析为 `collaborative`
- `design/registry/entities.yaml` 有 3 个实体和 2 个常量，每个都有 `source:` GDD
- `design/gdd/` 包含 4 个系统 GDD 以及 `game-concept.md`、`systems-index.md` 和一个 `gdd-cross-review-[date].md`
- GDD 中为已注册名称声明的每个值都与注册表匹配

**输入：** `/consistency-check`

**预期行为：**
1. 技能读取注册表并报告 `Registry loaded: 3 entities, 0 items, 0 formulas, 2 constants` 和 `Scope: full`
2. 技能 glob `design/gdd/*.md`，排除 `game-concept.md`、`systems-index.md` 和交叉审查报告，并在扫描前报告 4 个在范围内的 GDD
3. 每个已注册名称在范围内的 GDD 中用 grep 搜索，带 3 行上下文；不完整读取任何 GDD
4. 无冲突或过期条目；报告的 Clean Entries 行显示 `✅ 5 registry entries verified across all GDDs with no conflicts.`
5. 结论：PASS
6. 不向 `docs/consistency-failures.md` 追加任何内容；面包屑追加到 `production/session-state/active.md`
7. 以 `AskUserQuestion` 组件收尾（Fix / Run `/design-review` / Stop）

**断言：**
- [ ] 在扫描前报告在范围内的 GDD 列表（4 个 GDD，非系统文档已排除）
- [ ] 扫描用 grep 搜索已注册名称，而非完整读取每个 GDD
- [ ] 注册表与 GDD 一致时结论为 PASS
- [ ] 不写入 `design/registry/entities.yaml` 和 `docs/consistency-failures.md`；唯一的写入是 `active.md` 面包屑
- [ ] 面包屑指明 `docs/consistency-failures.md`，而非虚构的报告文件
- [ ] 技能以 `AskUserQuestion` 组件收尾，而非纯文本

---

### 用例 2：失败路径——某个 GDD 与已注册常量矛盾

**夹具：**
- `modes.workflow` 解析为 `full`
- 注册表 `constants:` 有 `crit_multiplier` 值 1.5，`source: design/gdd/combat.md`，`referenced_by: [combat.md, loot.md]`
- `design/gdd/combat.md` 声明 `crit_multiplier = 1.5`
- `design/gdd/loot.md` 声明 `crit_multiplier = 2.0`
- 注册表条目比 `combat.md` 的最后一次提交更新（未过期）

**输入：** `/consistency-check`

**预期行为：**
1. 常量扫描（3d）grep `crit_multiplier` 并在 `loot.md` 中找到 2.0
2. 阶段 4 对冲突章节做针对性读取，并使用注册表 `source:` 判定 `combat.md` 为权威
3. 报告列出 `🔴 crit_multiplier`：Registry（source: combat.md）= 1.5；Conflict in loot.md = 2.0；Resolution：将 `loot.md` 改为 1.5
4. 结论：CONFLICTS FOUND
5. 技能询问"May I append 1 entries to `docs/consistency-failures.md` (creating it if absent)?"；批准后追加该冲突的条目（若文件不存在则先创建并写入其表头），指明 `combat.md` 与 `loot.md`
6. 不自动编辑任何 GDD；冲突未解决时最终结论为 BLOCKED

**断言：**
- [ ] 结论为 CONFLICTS FOUND（非 PASS）
- [ ] 冲突条目指明两个 GDD 并显示两个值（1.5 和 2.0）
- [ ] 冲突分类为 🔴 CONFLICT，解决方案针对非源 GDD（`loot.md`）
- [ ] 仅在指明 `docs/consistency-failures.md` 的"May I append"询问获批后，才向其追加反思日志条目
- [ ] 技能不自动解决冲突——用户未选择修复前不编辑任何 GDD
- [ ] 冲突未解决时最终结论为 BLOCKED

---

### 用例 3：过期注册表——源 GDD 在注册表条目之后变更

**夹具：**
- `modes.workflow` 解析为 `full`
- 注册表 `entities:` 有 `goblin`，`health: 40`，`source: design/gdd/combat.md`
- `design/gdd/combat.md` 现在说 goblin 生命值为 50；`git log` 显示它在注册表条目写入后变更
- 没有其他 GDD 声明 goblin 生命值

**输入：** `/consistency-check entity:goblin`

**预期行为：**
1. 范围限定为所有 GDD 中的 `goblin` 实体；报告显示 `Scope: entity:goblin`
2. 在源 GDD 本身发现不匹配；阶段 4 检查 git 历史并分类为 ⚠️ STALE REGISTRY，而非 🔴 CONFLICT
3. 报告在 Stale Registry Entries 下列出：注册表说 40，源 GDD 现在说 50
4. 技能询问："May I update `design/registry/entities.yaml` to fix the 1 stale entries?"
5. 批准后：`health` 变为 50，`revised:` 设为今天，并添加 `# was: 40 before [date]` 注释；不删除任何内容
6. 写入后结论为 COMPLETE
7. 不向 `docs/consistency-failures.md` 追加条目（只记录 🔴 CONFLICT 条目）

**断言：**
- [ ] 发现分类为 ⚠️ STALE REGISTRY，区别于 🔴 CONFLICT
- [ ] 在"May I update `design/registry/entities.yaml`"询问获批前不写入注册表
- [ ] 获批的更新设置 `revised:` 并在 `# was:` 注释中保留旧值
- [ ] 写入后结论为 COMPLETE
- [ ] 过期注册表发现不写入 `docs/consistency-failures.md`

---

### 用例 4：边界情况——空注册表，无 GDD

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/registry/entities.yaml` 是随附的存根，无条目
- `design/gdd/` 为空

**输入：** `/consistency-check`

**预期行为：**
1. 技能读取注册表并发现无条目
2. 技能输出："Entity registry is empty. Run `/design-system` to write GDDs — the registry is populated automatically after each GDD is completed. Nothing to check yet."
3. 结论：**NOT ASSESSED — entity registry empty**；技能停止，不扫描
4. 不写入任何文件——不写注册表、不写反思日志、不写会话状态面包屑

**断言：**
- [ ] 技能输出空注册表消息并指明 `/design-system`
- [ ] 结论为 NOT ASSESSED，以空注册表为原因——绝不是 PASS、CONFLICTS FOUND、COMPLETE 或 BLOCKED，也绝不是完全没有结论（`.claude/rules/skill-authoring.md` 义务 1）
- [ ] 不写入任何文件，包括 `production/session-state/active.md`
- [ ] 技能不崩溃或产生部分报告

---

### 用例 4b：边界情况——注册表已填充，但无 GDD 在范围内

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/registry/entities.yaml` 有 3 个实体，每个都有 `source:` GDD
- `design/gdd/gdd-cross-review-[date].md` 存在，且此后没有系统 GDD 变更

**输入：** `/consistency-check since-last-review`

**预期行为：**
1. 技能读取注册表并报告加载了什么
2. `since-last-review` 过滤器找到零个变更的 GDD；在范围内列表为空
3. 结论：**NOT ASSESSED — no GDDs in scope**，指明 `since-last-review` 为
   产生零结果的过滤器；技能停止，不扫描

**断言：**
- [ ] 结论为 NOT ASSESSED，绝不是 PASS——没有比对任何内容
- [ ] 输出指明产生空范围的过滤器
- [ ] 不扫描任何 GDD，不写入冲突日志条目

---

### 用例 4c：边界情况——指定实体不在注册表中

**夹具：**
- `modes.workflow` 解析为 `full`
- `design/registry/entities.yaml` 有实体 `goblin` 和 `orc_warlord`
- `design/gdd/` 包含 3 个系统 GDD；其中一个提到 `goblin_shaman`

**输入：** `/consistency-check entity:goblin_shaman`

**预期行为：**
1. 技能读取注册表并报告加载了什么，`Scope: entity:goblin_shaman`
2. `goblin_shaman` 没有注册表条目
3. 结论：**NOT ASSESSED — `goblin_shaman` is not in the registry**，列出
   最接近的已注册名称（`goblin`）；技能停止，不扫描

**断言：**
- [ ] 结论为 NOT ASSESSED 并指明未注册实体——绝不是 PASS
- [ ] 列出最接近的已注册名称
- [ ] 不扫描任何 GDD，不写入任何内容

---

### 用例 5：总监门禁——完整审查模式下不生成门禁

**夹具：**
- `project.yaml`：`modes.review_mode: full`，`modes.workflow: full`
- `design/registry/entities.yaml` 有条目
- `design/gdd/` 包含 ≥2 个 GDD

**输入：** `/consistency-check`

**预期行为：**
1. 技能加载注册表并运行一致性扫描
2. 审查模式不起作用：技能的配置块只解析 `automation`、`workflow` 和 `system_overrides`
3. 任何时候都不生成总监门禁代理
4. 正常生成报告和结论

**断言：**
- [ ] 不生成总监门禁代理（无 CD-、TD-、PR-、AD- 前缀的门禁）
- [ ] 技能不解析或读取审查模式——其配置块只请求 `automation`、`workflow` 和 `system_overrides`
- [ ] 输出不包含"Gate: [GATE-ID]"或跳过门禁的条目
- [ ] 审查模式 `full` 对此技能行为无影响

---

### 用例 6：标准工作流——只比对必需的 GDD 章节

**夹具：**
- `modes.workflow` 解析为 `standard`
- 注册表 `constants:` 有 `crit_multiplier` 值 1.5，`source: design/gdd/combat.md`
- `design/gdd/combat.md` 的 `## Detailed Design` 声明 `crit_multiplier = 1.5`
- `design/gdd/loot.md` 仅在其 `## Tuning Knobs` 章节声明 `crit_multiplier = 2.0`，`standard` 不要求该章节；`loot.md` 的任何必需章节都未声明其值

**输入：** `/consistency-check`

**预期行为：**
1. 只比对 `standard` 要求章节内的命中
2. `loot.md` 在 `## Tuning Knobs` 中的值不比对：不为其提出 🔴 CONFLICT，不向 `docs/consistency-failures.md` 追加任何内容
3. 报告的 Scope 行说明检查仅覆盖必需章节，并列出 `loot.md` 的 Tuning Knobs 提及为此层级未检查
4. 结论：PASS——必需章节与注册表一致

**断言：**
- [ ] 在 `standard` 下 Tuning Knobs 值不报告为 🔴 CONFLICT
- [ ] 报告说明仅必需章节的范围，并指明未检查的提及——更窄的扫描不是静默的
- [ ] 结论为 PASS

---

### 用例 6b：标准工作流——按系统覆盖使一个 GDD 为 `full`

**夹具：**
- `modes.workflow` 解析为 `standard`；`project.yaml` 有
  `workflow_overrides.system_overrides.loot: full`
- 注册表 `constants:` 有 `crit_multiplier` 值 1.5，`source: design/gdd/combat.md`
- `design/gdd/combat.md` 的 `## Detailed Design` 声明 `crit_multiplier = 1.5`
- `design/gdd/loot.md` 仅在其 `## Tuning Knobs` 章节声明 `crit_multiplier = 2.0`

**输入：** `/consistency-check`

**预期行为：**
1. 配置块列出 `system_overrides: loot=full`，因此 `loot.md` 的有效层级为 `full`，其所有章节都比对；`combat.md` 保持 `standard`
2. `loot.md` 的 Tuning Knobs 值被比对并作为 `🔴 crit_multiplier` 提出（注册表 1.5，`loot.md` 2.0）
3. 报告的 Scope 行指明 `loot.md` 按 `full` 检查
4. 结论：CONFLICTS FOUND

**断言：**
- [ ] `loot.md` 中的 Tuning Knobs 值被比对，因为其有效层级为 `full`——仅项目层级会跳过它（用例 6）
- [ ] 报告指明有效层级与项目不同的 GDD
- [ ] 结论为 CONFLICTS FOUND，非 PASS

---

### 用例 7：注册表在阶段 1 之后变更——重新读取，然后每个 Edit 锚定在当前文本上

**夹具：**
- `modes.workflow` 解析为 `full`
- 运行开始时的注册表：`entities:` 有 `goblin`（`health: 40`，
  `source: design/gdd/combat.md`）和 `orc`；`items:` 有 `health_potion`
- `design/gdd/combat.md` 说 goblin 生命值为 50（在注册表
  条目写入后变更：过期）；`iron_sword` 在 `combat.md` 和
  `loot.md`