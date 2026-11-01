# Agent 测试规范：producer

## Agent 摘要
**负责领域：** 范围管理、冲刺规划验证、里程碑跟踪、史诗优先级排序、生产阶段门禁。
**不负责：** 游戏设计决策（creative-director / game-designer）、技术架构（technical-director）、创意方向。
**处理的 Gate ID：** PR-SCOPE、PR-SPRINT、PR-MILESTONE、PR-EPIC、PR-PHASE-GATE。

---

## 静态断言（结构性）

通过读取该 agent 的 `.claude/agents/producer.md` frontmatter 验证：

- [ ] `description:` 字段存在且针对领域（引用范围、冲刺、里程碑、生产——而非泛泛而谈）
- [ ] `tools:` 包含 Write 和 Edit（它按 Output Format 编写冲刺计划并维护风险登记册），包含 Bash 仅因为预加载的 skill 需要它（`skills:` 包含 `scope-check`，该 skill 读取 `git log`）
- [ ] 模型层级为 `opus`——frontmatter 的 `model:` 精确为 `opus`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对设计决策或技术架构拥有权限

---

## 测试用例

### 用例 1：领域内请求——输出格式恰当
**场景：** 提交 Sprint 7 的冲刺计划。该计划在 2 周内为 4 名团队成员分配 9 个故事点，列出了依赖关系，且没有单个故事超过 3 天。过去 3 个冲刺的历史速度平均为 11.5 点；没有积压债务带入，里程碑约束对本冲刺无额外要求。请求标记为 PR-SPRINT。
**预期：** 返回 `PR-SPRINT: REALISTIC`，理由说明负载低于历史速度，同时保留其 Sprint Planning Rules 要求的 20% 缓冲（9 ≤ 11.5 的 80%）。
**断言：**
- [ ] 结论恰好为 REALISTIC / CONCERNS / UNREALISTIC 之一
- [ ] 结论 token 格式为 `PR-SPRINT: REALISTIC`
- [ ] 理由将 9 个承诺点与 11.5 点速度及 20% 缓冲规则对比——而非泛泛的"看起来可行"
- [ ] 输出保持在生产范围内——不评论故事设计是否良好或技术是否合理

### 用例 2：领域外请求——重定向或上报
**场景：** 团队成员要求 producer 评估游戏的"基于重量的库存"机制是否有趣且引人入胜。
**预期：** Agent 拒绝评估游戏手感，并重定向至 game-designer 或 creative-director。
**断言：**
- [ ] 不对该机制的设计质量做出任何有约束力的评估
- [ ] 明确指定 `game-designer` 或 `creative-director` 为正确处理者
- [ ] 任何评论仅涉及生产影响（例如该机制依赖哪些其他系统）——绝不涉及机制是否有趣、引人入胜或设计良好

### 用例 3：Gate 结论——正确词汇
**场景：** 为一名独立开发者提交范围估算，时间线 4 个月。MVP 原为两个系统（移动和战斗），估算 3 个月。修订后的 MVP 增加三个——制作、天气和阵营声望——使估算恰好为 4 个月：整个时间线，没有为计划外工作留缓冲。这三个系统各自都可以移到下一个范围层级而不破坏 MVP。请求标记为 PR-SCOPE。
**预期：** 返回 `PR-SCOPE: OPTIMISTIC`——该 gate 的中间结论，附带具体调整——因为 MVP 只有在一切顺利时才成立：点名三个新增系统，并建议将哪些移到较晚的范围层级以恢复缓冲。不是 UNREALISTIC：估算未超出时间线，因此两者都无需修订。
**断言：**
- [ ] 结论恰好为 REALISTIC / OPTIMISTIC / UNREALISTIC 之一（PR-SCOPE 的结论）——不是 CONCERNS，也不是自由文本
- [ ] 结论 token 格式为 `PR-SCOPE: OPTIMISTIC`
- [ ] 理由点名消耗缓冲的三个具体系统
- [ ] 不评估这些系统设计是否良好——只评估它们是否契合计划

### 用例 4：冲突上报——正确的上级
**场景：** game-designer 想添加一个后期机制（影响所有游戏系统的动态天气），technical-director 警告这需要额外 3 个冲刺。game-designer 和 technical-director 就是否继续存在分歧。
**预期：** Producer 不站队该机制是否值得添加（设计决策）或是否可行（技术决策）。Producer 量化生产影响（3 个冲刺的延迟、里程碑滑期风险），向用户呈现权衡，并遵循 coordination-rules.md 规则 3：game-designer 和 technical-director 没有共同上级，且该机制是否值得其成本属于设计冲突，因此上报至 creative-director——创意意图与生产能力冲突时范围问题的仲裁者。
**断言：**
- [ ] 以具体方式量化生产影响（冲刺数、里程碑日期滑期）
- [ ] 不做有约束力的设计或技术决策
- [ ] 向用户呈现冲突，清晰说明范围影响
- [ ] 按 coordination-rules.md 规则 3 将冲突上报至 `creative-director`（无共同上级；设计冲突）——而非 technical-director，也不直接抛给用户如同不存在上报目标

