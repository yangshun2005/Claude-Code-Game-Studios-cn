# 技能测试规范：/balance-check

## 技能摘要

`/balance-check [system-name|path-to-data-file]` 识别平衡域（战斗、经济、成长、掉落，或直接给定的数据文件），从 `assets/data/` 和 `design/balance/` 读取该域的数据文件，然后读取目标值——优先从 `design/registry/entities.yaml`（其 `constants:` 和 `formulas:` 块）读取，注册表未提供的则从 `design/gdd/` 中该系统的 GDD 读取。它将每个数据值与其目标比较——注册表常量的 `value`、注册表公式的 `output_range`，或 GDD 的范围——然后运行域特定检查，并打印报告，包含已分析数据源、健康摘要、检测到的异常值表（条目/值 | 预期范围 | 实际值 | 问题）、退化策略、成长分析、按优先级排序的建议表和需关注的值。找不到的输入会使受影响的部分标记为 `NOT ASSESSED — NO DATA`，并指明缺失的来源和 `/design-system`，绝不估算；没有目标的值列为 `no stated range — not judged`。报告之后，`AskUserQuestion` 提供 [A] 立即修复、[B] 将报告保存到 `design/balance/balance-check-[system]-[date].md`、[C] 停止；仅在 [B] 时写入。不调用任何总监门禁。健康摘要，首个匹配优先：CRITICAL ISSUES、CONCERNS、NOT ASSESSED、HEALTHY。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：HEALTHY、CONCERNS、CRITICAL ISSUES、NOT ASSESSED
- [ ] 具有写入前询问的语言：保存是一个 `AskUserQuestion` 选项，指明其目标路径，且仅当选择该选项时才执行 Write
- [ ] 具有下一步交接（审查发现后该做什么）

---

## 总监门禁检查

无。平衡检查是分析技能；任何审查模式下都不调用门禁。

---

## 夹具默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身夹具另有说明。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有平衡值都在预期范围内

**夹具：**
- `assets/data/combat-balance.json` 存在，含 6 个属性值
- `design/registry/entities.yaml` 无条目（随附的空存根）
- `design/gdd/combat-system.md` 为全部 6 个属性给出预期范围
- 全部 6 个值都在其范围内；不存在退化策略

**输入：** `/balance-check combat`

**预期行为：**
1. 技能从参数识别出战斗域
2. 技能读取 `assets/data/combat-balance.json`（以及 `design/balance/` 中的任何战斗文件）
3. 注册表无条目，因此技能仅使用 GDD 获取预期范围
4. 6 个值各自与 GDD 范围比较，均未超出；战斗检查未发现主导选项
5. 报告在已分析数据源下列出 `assets/data/combat-balance.json` 和 `design/gdd/combat-system.md`；检测到的异常值表无行
6. 健康摘要为 HEALTHY
7. 阶段 6 的 `AskUserQuestion` 提供 [A] / [B] / [C]；用户选择前不写入任何内容

**断言：**
- [ ] 已分析数据源列出每个读取的文件，包括数据文件和 GDD
- [ ] 跳过空注册表，由 GDD 提供预期范围
- [ ] 检测到的异常值表无行
- [ ] 健康摘要为 HEALTHY
- [ ] 用户选择选项 [B] 前不写入任何文件

---

### 用例 2：异常值——玩家伤害远高于其预期范围

**夹具：**
- `assets/data/combat-balance.json` 含 `player_damage_base: 140`
- `design/gdd/combat-system.md` 为 `player_damage_base` 给出预期范围 90–110
- 所有其他属性都在范围内

**输入：** `/balance-check combat`

