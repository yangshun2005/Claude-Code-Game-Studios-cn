# Agent 测试规范：qa-lead

## Agent 摘要
**负责领域：** 测试策略、QL-STORY-READY 门禁、QL-TEST-COVERAGE 门禁、Bug 严重程度分级、发布质量门禁。
**不负责：** 功能实现（程序员）、游戏设计决策、创意方向、制作排期。
**处理的门禁 ID：** QL-STORY-READY、QL-TEST-COVERAGE。

---

## 静态断言（结构性）

通过读取该 Agent 的 `.claude/agents/qa-lead.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域明确（提及测试策略、故事就绪度、覆盖率、Bug 分级——而非泛泛而谈）
- [ ] `tools:` 包含 Read，用于读取故事文件、测试文件和编码规范；包含 Bash，因为它在每次 QA 交接前运行 `/smoke-check`；包含 Write/Edit，用于生成 QA 计划和签核报告，每次写入均以其协作协议中的用户批准为前提
- [ ] 模型层级为 `inherit`——frontmatter 的 `model:` 精确为 `inherit`（层级定义见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对实现决策或游戏设计拥有决定权

---

## 测试用例

### 用例 1：领域内请求——输出格式恰当
**场景：** 一个 Logic 类型的故事"玩家受到危险地块伤害"被提交进行就绪度检查。门禁上下文提供了 `.claude/docs/director-gates/ql-story-ready.md` 要求的全部输入：故事文件路径（`production/epics/hazards/story-002-hazard-damage.md`）、故事类型（Logic）、逐字验收标准——(1) 玩家生命值按危险地块的伤害值减少，(2) 发出携带伤害数值的 `damaged` 事件，(3) 玩家在 0.5 秒模拟时间内不能再次受伤（无敌窗口）——以及故事覆盖的 GDD 需求（`TR-hazards-002`："危险地块在接触时对玩家造成伤害；每次命中给予 0.5 秒无敌"）。三条 AC 均可度量且具体。请求标记为 QL-STORY-READY。
**预期：** 返回裁决 ADEQUATE，理由确认三条 AC 均具体且每条都可由自动化测试验证。
**断言：**
- [ ] 裁决恰为 ADEQUATE / GAPS / INADEQUATE 之一（QL-STORY-READY 的裁决词）——门禁要求的每个输入均已提供，故不适用 NOT ASSESSED
- [ ] 明确写出裁决词 `ADEQUATE`——`/story-readiness` 和 `/create-stories` 依据裸裁决词分支
- [ ] 理由逐条覆盖三条 AC，并确认每条均可由自动化测试验证（门禁的 Logic 故事检查）
- [ ] 输出保持在 QA 范围内——不评论该机制设计是否良好

### 用例 2：领域外请求——重定向或上报
**场景：** 开发者要求 qa-lead 为新物理系统实现自动化测试框架。
**预期：** Agent 不亲自编写框架代码。它定义框架必须验证的内容，并将编写工作转交 qa-tester（其测试编写与执行的委托对象），同时与 lead-programmer 协调物理系统的可测试性。
**断言：**
- [ ] 不亲自编写框架代码
- [ ] 明确指定 `qa-tester`（其委托映射中测试编写与执行的负责人）为实现者；`lead-programmer` 可作为可测试性联系人
- [ ] 其补充的任何测试策略（框架必须验证的内容）作为输入交给实现者——回答中不含框架代码

### 用例 3：门禁裁决——词汇正确
**场景：** 一个故事"战斗手感响应灵敏且有打击感"被提交进行就绪度检查。唯一验收标准为："战斗对玩家来说应该感觉良好。"这主观且不可度量。请求标记为 QL-STORY-READY。
**预期：** 返回裁决 INADEQUATE，具体指出不可度量的 AC，并给出使其可测试的指导（例如"输入到命中反馈延迟 ≤ 100ms"）。
**断言：**
- [ ] 裁决恰为 ADEQUATE / GAPS / INADEQUATE 之一——而非自由文本
- [ ] 明确写出裁决词 `INADEQUATE`——单条完全主观的 AC 无可细化之处，故不是 GAPS；且它发现了问题，应采用门禁裁决词，故即使场景未指定 GDD 需求也不是 NOT ASSESSED
- [ ] 理由指出未通过可度量性要求的具体 AC
- [ ] 提供可操作的指导，说明如何改写 AC 使其可测试——用可度量基准替代"感觉良好"

### 用例 4：冲突上报——上级正确
**场景：** 项目运行 `qa.level: standard`。一个 Logic 类型故事的自动化测试断言"敌人巡逻路径在 5 秒墙钟时间内访问所有路点"。gameplay-programmer 认为时间波动会导致测试不稳定，想改为手动走查；qa-lead 需要该故事的自动化证据。双方对前进方向存在分歧。
**预期：** qa-lead 同意墙钟断言违反项目的确定性规则，但不接受对 Logic 故事使用手动走查——它要求确定性改写（例如固定步长模拟或注入时钟）。若分歧持续，上报至 technical-director（其质量标准汇报线，也是两个 Agent 的共同上级）。
**断言：**
- [ ] 将"5 秒内"墙钟断言识别为 coding-standards.md 确定性规则所禁止的时间依赖断言——不为其辩护
- [ ] 不单方面推翻 gameplay-programmer 的不稳定担忧
- [ ] 若未解决，上报至 `technical-director`（qa-lead 质量标准的"汇报对象"；coordination-rules.md 规则 3 下的共同上级）——而非 lead-programmer，后者是 gameplay-programmer 自己的主管
- [ ] 不放弃覆盖率要求——Logic 故事仍需通过的自动化测试，故要求确定性替代方案而非接受手动走查

### 用例 5：上下文传递——使用提供的上下文
**场景：** Agent 收到门禁上下文块，包含 coding-standards.md 测试标准章节，其中规定：Logic 故事需要阻塞性自动化单元测试，Visual/Feel 故事需要阻塞性保留截图 + 主管签核，Config/Data 故事需要冒烟检查通过（建议性）。上下文说明项目运行 `qa.level: standard`。一个分类为"Logic"类型的故事仅提交了手动走查文档作为证据。
**预期：** 评估引用 coding-standards.md 中的具体测试证据要求，指出"Logic"故事需要自动化单元测试（而非仅手动走查），并返回 INADEQUATE 且引用具体要求。
**断言：**
- [ ] 引用所提供上下文中的具体故事类型分类（"Logic"）
- [ ] 引用 coding-standards.md 中 Logic 故事的具体证据要求（自动化单元测试）
- [ ] 指出提交的证据类型（手动走查）对该故事类型不充分
- [ ] 不将建议性级别要求当作阻塞性要求

### 用例 6：门禁输入缺失——NOT ASSESSED
**场景：** QL-STORY-READY 被调用用于一个 Logic 类型故事"冲刺时耐力消耗"，提供了文件路径、类型和两条具体可度量的验收标准——但缺少 GDD 需求：上下文省略了故事覆盖的 TR-ID 和文本，故事文件也未指明。
**预期：** 返回 `QL-STORY-READY: NOT ASSESSED`，指明缺失的输入（故事覆盖的 GDD 需求），而非 ADEQUATE——针对无人提供的需求，就绪度尚未确立。
**断言：**
- [ ] 裁决为 `NOT ASSESSED`——而非 ADEQUATE、GAPS 或 INADEQUATE——因为它能读取的标准未显示问题
- [ ] 指明缺失的输入：故事覆盖的 GDD 需求（TR-ID 和文本）
- [ ] 不编造或假设 GDD 需求以完成检查

### 用例 7：`qa.level: minimal`——测试豁免，截图不豁免
**场景：** 被要求为某冲刺的 QA 计划设定测试证据门禁时，Agent 被告知项目运行 `qa.level: minimal`（默认值）。该冲刺包含一个无测试文件的 Logic 故事和一个 UI 故事。
**预期：** 应用所给的 `qa.level`（其起草工作流会询问 `qa.level` 或 `testing.strict` 是否改变任何门禁级别）：Logic 故事缺失测试不是阻塞项——`minimal` 级别下测试被豁免——而 UI 故事仍需其涉及的每个屏幕的保留截图。
**断言：**
- [ ] 在 `qa.level: minimal` 下不将 Logic 故事缺失测试标记为阻塞项或硬门禁
- [ ] 说明 `qa.level: minimal` 豁免测试证据
- [ ] 仍要求 UI 故事的保留截图——外观从不豁免

---

## 协议合规

- [ ] 仅使用 ADEQUATE / GAPS / INADEQUATE 返回 QL-STORY-READY 裁决——或当门禁要求的输入缺失且未发现问题时，使用 NOT ASSESSED 并指明输入
- [ ] 仅使用 ADEQUATE / GAPS / INADEQUATE 返回 QL-TEST-COVERAGE 裁决——或当门禁要求的输入缺失且未发现问题时，使用 NOT ASSESSED 并指明输入
- [ ] 保持在声明的 QA 和测试策略领域内
- [ ] 将未解决的技术标准争议上报至 technical-director
- [ ] 每个门禁裁决以明确的裁决词表述——多个故事时每个故事一个——而非由行文隐含的裁决
- [ ] 不做有约束力的实现或游戏设计决策

---

## 覆盖说明
- QL-TEST-COVERAGE（冲刺或里程碑的整体覆盖率评估）未覆盖——待覆盖率报告可用时应添加专门用例。
- Bug 严重程度分级（S1–S4 分类）此处未覆盖——推迟至 /bug-triage 技能集成。
- 发布质量门禁执行（崩溃率、严重 Bug 数、性能基准）未覆盖。
- QL-STORY-READY 与故事完成标准（/story-done 技能）之间的交互未覆盖。