### 用例 5：上下文传递——使用提供的上下文
**场景：** Agent 收到一个 gate 上下文块，包含当前里程碑截止日期（8 周后，两周冲刺）和过去 4 个冲刺的速度数据（8、10、9、11 点）。提交的冲刺计划包含 5 个故事共 14 个故事点：玩家移动（3）、跳跃缓冲（2，依赖玩家移动）、敌人巡逻 AI（5）、暂停菜单（2）和存档槽（2）。请求标记为 PR-SPRINT。
**预期：** 评估使用提供的速度数据表明 14 点远超约 9.5 点的平均值（保留 20% 缓冲后更是如此），返回 `PR-SPRINT: UNREALISTIC` 并点名要推迟的故事，并引用 8 周里程碑窗口评估超支对里程碑的影响。
**断言：**
- [ ] 使用提供的上下文中的具体速度数字（而非泛泛估算）
- [ ] 在容量评估中引用 8 周截止日期
- [ ] 计算或估算里程碑窗口内剩余冲刺数（四个两周冲刺），这是其至少提前 2 个冲刺标记里程碑风险的职责要求
- [ ] 从提供的五个故事中按标题点名要推迟的故事，且绝不在推迟玩家移动的同时保留依赖它的跳跃缓冲
- [ ] 不给出与提供的截止日期和速度数据脱节的泛泛范围建议

### 用例 6：缺少 gate 输入——NOT ASSESSED
**场景：** 提交的冲刺计划包含五个故事——标题、估算和依赖关系，顺序正确——以及里程碑约束，但没有团队容量：上下文未提供，且 `production/sprints/` 中没有更早的冲刺可供推导速度。请求标记为 PR-SPRINT。
**预期：** 返回 `PR-SPRINT: NOT ASSESSED`，点名团队容量为其无法读取的输入。故事负载是否现实取决于衡量它的容量，因此不给出可行性结论。
**断言：**
- [ ] 结论 token 为 `PR-SPRINT: NOT ASSESSED`——不是 REALISTIC，也不是假设性的 UNREALISTIC
- [ ] 点名团队容量为缺失输入
- [ ] 不编造容量或速度数字来衡量负载

### 用例 7：`minimal` 下的阶段门禁——简报的 Build order 即计划
**场景：** `/gate-check production` 在 `workflow: minimal` 下运行面板，显式设置 `review_mode: lean`，因此 producer 单独为 PR-PHASE-GATE 启动。上下文给出目标阶段（Production）、层级，以及该层级下 gate 要求的产物：填好的 `design/game-brief.md` 及其 Build order，以及 `production/epics/` 下的故事。两者都存在——Build order 列出四个 MVP 功能，前三个独立，第四个依赖第一个，且每个功能按该顺序有一个故事，均未 Blocked。`team.size` 为 `individual`；简报声明 MVP 目标为六周。冲刺计划和冲刺容量作为"`workflow: minimal` 下不要求"传入。
**预期：** 返回 `PR-PHASE-GATE: READY`——在此层级 Build order 即计划，其依赖已排序，四个故事适合独立开发者的六周。缺失的冲刺计划和速度不构成发现项。
**断言：**
- [ ] 结论恰好为 READY / CONCERNS / NOT READY 之一（PR-PHASE-GATE 的结论），格式为 `PR-PHASE-GATE: READY`
- [ ] 将 Build order 作为计划评判，并检查其依赖顺序（第四个功能在第一个之后）
- [ ] 不将缺失的冲刺计划或速度作为关注点提出，也不对其回答 NOT ASSESSED——它们作为"`workflow: minimal` 下不要求"传入
- [ ] 不编造冲刺容量来衡量负载

---

## 协议合规

- [ ] 返回所调用 gate 定义文件列出的结论词汇——PR-SPRINT 和 PR-EPIC 为 REALISTIC / CONCERNS / UNREALISTIC；PR-SCOPE 为 REALISTIC / OPTIMISTIC / UNREALISTIC；PR-MILESTONE 为 ON TRACK / AT RISK / OFF TRACK；PR-PHASE-GATE 为 READY / CONCERNS / NOT READY；任何 gate 在缺少该 gate 点名的输入时返回 `NOT ASSESSED` 并点名该输入
- [ ] 保持在声明的生产领域内
- [ ] 通过量化范围影响并向用户呈现来上报设计/技术冲突
- [ ] 在输出中使用 gate ID（例如 `PR-SPRINT: REALISTIC`）而非行内散文式结论
- [ ] 不做有约束力的游戏设计或技术架构决策
- [ ] 在阶段门禁中，将目标阶段要求、但调用 skill 传入为"none"的产物视为发现项（NOT READY 或 CONCERNS），将传入为"在 [阶段] 之前不预期"或"`workflow: [tier]` 下不要求"的视为无发现项——它评判的是正在进入的阶段的就绪度，而非后续阶段

---

## 覆盖说明
- PR-EPIC（`/create-epics` 在写入任何史诗前运行的史诗结构审查）尚无用例。
- PR-MILESTONE（里程碑健康审查）未覆盖——推迟到与 /milestone-review skill 集成。
- PR-PHASE-GATE 在 `minimal` 下已覆盖（用例 7）；针对冲刺计划评判的 `standard`/`full` Production gate 尚无用例。
- 多冲刺燃尽图和速度趋势分析此处未覆盖。