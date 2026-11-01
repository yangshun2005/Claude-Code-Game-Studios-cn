# 技能测试规范：/team-live-ops

## 技能摘要

通过 7 阶段规划流水线编排 live-ops 团队，产出赛季或活动计划。协调 live-ops-designer、economy-designer、analytics-engineer、community-manager、narrative-director 和 writer。阶段 0 解析 `review_mode`、`automation`、`team.size` 和 `workflow`，技能在阶段 1 之前公布激活集——`individual` 生成 live-ops-designer + economy-designer，`small` 增加 analytics-engineer + community-manager，`studio` 增加 writer + narrative-director。阶段 3 和阶段 4（经济设计与分析）同时运行。阶段 7 对照 `design/live-ops/ethics-policy.md` 检查经济设计。子代理在 `production/` 下写入工作产物；编排器编译最终文档，仅在通过 `AskUserQuestion` 询问"我可以将赛季计划写入 `design/live-ops/…` 吗？"并获得批准后，才将其写入 `design/live-ops/seasons/`。最终产出整合的赛季计划，需用户批准后方可移交生产。

---

## 静态断言（结构性）

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 输出文档部分：编排器在写入前通过 `AskUserQuestion` 询问"我可以将赛季计划写入 `design/live-ops/…` 吗？"，因为 `design/` 不在受限写入例外范围内
- [ ] 具有文件写入协议部分，将子代理写入限制在受限例外范围内（指定路径、`production/`、`docs/` 或 `tests/` 下的新产物、受门控的阶段）——子代理在这些边界之外需询问
- [ ] 为每个代理在 `production/live-ops/[season-slug]/` 下指定目标文件——`brief.md`、`narrative.md`、`economy.md`、`analytics.md`、`narrative-text.md`、`copy.md`、`comms.md`——每个代理一个文件
- [ ] 将阶段门控定义为流水线自身列出的决策点，无论 `automation` 模式如何，且不在阶段门控处声称生成总监
- [ ] 结尾具有下一步移交，引用 `/design-review`、`/sprint-plan` 和 `/team-release`
- [ ] 在阶段转换处使用 `AskUserQuestion` 捕获用户批准后再继续（`collaborative` 模式）
- [ ] 阶段 0 解析 `team.size`，技能要求在阶段 1 之前进行一行激活集公告
- [ ] 明确声明阶段 3 和阶段 4 可以同时运行（并行生成）
- [ ] 存在错误恢复部分：验证指定产物在磁盘上，立即暴露 BLOCKED，始终产出部分报告，若无法解决则以 BLOCKED 结束
- [ ] 输出文档部分指定 `design/live-ops/seasons/` 下的路径

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——全部 7 个阶段完成，产出赛季计划

**夹具：**
- 已解析配置块：`team.size: studio`、`automation: collaborative`、`workflow: standard`（固定：未设置时解析为 minimal，其下一步使用简报的构建顺序而非 `/sprint-plan`；用例 6 覆盖该情况）
- `design/live-ops/economy-rules.md` 存在，包含当前经济配置
- `design/live-ops/ethics-policy.md` 存在，包含项目伦理政策
- 游戏概念文档存在于其标准路径
- 正在规划的新赛季名称没有现有赛季文档

**输入：** `/team-live-ops "Season 2: The Frozen Wastes"`

**预期行为：**
1. 在任何生成之前，一行公告 `team.size: studio` 的激活集
2. 阶段 1：通过 Agent 生成 `live-ops-designer`；接收包含范围、内容列表和留存机制的赛季简报；呈现给用户
3. AskUserQuestion：用户在阶段 2 开始前批准阶段 1 输出
4. 阶段 2：通过 Agent 生成 `narrative-director`；读取阶段 1 赛季简报；产出叙事框架文档（主题、故事钩子、背景故事关联）；呈现给用户
5. 阶段 3 和阶段 4（并行）：在等待任一结果之前，通过两次 Agent 调用同时生成 `economy-designer` 和 `analytics-engineer`；economy-designer 读取 `design/live-ops/economy-rules.md`
6. 阶段 5：并行生成 `narrative-director` 和 `writer`，产出游戏内叙事文本和面向玩家的文案；两者都读取阶段 2 叙事框架文档
7. 阶段 6：通过 Agent 生成 `community-manager`；读取赛季简报、经济设计和叙事框架；产出包含草稿文案的沟通日历
8. 阶段 7：收集所有阶段输出；呈现整合的赛季计划摘要，包括经济健康检查、分析就绪度、对照 `design/live-ops/ethics-policy.md` 的伦理审查以及待解决问题
9. AskUserQuestion：用户批准完整赛季计划
10. 编排器编译三份文档，并通过 AskUserQuestion 询问"我可以将赛季计划写入 `design/live-ops/seasons/…` 吗？"，涵盖 `S2_[name].md`、`S2_[name]_analytics.md` 和 `S2_[name]_comms.md`；仅在批准后写入
11. 判定：COMPLETE——赛季计划已产出并移交生产

