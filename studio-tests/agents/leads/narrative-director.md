# Agent 测试规范：narrative-director

## Agent 摘要
**负责领域：** 故事架构、角色设计方向、世界观构建监督、ND-CONSISTENCY 门禁、对话策略。
**不负责：** 视觉艺术风格（art-director）、技术系统或代码（lead-programmer）、制作排期（producer）、游戏机制规则（game-designer）。
**处理的门禁 ID：** ND-CONSISTENCY（裁决为 APPROVE / CONCERNS / REJECT，定义于 `.claude/docs/director-gates/nd-consistency.md`）。

---

## 静态断言（结构性）

通过读取该 Agent 的 `.claude/agents/narrative-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域明确（涉及故事、角色、世界观构建、一致性——而非泛泛描述）
- [ ] `tools:` 包含 Read、Glob、Grep、Write、Edit、WebSearch，且 `disallowedTools:` 列出 Bash
- [ ] 模型层级为 `inherit`——frontmatter 的 `model:` 精确为 `inherit`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对视觉风格、技术系统或制作排期拥有权限

---

## 测试用例

### 用例 1：领域内请求——输出格式恰当
**场景：** 提交一份关于"The Sunken Archive"地点的新传说文档。文档设定该 Archive 在大崩塌期间于 200 年前被淹没，与世界圣经中既定时间线一致。所引用的具名角色均与其既定档案一致，语气符合叙事圣经的语气指南。门禁上下文通过了 `.claude/docs/director-gates/nd-consistency.md` 所要求的每一项输入：文档路径、语气指南、世界圣经的时间线规则，以及文档所引用角色的档案。请求标记为 ND-CONSISTENCY。
**预期：** 返回 `APPROVE`，理由中确认时间线一致、角色一致及语气符合。
**断言：**
- [ ] 裁决恰为 APPROVE / CONCERNS / REJECT 之一（ND-CONSISTENCY 门禁的词汇）——此处为 APPROVE
- [ ] 理由回答了门禁提示所问的每个问题：角色与既定档案的一致性、与既定传说的矛盾、以及语气
- [ ] 理由引用了所验证的具体既定事实（200 年时间线、大崩塌事件）
- [ ] 输出保持在叙事范围内——不评论该地点的视觉设计或其技术实现

### 用例 2：领域外请求——重定向或上报
**场景：** 一名开发者要求 narrative-director 审查并优化用于 Archive 器物"ancient glow"视觉效果的着色器代码。
**预期：** Agent 拒绝评估着色器代码。它指出 `technical-artist`——其自身"禁止"清单中着色器、VFX 与渲染代码的负责人——负责审查和优化；同时指出 `art-director`（负责效果应呈现的样子）也可以。它补充的任何叙事输入仅为效果应承载的氛围。
**断言：**
- [ ] 不审查、重写或优化着色器代码，不对视觉实现做出任何约束性决定
- [ ] 依据其"审查或更改着色器、VFX 或渲染代码（technical-artist 负责；art-director 负责应呈现的样子）"的限制，指出 `technical-artist`（或引擎的着色器专家）为着色器代码的负责人——同时指出 `art-director` 可接受，仅指出 art-director 则不可
- [ ] 其给出的任何叙事输入（例如"应感觉古老而神圣，而非技术感"）均定位为给视觉负责人的输入，而非视觉或技术决定

### 用例 3：门禁裁决——词汇正确
**场景：** 提交一份关于角色"Aldric Vorne"的新角色背景故事文档。文档称 Aldric 于 150 年前出生在首都，并亲眼见证了大崩塌；背景故事的核心弧线是他对所目睹崩塌的愧疚。既定世界圣经称 Aldric 出生于大崩塌后 50 年的一座外省小镇，而非首都。请求标记为 ND-CONSISTENCY。
**预期：** 返回 `REJECT`——被矛盾的事实承载着背景故事的核心弧线，因此破坏了其叙事基础，而非局部修正——列出两处矛盾：出生时间（150 年前 vs. 崩塌后 50 年）与出生地点（首都 vs. 外省小镇）。
**断言：**
- [ ] 裁决恰为 APPROVE / CONCERNS / REJECT 之一——而非自由文本；此处为 REJECT，因为被矛盾的事实对整个背景故事是承重的（它发现的矛盾以门禁的措辞为准，因此即使未传入角色档案也不是 NOT ASSESSED）
- [ ] 裁决以门禁的 `REJECT [contradictions ...]` 形式具体承载两处矛盾——而非仅"与传说不符"
- [ ] 引用世界圣经作为既定事实的权威来源
- [ ] 不自行撰写替代背景故事——重写属于内容作者（传说条目在其 Delegation Map 中委托给 `writer` / `world-builder`）

### 用例 4：冲突上报——正确的上级
**场景：** 一名 writer 在其最新对话中设定古代文明"仅以歌唱交流"。world-builder 的现有传说条目描述同一文明通过书写字形交流。二者均向 narrative-director 汇报，且二者对何为正典存在分歧。
**预期：** narrative-director 在其自身层级解决冲突——它是 writer 与 world-builder 的共同上级，因此不上报给 creative-director。遵循其协作协议，它提出调和选项并附推荐的正典裁决（例如"字形书写为正典的主要交流方式；歌唱为仪式/典礼"），将最终决定留给用户，然后指示 writer 与 world-builder 双方将工作对齐至确认的裁决。
**断言：**
- [ ] 作为共同上级自行解决冲突——不将此叙事内冲突交由 creative-director 处理
- [ ] 提出调和选项并明确陈述推荐裁决，将最终正典决定留给用户
- [ ] 指示双方更新文档以符合裁决：writer 修订对话，world-builder 更新传说条目
- [ ] 提议将裁决作为正典事实记录在世界观构建文档中，以便后续传说可据此核查

### 用例 5：上下文传入——使用提供的上下文
**场景：** 提交一个新故事章节进行 ND-CONSISTENCY 审查。它引入了一个此前未登记的角色。门禁上下文传入三份文档：世界圣经（大崩塌的时间线与起因）、角色登记表（正典角色的年龄、出身与效忠）、以及一份描述 Sunken Archive Keepers 的派系文档。
**预期：** 评估将新角色与角色登记表交叉比对（无冲突），将章节的时间线引用与世界圣经核对，并依据派系文档评估章节对 Archive Keepers 的刻画。使用三份提供文档中的具体事实并返回 ND-CONSISTENCY 裁决。
**断言：**
- [ ] 将新角色与提供的角色登记表交叉比对
- [ ] 将时间线引用与提供的世界圣经事实核对
- [ ] 依据提供的派系文档评估派系刻画
- [ ] 返回 APPROVE / CONCERNS / REJECT 之一，理由可追溯至三份提供的文档——无泛泛的叙事反馈

### 用例 6：门禁输入缺失——NOT ASSESSED
**场景：** 对一个"Aldric Vorne"发言的新对话文件调用 ND-CONSISTENCY。门禁上下文传入文件路径与世界观构建规则，对话与二者均无矛盾——但 Aldric 的角色档案（门禁所要求的输入）未传入，因此无法核对其声音。
**预期：** 返回 `ND-CONSISTENCY: NOT ASSESSED`，指出缺失的输入（Aldric Vorne 的角色档案），而非 APPROVE——针对从未见过的档案的声音一致性尚未确立。
**断言：**
- [ ] 裁决为 `NOT ASSESSED`——而非 APPROVE、CONCERNS 或 REJECT——因为其可读取的输入未显示不一致
- [ ] 指出缺失的输入：Aldric Vorne 的角色档案
- [ ] 不从对话本身或记忆中重建 Aldric 的既定声音以完成检查

---

## 协议合规

- [ ] 仅使用 APPROVE / CONCERNS / REJECT 返回 ND-CONSISTENCY 裁决——或当门禁所要求的输入缺失且未发现不一致时，返回 NOT ASSESSED 并指出该输入
- [ ] 创建叙事文档前询问"May I create [filepath] with the section skeleton?"，写入每个章节前询问"May I write this section to [filepath]?"
- [ ] 保持在声明的叙事领域内
- [ ] 在自身层级解决叙事内冲突而非上报，同时将最终正典决定留给用户
- [ ] CONCERNS 或 REJECT 裁决按门禁提示的 `CONCERNS [...]` / `REJECT [...]` 形式指出具体的不一致或矛盾——绝不仅是一个裁决词
- [ ] 不做约束性的视觉设计、技术或制作决定

---

## 覆盖说明
- 对话质量审查（区别于世界观构建一致性）未覆盖——应添加专门用例。
- CONCERNS 裁决（可修复、不破坏叙事基础的不一致）无专门用例。
- 跨完整章节集的多文档一致性检查未覆盖——推迟至 /review-all-gdds 集成。
- 机制变更的叙事影响（例如削弱故事张力的游戏机制）需与 game-designer 协调，此处未覆盖。
- 角色弧线审查（进展、动机随时间的连贯性）未覆盖。