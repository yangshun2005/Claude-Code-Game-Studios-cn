# Agent 测试规范：creative-director

## Agent 摘要
**负责领域：** 创意愿景、游戏支柱、GDD 对齐、系统拆解反馈、叙事方向、试玩反馈解读、阶段门禁（创意方面）。
**不负责：** 技术架构或实现细节（交由 technical-director）、制作排期（producer）、视觉美术风格执行（交由 art-director）。
**处理的 Gate ID：** CD-PILLARS、CD-GDD-ALIGN、CD-SYSTEMS、CD-NARRATIVE、CD-PLAYTEST、CD-PHASE-GATE。

---

## 静态断言（结构性）

通过读取该 agent 的 `.claude/agents/creative-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且针对本领域（提及创意愿景、支柱、GDD 对齐——而非泛泛描述）
- [ ] `tools:` 为 Read、Glob、Grep、Write、Edit、WebSearch，且 `disallowedTools:` 列出 Bash
- [ ] 模型层级为 `opus`——frontmatter `model:` 精确为 `opus`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对技术架构或制作排期拥有权限

---

## 测试用例

### 用例 1：领域内请求——输出格式恰当
**场景：** 一款叙事生存游戏的支柱集提交压力测试，包含其反支柱、核心幻想和独特钩子。钩子点明最接近的同类作品——"像《饥荒》，而且每失去一名幸存者都会永久改变世界"——且三根支柱中没有一个是该同类作品所围绕的目标。三根支柱为"涌现故事""有意义的牺牲""有生活气息的世界"，每根都有定义和设计测试，测试点明它能裁决的真实决策；支柱之间相互拉扯（牺牲 vs. 玩家想守护的世界）。请求标记为 CD-PILLARS。
**预期：** 返回 `CD-PILLARS: APPROVE`，按门禁标准对每根支柱给出具体反馈：是否可证伪、是否与其他支柱形成张力、是否让游戏有区分度、是否能裁决真实的设计分歧。
**断言：**
- [ ] 结论恰好为 APPROVE / CONCERNS / REJECT 之一
- [ ] 结论 token 格式为 `CD-PILLARS: APPROVE`（Gate ID 前缀、冒号、结论关键词）
- [ ] 按门禁标准（可证伪性、张力、与《饥荒》的区分度）逐一点名三根支柱给出反馈，而非泛泛的创意建议
- [ ] 输出限于创意范围——不评论引擎可行性或迭代排期

### 用例 2：领域外请求——重定向或上报
**场景：** 开发者请 creative-director 审查一个用于存储玩家存档数据的 PostgreSQL Schema 提案。
**预期：** Agent 拒绝评估该 Schema，并重定向至 technical-director。
**断言：**
- [ ] 不对 Schema 设计做出任何有约束力的决定
- [ ] 明确点名 `technical-director` 为正确处理者
- [ ] 对 Schema 的任何评论仅限于其创意影响（例如存档必须记住哪些玩家选择）——不提出任何表、列、键或查询设计

### 用例 3：门禁结论——用词正确
**场景：** 提交"制作"系统的 GDD，游戏支柱为"好奇心总有回报"和"世界会记住"，MDA 目标以 Discovery 优先。第 4 节（公式）定义了一个惩罚探索的资源衰减公式——与"自由漫游无惧"的 Player Fantasy 部分矛盾，且违背好奇心支柱。其余各节均服务于支柱，重新调整公式即可解决冲突，无需重新设计系统。请求标记为 CD-GDD-ALIGN。
**预期：** 返回 `CD-GDD-ALIGN: CONCERNS`，具体引用公式行为与 Player Fantasy 陈述之间的矛盾。修复以创意约束形式表述；公式本身留给设计团队。
**断言：**
- [ ] 结论恰好为 APPROVE / CONCERNS / REJECT 之一——不是自由文本
- [ ] 结论 token 格式为 `CD-GDD-ALIGN: CONCERNS`
- [ ] 理由引用或直接提及 GDD 第 4 节（公式）、Player Fantasy 部分和"好奇心总有回报"支柱
- [ ] 将修复表述为创意约束（例如衰减不得惩罚漫游），并将公式留给 `game-designer`（其机制设计代表）——不撰写替代公式数值

### 用例 4：冲突上报——正确的上级
**场景：** technical-director 提出担忧：核心循环机制（实时分支对话）实现成本过高，建议砍掉。creative-director 基于创意理由不同意。
**预期：** creative-director 承认技术约束，不推翻 technical-director 的可行性评估，但保留定义创意目标的权限。对于冲突本身，creative-director 是创意层面的最高上报点，在实现可行性上服从 technical-director，同时为设计意图辩护。解决路径是双方共同向用户呈现权衡选项。
**断言：**
- [ ] 不单方面推翻 technical-director 的可行性担忧
- [ ] 清楚区分"我们创意上想要什么"与"如何实现"
- [ ] 提议向用户呈现权衡，而非单方面解决
- [ ] 不声称拥有实现决策权

### 用例 5：上下文传递——使用提供的上下文
**场景：** Agent 收到门禁上下文块，包含游戏支柱文档（`design/gdd/game-pillars.md`）和一份待审查的新系统 GDD。支柱文档定义"玩家创作权""后果永久性""世界响应性"为三根核心支柱。请求标记为 CD-GDD-ALIGN。
**预期：** 评估使用所提供文档中的确切支柱词汇，而非泛泛的创意启发式。任何批准或担忧都关联回三根具名支柱中的一根或多根。
**断言：**
- [ ] 使用所提供上下文文档中的确切支柱名称
- [ ] 不生成与所提供支柱脱节的泛泛创意反馈
- [ ] 引用与被审查机制最相关的具体支柱
- [ ] 不引用所提供文档中不存在的支柱

### 用例 6：门禁输入缺失——NOT ASSESSED
**场景：** 提交一份完整、内部一致的系统 GDD，含 Player Fantasy 部分，但没有游戏支柱：上下文未提供，且 `design/gdd/game-concept.md`、`design/gdd/game-pillars.md`、`design/game-brief.md` 均不存在。请求标记为 CD-GDD-ALIGN。
**预期：** 返回 `CD-GDD-ALIGN: NOT ASSESSED`，点名游戏支柱为无法读取的输入。无法判断与无人提供的支柱的对齐情况，因此不给出对齐结论。
**断言：**
- [ ] 结论 token 为 `CD-GDD-ALIGN: NOT ASSESSED`——而非 APPROVE，尽管 GDD 本身看起来没有问题
- [ ] 点名游戏支柱为缺失输入
- [ ] 不从被审查的 GDD 推断支柱并据此判断对齐

---

## 协议合规

- [ ] 返回所调用门禁定义文件列出的结论词汇——CD-PILLARS、CD-GDD-ALIGN、CD-SYSTEMS、CD-NARRATIVE、CD-PLAYTEST 为 APPROVE / CONCERNS / REJECT；CD-PHASE-GATE 为 READY / CONCERNS / NOT READY；任何门禁在缺少其点名输入时返回 `NOT ASSESSED` 并点名该输入
- [ ] 保持在声明的创意领域内
- [ ] 通过向用户呈现权衡来上报冲突，而非单方面推翻
- [ ] 输出中使用 Gate ID（例如 `CD-PILLARS: APPROVE`），而非行内散文式结论
- [ ] 不做有约束力的跨领域决定（技术、制作、美术执行）
- [ ] 在阶段门禁中，目标阶段要求且调用技能传入为"none"的构建产物视为发现项（NOT READY 或 CONCERNS）；传入为"在[阶段]之前不预期"或"在 `workflow: [tier]` 下不需要"的视为无发现项——它判断的是即将进入阶段的就绪度，而非后续阶段

---

## 覆盖说明
- 多门禁场景（例如单次提交同时触发 CD-PILLARS 和 CD-GDD-ALIGN）此处未覆盖——留待集成测试。
- CD-PHASE-GATE（创意总监自身的就绪审查，由 `/gate-check` 在 `workflow: full` 下运行）尚无用例。
- CD-PLAYTEST（审查 `/playtest-report` 报告）尚无用例。
- 与 art-director 在视觉支柱对齐上的交互未覆盖。