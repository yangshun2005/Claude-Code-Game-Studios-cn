# 技能测试规范：/scope-check

## 技能摘要

`/scope-check` 是一个 Haiku 层级的只读技能，用于将功能、冲刺或里程碑的原始计划范围与实际已实现的内容进行对比。它接收完整参数字符串——功能名称（可能包含多个单词）、冲刺编号或里程碑名称——并在 `design/gdd/[feature].md`（或 `design/` 中的匹配文件）、`production/sprints/sprint-NNN.md` 或 `production/milestones/[name].md` 中定位基线。若基线缺失，则报告缺失文件并停止。随后读取当前状态（相关源文件、`git log`、TODO/FIXME 注释、当前冲刺计划），生成对比报告：原始范围、当前范围、范围新增、范围移除、膨胀评分、风险评估以及削减/推迟/保留/标记建议。结论依据净范围变化：≤10% 通过，>10–25% 需关注，>25% 失败。当百分比无意义时——基线未列举任何条目，或当前状态无法读取——结论为未评估，并替换数值块。不写入任何文件，不调用总监门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含结论关键词：PASS、CONCERNS、NOT ASSESSED、FAIL
- [ ] 不要求"我可以写入"之类的措辞（技能声明为只读，且 `allowed-tools` 不含 Write/Edit）
- [ ] 具有下一步交接（阶段 5：根据结论进行后续操作）

---

## 总监门禁检查

无。范围检查是只读咨询技能，不调用任何门禁。

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 均设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——多词功能按计划推进

**夹具：**
- `design/gdd/inventory-crafting-system.md` 列举了 20 个范围条目
- `src/gameplay/inventory/` 和 `src/gameplay/crafting/` 实现了全部 20 个条目，外加一个在近期提交中发现的新增项（物品排序）
- 没有条目被丢弃

**输入：** `/scope-check inventory crafting system`

**预期行为：**
1. 技能使用完整参数 "inventory crafting system" 定位基线——而非仅使用 "inventory"
2. 技能读取 `design/gdd/inventory-crafting-system.md` 作为基线（阶段 1）
3. 技能读取当前状态：相关源文件、`git log --oneline` 中的工作记录、TODO/FIXME 注释（阶段 2）
4. 报告列出原始范围和当前范围；范围新增表列出物品排序及其来源、日期、理由和工作量
5. 膨胀评分：原始 20，当前 21，新增 1（+5%），移除 0
6. 结论：**PASS**——按计划推进
7. 阶段 5：无需操作；建议在下一个里程碑前重新运行；以"在做出削减后再次运行 `/scope-check [name]` 以验证结论改善"结束

**断言：**
- [ ] 基线从完整的多词参数定位，而非其第一个词
- [ ] 新增项在范围新增表中列出，包含来源、时间、合理性及工作量列
- [ ] 膨胀评分显示原始、当前、新增和移除数量及净百分比
- [ ] 净变化 ≤10% 时结论为 PASS
- [ ] 不写入任何文件

---

### 用例 2：显著蔓延——冲刺增加了计划外条目

**夹具：**
- `production/sprints/sprint-003.md` 列出了 8 个计划故事
- 冲刺开始以来的提交增加了在线排行榜、成就系统和照片模式，均未出现在冲刺计划中
- 没有计划故事被丢弃

**输入：** `/scope-check sprint-3`

**预期行为：**
1. 技能读取 `production/sprints/sprint-003.md` 作为基线
2. 技能从 `git log` 和源码树中发现三个新增项
3. 范围新增表列出全部三项；膨胀评分显示原始 8，当前 11，+37.5%
4. 建议将条目分类为削减/推迟/保留/标记
5. 结论：**FAIL**——显著蔓延
6. 阶段 5：建议上报给制作人，并引用 `/sprint-plan update` 进行重新规划或 `/estimate` 重新建立基线

**断言：**
- [ ] 每个计划外新增项在范围新增表中明确列出
- [ ] 净变化在 25% 到 50% 之间时结论为 FAIL
- [ ] 输出建议上报给制作人并引用 `/sprint-plan update` 或 `/estimate`
- [ ] 技能不编辑冲刺计划或移除故事——发现仅为建议

---

### 用例 3：基线缺失——报告缺失文件并停止

**夹具：**
- 无 `design/gdd/crafting.md`，且 `design/` 中无文件匹配 "crafting"
- `src/gameplay/crafting/` 包含已实现的代码

**输入：** `/scope-check crafting`