**断言：**
- [ ] 在生成第一个代理之前出现命名 `team.size: studio` 的激活集行
- [ ] 全部 7 个阶段按顺序执行；阶段 3 和阶段 4 作为并行 Agent 调用发出
- [ ] 阶段 7 整合摘要包含全部六个部分（赛季简报、叙事框架、经济设计、分析计划、内容清单、沟通日历）
- [ ] 阶段 7 的伦理审查部分明确引用 `design/live-ops/ethics-policy.md`
- [ ] 三份输出文档写入 `design/live-ops/seasons/`，采用 `S[N]_[name].md`、`S[N]_[name]_analytics.md`、`S[N]_[name]_comms.md` 命名约定
- [ ] `design/live-ops/` 文档仅由编排器在其 `AskUserQuestion` 批准后写入——没有子代理在 `design/` 下写入
- [ ] 最终输出中出现判定：COMPLETE，且仅在用户批准赛季计划之后
- [ ] 下一步引用 `/design-review`、`/sprint-plan` 和 `/team-release`

---

### 用例 2：发现伦理违规——奖励元素违反伦理政策

**夹具：**
- 所有标准 live-ops 夹具存在（economy-rules.md、ethics-policy.md）
- `design/live-ops/ethics-policy.md` 包含规则"没有公布掉落率和保底机制的随机付费奖励"
- economy-designer（阶段 3）提出"Mystery Chest"机制，具有随机付费奖励、未公布掉落率且无保底机制

**输入：** `/team-live-ops "Season 3: Shadow Tournament"`

**预期行为：**
1. 阶段 1–4 正常进行；economy-designer 提出 Mystery Chest 机制
2. 阶段 7：编排器对照伦理政策审查阶段 3 输出；识别 Mystery Chest 违反政策的"没有公布掉落率和保底机制的随机付费奖励"规则
3. 阶段 7 摘要的伦理审查部分明确标记违规："伦理标记：阶段 3 经济设计中的 Mystery Chest 机制违反'没有公布掉落率和保底机制的随机付费奖励'。在解决之前，批准被阻止。"
4. 在提供赛季计划批准之前，通过 AskUserQuestion 呈现解决选项
5. 在伦理违规解决或用户明确豁免之前，技能不发出 COMPLETE 判定，也不写入输出文档

**断言：**
- [ ] 阶段 7 伦理审查部分明确命名违规元素及其违反的政策规则
- [ ] 存在伦理违规时，技能不自动批准赛季计划
- [ ] 使用 AskUserQuestion 暴露违规并提供解决选项（修改经济设计、附书面理由覆盖、取消）
- [ ] 违规未解决时不写入输出文档
- [ ] 若用户选择修改：技能重新生成 economy-designer 以产出修正后的设计，然后返回阶段 7 审查
- [ ] 仅在伦理标记清除后才发出判定：COMPLETE；取消以判定：BLOCKED 结束

---

### 用例 3：无参数——显示用法指引

**夹具：**
- 任意项目状态

**输入：** `/team-live-ops`（无参数）

**预期行为：**
1. 检测到无参数
2. 输出："用法：`/team-live-ops [赛季名称或活动描述] [--review full|lean|solo]`——提供要规划的赛季或 live 活动的名称或描述。"
3. 技能立即退出，不生成任何子代理

**断言：**
- [ ] 技能不猜测赛季名称或捏造范围
- [ ] 用法消息显示完整 argument-hint：`/team-live-ops [赛季名称或活动描述] [--review full|lean|solo]`
- [ ] 在参数检查失败之前不发出任何 Agent 调用
- [ ] 不读取或写入任何文件

---

### 用例 4：并行阶段验证——阶段 3 和阶段 4 同时运行

**夹具：**
- 已解析配置块：`team.size: small`、`automation: collaborative`
- 所有标准 live-ops 夹具存在
- 阶段 1（赛季简报）和阶段 2（叙事框架）已批准
- 阶段 3（economy-designer）和阶段 4（analytics-engineer）的输入相互独立

**输入：** `/team-live-ops "Season 1: The First Thaw"`（在阶段 3/4 转换处观察）

**预期行为：**
1. 阶段 2 获用户批准后，编排器在等待任一结果之前发出两次 Agent 调用（economy-designer 和 analytics-engineer）
2. 两个代理都接收赛季简报作为上下文；analytics-engineer 不等待 economy-designer 输出才开始
3. economy-designer 输出和 analytics-engineer 输出在阶段 5 开始前一起收集
4. 若两个并行代理之一阻塞，另一个继续；报告部分结果

**断言：**
- [ ] 阶段 3 和阶段 4 的两次 Agent 调用在等待任一结果之前发出——它们不是顺序的
- [ ] analytics-engineer 提示不包含 economy-designer 输出作为必需输入（输入相互独立）
- [ ] 若 economy-designer 阻塞但 analytics-engineer 成功，分析输出被保留，阻塞通过 AskUserQuestion 暴露
- [ ] 阶段 5 在收集完阶段 3 和阶段 4 两个结果之前不开始
- [ ] 技能文档明确声明"阶段 3 和阶段 4 可以同时运行"

---

### 用例 5：缺少伦理政策——`design/live-ops/ethics-policy.md` 不存在

