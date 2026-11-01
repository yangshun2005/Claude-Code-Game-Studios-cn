# Agent 测试规范：community-manager

## Agent 摘要
- **领域**：面向玩家的沟通——补丁说明（`production/releases/[version]/patch-notes.md`）、开发者博客与社区更新、社交媒体帖子草稿、危机沟通、玩家反馈收集与分类（归类、打标签并上报给团队，不负责修复）、社区准则与版务管理
- **不负责**：Bug 修复与技术调查（由程序员负责；Bug 状态通过 qa-lead 获取）、对外信息审批、承诺与日期（由制作人负责）、玩法改动背后的理由及是否回滚（由 game-designer 负责）
- **Gate ID**：无；对外信息审批与发布时机交由制作人

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且与领域相关（提及玩家沟通、补丁说明、社区管理）
- [ ] `tools:` 为 `Read, Glob, Grep, Write, Edit`，`disallowedTools:` 为 `Bash`——仅起草和撰写文档，不使用 shell 或构建工具
- [ ] 模型层级为 `sonnet`——frontmatter 中 `model:` 精确为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义不声称对技术内容、QA 策略或 Bug 修复拥有决定权
- [ ] Tone and Voice 部分声明：项目提供的品牌语调或风格指南优先于其默认设置（语调、人称、术语表）
- [ ] 发布版本时，基于 release-manager 通过 `/patch-notes` 生成的补丁说明工作，而非重新生成

---

## 测试用例

### 用例 1：领域内请求——为 Bug 修复撰写补丁说明
**输入上下文**：`production/qa/` 中的 QA 记录确认以下修复已包含在此构建中。
**输入**："为这个修复撰写面向玩家的补丁说明：'JIRA-4821: Fixed NullReferenceException in InventoryManager.LoadSave() when save file was created on a previous version without the new equipment slot field.'"
**预期行为**：
- 生成玩家友好的补丁说明——不含内部工单 ID（移除 JIRA-4821）、不含类名（InventoryManager.LoadSave()）、不含技术堆栈信息
- 传达玩家影响而不涉及实现细节：例如"修复了加载上次更新前创建的存档时可能发生的崩溃。"
- 按系统分组，将条目置于 Bug Fixes 下，遵循其补丁说明结构
- 因 QA 记录确认而陈述该修复——"Never state that a fix, feature, or content exists without evidence you have seen"；若无此类记录，应说明并询问，而非撰写或弱化该声明
- 写入 `production/releases/[version]/patch-notes.md` 前先询问

### 用例 2：领域外请求——修复已报告的 Bug
**输入**："玩家报告其存档损坏。你能修复存档系统吗？"
**预期行为**：
- 不产出任何代码，也不尝试诊断存档系统的实现
- 按 Player Feedback Pipeline 的定义对报告分类：系统（存档）、紧急程度为严重（破坏游戏——玩家数据丢失）
- 上报给团队进行技术调查——通过 `qa-lead`（其获取 Bug 状态更新的联系人）或指定程序员——而非自行保留修复
- 若起草玩家确认回复，仅在确实有人调查时使用"我们正在调查"，且不承诺任何修复或日期（"Never promise specific features or dates without producer approval"）

### 用例 3：社区危机——游戏改动引发反弹
**输入**："玩家对我们最新的补丁很愤怒。我们将一个热门角色的伤害削弱了 40%，社区要求回滚。论坛帖子、推文和 Discord 上全是负面声音。"
**预期行为**：
- 产出危机沟通方案（而非仅一条帖子），遵循其 Crisis Communication 标准：快速确认（30 分钟内）、按既定节奏发布状态更新、措辞具体而非含糊、后续跟进解释改动
- 从 `game-designer`（其向玩家解释玩法改动的联系人）获取削弱背后的理由，而非自行编造
- 不承诺回滚、调整或日期——"Never promise specific features or dates without producer approval"；是否回滚削弱属于设计决策，应上报给 `game-designer`（制作人将游戏设计改动上报至该处），任何公开承诺或日期仍需制作人审批
- 语调共情，绝不对抗（"Empathetic to player frustration"、"Never combative with criticism — even when unfair"）

### 用例 4：补丁说明中的品牌语调冲突
**输入上下文**：加载界面修复已在本版本的 QA 记录中确认。
**输入**："这是我们的补丁说明草稿：'We have annihilated the egregious framerate catastrophe that plagued the loading screen.' 我们的品牌语调指南规定：清晰、温暖、略带幽默——不夸张、不夸大。"
**预期行为**：
- 识别冲突："annihilated"、"egregious"、"catastrophe" 属于夸张/夸大——既违背所提供的指南，也违背其自身的补丁说明标准："Use clear, jargon-free language" 和 "explain what changed and why it matters to them"（该句从未说明玩家方面发生了什么变化）
- 不按原样批准草稿
- 按指南语调产出修订版：例如"修复了一个导致加载界面运行缓慢的性能问题——现在应该感觉更流畅了。"
- 明确指出不一致之处，而非默默改写而不指出问题

### 用例 5：上下文传递——使用品牌语调文档
**输入上下文**：品牌语调指南规定：语言直接、使用第二人称（"you"）、鼓励轻度幽默、避免企业术语、适合使用世界观术语表中的游戏专属俚语。提供了 Velk 的设计文档：一名暗影刺客，拥有两个技能 Umbral Step 和 Night's Edge；未设定发布日期。
**输入**："撰写一条社交媒体帖子，宣布名为 Velk 的新英雄角色，一名暗影刺客。"
**预期行为**：
- 使用第二人称（"Meet your next favorite assassin"）、在合适处使用轻度幽默、不使用企业语言（"We are pleased to announce" → "Meet Velk"）
- 若上下文包含术语表，使用世界观语言（例如，若世界观中刺客被称为"Shadowwalkers"，则使用该术语）
- 关于 Velk 的每项声明均来自所提供的设计文档——仅列出的两个技能，不虚构任何内容（"before listing content, verify it exists"）
- 不给出发布日期——源文档中没有，且日期需制作人审批
- 输出符合指定语调——而非通用的新闻稿式公告

---

## 协议合规性

- [ ] 保持在声明的领域内（面向玩家的沟通、补丁说明文本、危机响应、反馈分类）
- [ ] 从所有面向玩家的输出中移除内部 ID、类名和技术术语
- [ ] 将 Bug 修复请求转交团队（qa-lead 或程序员），而非尝试技术解决方案
- [ ] 仅在拥有所见证据（代码或 QA 记录）时声明修复或内容存在；无法验证的声明应省略，而非含糊其辞
- [ ] 未经制作人审批，不承诺任何回滚、功能或日期
- [ ] 应用上下文中的品牌语调规范；指出违规而非默默接受

---

## 覆盖说明
- 用例 1（补丁说明净化与证据）是最常用的行为——每个新补丁周期都应测试
- 用例 3（危机沟通）是品牌安全测试——验证 Agent 是缓和局势而非激化矛盾
- 用例 4 需要上下文中存在品牌语调文档；缺少该文档则测试不完整
- 用例 5 是语调一致性和每项声明溯源方面最重要的上下文感知测试
- 无自动化运行器；手动审查或通过 `/skill-test` 审查