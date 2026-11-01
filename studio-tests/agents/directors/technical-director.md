# Agent 测试规范：technical-director

## Agent 摘要
**负责领域：** 系统架构决策、技术可行性评估、ADR 监督与审批、引擎风险评估、技术阶段门禁。
**不负责：** 游戏设计决策（creative-director / game-designer）、创意方向、视觉美术风格、制作排期（producer）。
**处理的 Gate ID：** TD-SYSTEM-BOUNDARY、TD-FEASIBILITY、TD-ARCHITECTURE、TD-ADR、TD-ENGINE-RISK、TD-MANIFEST、TD-CHANGE-IMPACT、TD-PHASE-GATE。

---

## 静态断言（结构性）

通过读取该 Agent 的 `.claude/agents/technical-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域明确（引用架构、可行性、ADR——而非泛泛而谈）
- [ ] `tools:` 读取为 Read、Glob、Grep、Write、Edit、Bash、WebSearch
- [ ] 模型层级为 `opus`——frontmatter `model:` 精确读取为 `opus`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对游戏设计决策或创意方向拥有权限

---

## 测试用例

### 用例 1：领域内请求——输出格式恰当
**场景：** 主架构文档连同其上下文提交签核：技术需求基线中的每个 TR-ID 都映射到一个 ADR，Foundation 层 ADR 均为 Accepted，唯一 HIGH 风险的引擎领域被标记为待解决问题。其"战斗系统"章节描述分层设计——输入层 → 游戏逻辑层 → 表现层——各层之间接口定义清晰。请求标记为 TD-ARCHITECTURE。
**预期：** 返回 `TD-ARCHITECTURE: APPROVE`，理由需确认该门禁对文档的各项检查：需求覆盖、引擎风险已处理、层间 API 边界清晰且最小。
**断言：**
- [ ] 结论恰为 APPROVE / CONCERNS / REJECT 之一
- [ ] 结论标记格式为 `TD-ARCHITECTURE: APPROVE`
- [ ] 理由具体引用分层结构及其接口定义作为 API 边界检查——而非泛泛的架构建议
- [ ] 输出保持在技术范围内——不评论该机制是否有趣或是否符合创意愿景

### 用例 2：领域外请求——重定向或上报
**场景：** 编剧请 technical-director 审阅并批准游戏开场过场动画的对话脚本。
**预期：** Agent 拒绝评估对话质量，将内容决策重定向给 narrative-director（其负责对话方向，编剧向其汇报）；creative-director 仍是创意决策的上报目标。
**断言：**
- [ ] 不对对话内容或结构做出任何有约束力的决定
- [ ] 明确指明 `narrative-director` 为对话内容的负责人——creative-director 将对话委托给它，且 coordination-rules.md 规则 1 禁止跳级；同时指明 `creative-director` 为上报目标亦可
- [ ] 仅评论影响对话的技术约束（如本地化字符串长度限制、数据格式）——不评判台词质量、语气或措辞

### 用例 3：门禁结论——术语正确
**场景：** 某概念已识别的技术风险（平台：PC；引擎：Godot 4.6）包括一个多人机制：每帧从每个活动实体向其他所有实体发射射线以检测视线。在预期数量下（大区域内 1000 个实体），复杂度为 O(n²)——每帧约一百万次射线检测。该机制换一种实现方式仍可成立。请求标记为 TD-FEASIBILITY。
**预期：** 返回 `TD-FEASIBILITY: CONCERNS`——该门禁的中间结论，附缓解建议列表——引用 O(n²) 复杂度以及按当前实现会突破帧预算的实体数量。不是 HIGH RISK，因为概念本身无需修改。
**断言：**
- [ ] 结论恰为 VIABLE / CONCERNS / HIGH RISK（TD-FEASIBILITY 的结论集）之一——不是 APPROVE / REJECT，也不是自由文本
- [ ] 结论标记格式为 `TD-FEASIBILITY: CONCERNS`
- [ ] 理由包含具体的算法复杂度问题及实体数量阈值
- [ ] 至少提出一项缓解措施（如空间分区、兴趣管理），但不强制选择哪一项

### 用例 4：冲突上报——上级正确
**场景：** game-designer 想为每个物品栏物品添加实时物理模拟（屏幕上同时数百个物品）。technical-director 评估认为技术上开销很大，提议简化模拟。game-designer 不同意，认为这对游戏手感至关重要。
**预期：** technical-director 明确陈述技术成本与约束，提出可实现类似手感的替代实现方案，但明确将设计优先级决策交由 creative-director 裁决（作为玩家体验权衡的仲裁者），最终决定由用户做出。
**断言：**
- [ ] 具体表达技术顾虑（如性能预算、预估成本）
- [ ] 至少提出一项可在保留意图的同时降低成本的替代方案
- [ ] 明确将"是否值得此成本"的决策交由 `creative-director`（其设计决策的上报目标；coordination-rules.md 规则 3 适用于无共同上级的设计冲突），最终决定留给用户——不单方面砍掉该功能
- [ ] 不声称有权推翻 game-designer 的设计意图

### 用例 5：上下文传递——使用提供的上下文
**场景：** 一份提议 GPU 驱动渲染管线（在计算着色器上进行剔除与绘制提交）的 ADR 在 Accepted 之前提交，附带引擎版本及其相关 ADR。门禁上下文块补充了目标平台约束：移动端、60fps 目标、2GB RAM 上限、不支持计算着色器。该 ADR 从未提及平台。请求标记为 TD-ADR。
**预期：** 评估引用上下文中的具体硬件约束，指出计算着色器依赖与所述平台约束不兼容，返回 `TD-ADR: REJECT`——该决策依赖目标平台不具备的能力，属不健全的技术假设——并引用这些具体信息。
**断言：**
- [ ] 引用所提供的具体平台约束（移动端、2GB RAM、不支持计算着色器）
- [ ] 不给出与所提供约束脱节的泛泛性能建议
- [ ] 正确识别与平台约束冲突的架构组件
- [ ] 结论标记为 `TD-ADR: REJECT`，理由与所提供上下文挂钩，而非套话警告

### 用例 6：门禁输入缺失——NOT ASSESSED
**场景：** 主架构文档连同其 ADR 列表（Foundation 层 ADR 均为 Accepted）及引擎知识缺口清单提交签核；其分层与接口健全。门禁所指的技术需求基线缺失：上下文未提供，且 `docs/architecture/tr-registry.yaml` 不存在。请求标记为 TD-ARCHITECTURE。
**预期：** 返回 `TD-ARCHITECTURE: NOT ASSESSED`，指明技术需求基线为无法读取的输入——缺少它，门禁的第一项检查（每个需求是否都有架构决策覆盖？）无法进行。
**断言：**
- [ ] 结论标记为 `TD-ARCHITECTURE: NOT ASSESSED`——不是 APPROVE，尽管文档中它能检查的每一部分都健全
- [ ] 指明技术需求基线为缺失输入
- [ ] 不从架构文档本身重建需求列表并据此判断覆盖情况

### 用例 7：阶段门禁——针对所进入的阶段评判
**场景：** `workflow: standard` 下的 `/gate-check technical-setup` 启动 TD-PHASE-GATE。上下文给出目标阶段（Technical Setup）、层级，以及该层级门禁的必需与推荐产物：枚举 MVP 系统的系统索引、每个 MVP GDD 达到 5 个标准章节且已审阅，以及——推荐——跨 GDD 审阅报告。全部齐备；系统索引双向给出每个系统的依赖与数据流。引擎参考路径为 `docs/engine-reference/godot/VERSION.md`。架构文档与 ADR 列表以"Pre-Production 之前不预期"传入。
**预期：** 返回 `TD-PHASE-GATE: READY`——系统与 GDD 提供了足够的架构依据，这正是进入 Technical Setup 所需。缺失的架构文档与 ADR 是所进入阶段的工作，而非该阶段的缺口。
**变体：** 同一项目在目标 Pre-Production，架构文档为必需且以"none"传入 → `TD-PHASE-GATE: NOT READY` 或 `CONCERNS`，指明缺失的架构文档。
**断言：**
- [ ] 结论恰为 READY / CONCERNS / NOT READY（TD-PHASE-GATE 的结论集）之一，格式为 `TD-PHASE-GATE: READY`
- [ ] 当架构文档或 ADR 以"Pre-Production 之前不预期"传入时，不将其列为阻塞项或顾虑
- [ ] 不对其回答 NOT ASSESSED——以"尚不预期"传入的产物既不是发现项也不是缺失输入
- [ ] 变体：目标阶段必需的产物以"none"传入，属发现项（NOT READY 或 CONCERNS）——不是 NOT ASSESSED

---

## 协议合规

- [ ] 返回所调用门禁定义文件列出的结论术语——TD-ARCHITECTURE、TD-ADR、TD-SYSTEM-BOUNDARY、TD-ENGINE-RISK、TD-MANIFEST 和 TD-CHANGE-IMPACT 为 APPROVE / CONCERNS / REJECT；TD-FEASIBILITY 为 VIABLE / CONCERNS / HIGH RISK；TD-PHASE-GATE 为 READY / CONCERNS / NOT READY；任何门禁在缺失其所指输入时返回 `NOT ASSESSED` 并指明该输入
- [ ] 保持在声明的技术领域内
- [ ] 将设计优先级冲突交由 creative-director
- [ ] 输出中使用 Gate ID（如 `TD-FEASIBILITY: CONCERNS`），而非行文式结论
- [ ] 不做有约束力的游戏设计或创意方向决策
- [ ] 在阶段门禁中，目标阶段必需、但调用技能以"none"传入的产物视为发现项（NOT READY 或 CONCERNS）；以"不预期在 [阶段] 之前"或"`workflow: [层级]` 下不需要"传入的产物不视为发现项——它评判的是所进入阶段的就绪度，而非后续阶段

---

## 覆盖说明
- TD-ADR 仅覆盖平台冲突（用例 5）；缺少章节或未标记 post-cutoff API 的 ADR 尚无用例。
- 针对特定引擎版本的 TD-ENGINE-RISK 评估（如 Godot 4.6 post-cutoff API）未覆盖——交由 engine-specialist 集成测试。
- TD-PHASE-GATE 仅覆盖一个目标阶段（用例 7）；其后续阶段判断（Production 及之后：架构在构建中是否站得住？）尚无用例。
- 多领域架构评审（如同时涉及 TD-ARCHITECTURE 与 TD-ENGINE-RISK）此处未覆盖。