**夹具：**
- `design/live-ops/economy-rules.md` 存在
- `design/live-ops/ethics-policy.md` 不存在
- 所有其他夹具存在

**输入：** `/team-live-ops "Season 4: Desert Heat"`

**预期行为：**
1. 阶段 1–6 正常进行
2. 阶段 7：编排器尝试运行伦理审查；检测到 `design/live-ops/ethics-policy.md` 缺失
3. 阶段 7 摘要包含缺口标记："伦理审查已跳过：未找到 `design/live-ops/ethics-policy.md`。经济设计未对照伦理政策审查。建议在生产开始前创建一份。"
4. 技能仍完成赛季计划并达到 COMPLETE 判定，但缺口在输出和赛季设计文档中显著标记
5. 下一步包括创建伦理政策文档的建议

**断言：**
- [ ] 伦理政策文件缺失时技能不报错
- [ ] 文件缺失时技能不捏造伦理政策规则
- [ ] 阶段 7 摘要明确注明伦理审查被跳过及其原因
- [ ] 尽管文件缺失，仍可达到判定：COMPLETE
- [ ] 缺口标记出现在赛季设计输出文档中（不仅在对话中）
- [ ] 下一步建议创建 `design/live-ops/ethics-policy.md`

---

### 用例 6：工作流 Minimal——冲刺步骤变为简报的构建顺序

**夹具：**
- 已解析配置块：`workflow: minimal`（来自 `modes.rigor: minimal`）、`team.size: studio`（固定以便完整流水线运行）、`automation: collaborative`
- `design/game-brief.md` 存在并包含构建顺序；没有 `production/sprints/` 文件夹
- `design/live-ops/economy-rules.md` 和 `design/live-ops/ethics-policy.md` 存在，且经济设计未引发伦理标记

**输入：** `/team-live-ops "Season 5: Harvest Moon"`

**预期行为：**
1. 阶段 0 解析 `workflow` 以及 `review_mode`、`automation` 和 `team.size`
2. 流水线如用例 1 运行，并在用户批准赛季计划后以判定：COMPLETE 结束
3. 下一步：对赛季设计文档运行 `/design-review`；不运行 `/sprint-plan`，而是将赛季的内容工作添加到简报的构建顺序中，因为 `minimal` 没有冲刺；内容就绪后运行 `/team-release`

**断言：**
- [ ] `workflow` 是阶段 0 解析的键之一
- [ ] 调度步骤说明将赛季的内容工作添加到简报的构建顺序中
- [ ] 不推荐 `/sprint-plan`
- [ ] 仍列出 `/design-review` 和 `/team-release`
- [ ] 运行的其余部分（激活集行、并行阶段 3 和阶段 4、伦理审查、`design/live-ops/seasons/` 文档前的"我可以写入吗"）不受 `workflow` 影响

---

## 协议合规性

- [ ] 在默认 `team.size: individual` 下，运行以 `Active set (team.size: individual): live-ops-designer + economy-designer` 行和 `Not spawned this run:` 行开头，命名 analytics-engineer、community-manager、narrative-director 和 writer——在任何代理生成之前
- [ ] 在 `collaborative` 模式下，每个阶段转换处使用 `AskUserQuestion`——用户在下个阶段开始前批准
- [ ] 当 analytics-engineer 在激活集中（`small` 或 `studio`）时，阶段 3 和阶段 4 并行生成，而非顺序
- [ ] 子代理仅写入编排器指定的路径，在受限例外范围内；`design/live-ops/` 文档由编排器在一次 `AskUserQuestion`"我可以写入赛季计划……"批准后写入
- [ ] 指定产物在磁盘上缺失的阶段被视为失败，而非完成
- [ ] 阶段 7 的伦理审查始终明确引用伦理政策文件路径
- [ ] 错误恢复：任何 BLOCKED 代理立即通过 AskUserQuestion 选项（跳过 / 重试 / 停止）暴露
- [ ] 若任何阶段阻塞则产出部分报告——工作绝不丢弃
- [ ] 判定：COMPLETE 仅在用户批准整合的赛季计划后；若存在未解决的伦理违规则为 BLOCKED
- [ ] COMPLETE 计划后的下一步包括 `/design-review`、`/sprint-plan`（在 `workflow: minimal` 下，以简报的构建顺序替代）和 `/team-release`；因伦理标记以 BLOCKED 结束的运行则改为命名修订和重新运行

---

## 覆盖说明

- 阶段 5 并行生成（narrative-director + writer）遵循与阶段 3/4 相同的模式，但此处未单独测试——它使用用例 4 中验证的相同并行 Agent 协议。
- "economy-rules.md 缺失"边缘情况未单独测试——它会表现为 economy-designer 的 BLOCKED 结果，并遵循用例 4 中隐式测试的标准错误恢复路径。
- 完整内容写入流水线（阶段 5 输出验证）由用例 1 正常路径整合摘要检查隐式验证。
- 社区经理沟通日历格式（发布前、发布日、赛季中期、最后一周）由用例 1 隐式验证；无需单独边缘情况。
- 用例 2 和用例 5 仅依赖 economy-designer 和阶段 7 审查，两者在每个 `team.size` 下都运行。