**预期行为：**
1. 阶段 1 查找 `design/gdd/crafting.md` 及 `design/` 中的匹配文件；均不存在
2. 技能报告缺失的基线文件并停止
3. 不生成对比报告、膨胀评分或百分比

**断言：**
- [ ] 输出列出其查找但未找到的基线文件
- [ ] 技能在阶段 1 停止——不渲染阶段 3 的对比报告
- [ ] 无基线时不产生百分比，也不产生 PASS 结论
- [ ] 不写入任何文件

---

### 用例 4：空基线——未评估，而非 0%

**夹具：**
- `design/gdd/dialogue.md` 存在，但仅包含章节标题和 `[TO BE CONFIGURED]` 占位符——无范围条目
- `src/narrative/dialogue/` 包含已实现的代码

**输入：** `/scope-check dialogue`

**预期行为：**
1. 阶段 1 找到基线文档，因此不在此停止
2. 阶段 4 检测到基线未列举任何范围条目
3. 膨胀评分块替换为 `Baseline unusable — see verdict` 及原因；不渲染 `Original items: 0` 或 `Net scope change: 0%`
4. 结论：**NOT ASSESSED**，说明基线侧无法读取及原因
5. 阶段 5：说明需要填充基线文档；不提供针对相同输入的重新运行

**断言：**
- [ ] 基线无条目时结论为 NOT ASSESSED，绝不为 PASS
- [ ] 报告中任何位置均不出现计算出的百分比
- [ ] 输出说明哪一侧不可用以及如何修复（填充基线文档）
- [ ] 技能不提供针对相同输入的重新运行

---

### 用例 4b：当前状态不可读——未评估，而非 −100%

**夹具：**
- `production/sprints/sprint-004.md` 列出了 6 个计划故事
- 无源文件与这些故事相关，`git log` 显示冲刺开始以来无提交，且无任何标记为进行中

**输入：** `/scope-check sprint-4`

**预期行为：**
1. 阶段 1 读取 `production/sprints/sprint-004.md` 作为基线——6 个条目
2. 阶段 2 找不到可读取的当前状态
3. 阶段 4 输出 NOT ASSESSED 而非计算变化：当前 0 个条目会计算出 −100%，而 ≤10% 行会将其读作 PASS
4. 膨胀评分块替换为 `Current state unusable — see verdict`；不渲染 `Current items: 0` 或百分比
5. 结论说明当前状态侧无法读取及原因，阶段 5 说明如何修复（将技能指向工作所在位置），但不提供针对相同输入的重新运行

**断言：**
- [ ] 结论为 NOT ASSESSED，绝不为 PASS
- [ ] 报告中不出现 −100%（或任何百分比）
- [ ] 输出将当前状态列为无法读取的一侧
- [ ] 技能不提供针对相同输入的重新运行

---

### 用例 5：完整审查模式下的轻微蔓延——需关注，无门禁

**夹具：**
- `project.yaml` 具有 `modes.review_mode: full`
- `production/milestones/alpha.md` 列举了 10 个范围条目
- 当前状态包含全部 10 个条目外加两个新增项（设置菜单重做和额外敌人类型）

**输入：** `/scope-check alpha`

**预期行为：**
1. 技能读取 `production/milestones/alpha.md` 作为基线
2. 膨胀评分：原始 10，当前 12，+20%
3. 结论：**CONCERNS**——轻微蔓延
4. 无论审查模式如何，均不调用总监门禁
5. 阶段 5：提议识别削减比例最佳的 2–3 个新增项，并引用 `/sprint-plan update`
6. 技能结束时不写入任何文件

**断言：**
- [ ] 净变化在 10% 到 25% 之间时结论为 CONCERNS
- [ ] 任何审查模式下均不调用总监门禁
- [ ] 输出提议识别削减比例最佳的新增项并引用 `/sprint-plan update`
- [ ] 不写入任何文件

---

## 协议合规性

- [ ] 在任何对比之前定位基线文档（`design/gdd/`、`production/sprints/`、`production/milestones/`），若缺失则停止
- [ ] 从源文件、`git log`、TODO/FIXME 注释和当前冲刺计划读取当前状态
- [ ] 结论遵循净变化表：≤10% PASS，>10–25% CONCERNS，>25% FAIL
- [ ] 当基线无条目或当前状态无法读取时，输出 NOT ASSESSED 而非百分比
- [ ] 不写入任何文件
- [ ] 不调用任何总监门禁
- [ ] 声明 `model: haiku`（当使用声明的层级时：`.claude/docs/model-tiers.md`）

---

## 覆盖说明

- >50% 的"失控"区间与用例 2 共享 FAIL 结论，不单独测试。