**预期行为：**
1. 技能读取数据文件和 `player_damage_base` 的 GDD 范围，并将每个值与其范围比较
2. 检测到的异常值表列出 `player_damage_base`，预期范围 90–110，实际值 140，并附问题描述
3. 建议表为其提供一行，含优先级、建议修复和影响
4. 需关注的值给出带理由的建议调整
5. 健康摘要为 CONCERNS——一个异常值可通过一次调参修复，无退化策略、成长停滞或无消耗循环
6. 若用户选择 [A] 并修改战斗 GDD 定义的 `player_damage_base`，技能提醒他们在提交前对该 GDD 运行 `/propagate-design-change`

**断言：**
- [ ] `player_damage_base` 出现在检测到的异常值表中，预期范围 90–110，实际值 140
- [ ] 建议表按优先级对修复排序
- [ ] 健康摘要为 CONCERNS——不是 HEALTHY，且在没有关键类发现时不是 CRITICAL ISSUES
- [ ] 选择 [A] 时，修改 GDD 定义的值后跟随 `/propagate-design-change` 提醒

---

### 用例 3：无设计目标——数据存在但无 GDD 或注册表定义范围

**夹具：**
- `assets/data/economy-balance.yaml` 存在，含 10 个普通属性值（价格和比率，仅凭数据无法显示水龙头/水槽循环）
- `design/registry/entities.yaml` 无条目
- `design/gdd/` 中无 GDD 覆盖经济

**输入：** `/balance-check economy`

**预期行为：**
1. 技能将输入记录为 FOUND / ABSENT：数据文件为 FOUND；注册表条目和经济 GDD 为 ABSENT
2. 技能读取数据文件并在已分析数据源下列出
3. 需要预期值的部分（检测到的异常值，以及需要目标的成长分析）标记为 `NOT ASSESSED — NO DATA`，指明缺失的经济 GDD 和空注册表，并指明 `/design-system` 为写入它们的技能
4. 不虚构预期范围，也不从数据本身推断
5. 健康摘要为 NOT ASSESSED——报告说明这些值无法判断，而非它们没问题

**断言：**
- [ ] 当注册表和 GDD 都不提供预期范围时，技能不虚构预期范围
- [ ] 受影响的部分标记为 `NOT ASSESSED — NO DATA`，而非显示为空表
- [ ] 输出指明缺失的设计来源（`design/gdd/` 中的经济 GDD、空的 `design/registry/entities.yaml`）和 `/design-system`
- [ ] 健康摘要为 NOT ASSESSED——不是 HEALTHY，也不是 CONCERNS，因为未发现任何错误

---

### 用例 4：注册表优先——注册表值覆盖矛盾的 GDD

**夹具：**
- `design/registry/entities.yaml` 的 `constants:` 列出 `player_damage_base`，含 `value: 100`、`unit: damage`、`source: design/gdd/combat-system.md`（注册表常量携带值，而非范围）
- `design/gdd/combat-system.md` 仍称 `player_damage_base` 范围 100–130（过时）
- `assets/data/combat-balance.json` 含 `player_damage_base: 125`

**输入：** `/balance-check combat`

**预期行为：**
1. 技能在读取 GDD 前先 grep `design/registry/entities.yaml`（`^  - name:` 带上下文）
2. `player_damage_base` 的目标是注册表常量的 `value`，即 100，而非 GDD 的范围
3. 125 与注册表值 100 不同，因此 `player_damage_base` 出现在检测到的异常值表中，预期范围 100（注册表值），实际值 125——尽管它位于过时的 GDD 范围内
4. 注册表未列出的任何值仍从 GDD 读取
5. 已分析数据源列出注册表、GDD 和数据文件

**断言：**
- [ ] 注册表在 GDD 之前读取
- [ ] `player_damage_base` 对照注册表值（100）判断，而非 GDD 范围（100–130）
- [ ] `player_damage_base: 125` 报告为异常值
- [ ] 已分析数据源包含 `design/registry/entities.yaml`

---

### 用例 5：无参数、门禁合规和保存路径

**夹具：**
- 战斗数据、注册表和 GDD 存在；1 个属性略微超出其范围
- `project.yaml`：`modes.review_mode: full`
- `modes.automation` 解析为 `collaborative`

**输入：** `/balance-check`（无参数）

**预期行为：**
1. 技能询问用户要检查哪个系统；用户回答 "combat"
2. 技能读取数据和设计目标并打印报告
3. 不调用任何总监门禁
4. 报告之后，`AskUserQuestion` 提供 [A] 立即修复、[B] 保存到 `design/balance/balance-check-combat-[date].md`、[C] 停止
5. 选择 [B]：报告写入 `design/balance/balance-check-combat-[YYYY-MM-DD].md`（如需要则创建目录），确认写入，技能以 "Re-run `/balance-check` after fixes to verify." 结束
6. 选择 [C]：不写入任何内容；总结未解决问题，并以同一重运行行结束输出

**断言：**
- [ ] 无参数时，技能在读取数据前询问要检查哪个系统
- [ ] 任何审查模式下都不调用总监门禁
- [ ] 报告在保存选项之前呈现，保存选项指明 `design/balance/balance-check-[system]-[date].md`
- [ ] 选项 [B] 写入该路径，日期为 YYYY-MM-DD 格式；选项 [C] 不写入任何内容
- [ ] [B] 和 [C] 都以 "Re-run `/balance-check` after fixes to verify." 结束

---

### 用例 6：孤立属性——无 GDD 或注册表定义的数据值

**夹具：**
- `assets/data/combat-balance.json` 含 6 个属性外加 `legacy_armor_mult: 1.5`
- `design/registry/entities.yaml` 无条目
- `design/gdd/combat-system.md` 为 6 个属性给出范围，对 `legacy_armor_mult` 无任何说明
- 6 个属性都在其范围内；不存在退化策略

**输入：** `/balance-check combat`

**预期行为：**
1. 每个值都与其目标比较；6 个属性在范围内
2. `legacy_armor_mult` 无目标，因此在需关注的值下列为 `no stated range — not judged`
3. 它不是检测到的异常值行，也不从报告中丢弃
4. 健康摘要为 NOT ASSESSED，指明 `legacy_armor_mult` 为无法判断的值

**断言：**
- [ ] `legacy_armor_mult` 报告为 `no stated range — not judged`，与任何异常值分开
- [ ] 孤立属性不被静默跳过，也不对照从数据推断的范围判断
- [ ] 健康摘要为 NOT ASSESSED，而非 HEALTHY，并指明未判断的值

---

## 协议合规

- [ ] 在分析前读取该域的数据文件（阶段 2）以及注册表然后 GDD（阶段 3）
- [ ] 将每个数据值与其目标比较（注册表常量 `value`、注册表公式 `output_range` 或 GDD 范围）；超出者为检测到的异常值行，无目标者为 `no stated range — not judged`
- [ ] 检测到的异常值表使用列 条目/值、预期范围、实际值、问题；建议表使用 优先级、问题、建议修复、影响
- [ ] 输入缺失的部分为 `NOT ASSESSED — NO DATA`，绝不估算，并指明缺失的来源和 `/design-system`
- [ ] 仅在用户选择保存选项时写入，且仅写入 `design/balance/balance-check-[system]-[date].md`
- [ ] 不调用任何总监门禁
- [ ] 健康摘要，首个匹配：CRITICAL ISSUES、CONCERNS、NOT ASSESSED、HEALTHY——仅当每个值都对照声明的目标判断过才为 HEALTHY

---

## 覆盖说明

- 所有输入都缺失的情况（无数据文件、无注册表条目、无 GDD）未单独测试；此时技能停止并报告 `NOT ASSESSED — NO DATA` 作为整体判定，指明缺失了什么以及哪个技能产生它。
- CRITICAL ISSUES（退化策略、成长停滞、无消耗循环）未单独设用例；用例 2 确认仅一个异常值即为 CONCERNS。
- 数据文件路径参数（阶段 1 "File path given"）未测试；此时域从文件内容推断。