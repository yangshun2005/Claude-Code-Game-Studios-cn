# 技能测试规范：/ux-design

## 技能摘要

`/ux-design` 逐节撰写 UX 文档。参数决定模式和输出文件：`hud` → `design/ux/hud.md`；`patterns` → `design/ux/interaction-patterns.md`；`accessibility` → `design/accessibility-requirements.md`（唯一写入 `design/ux/` 之外的模式）；其他值 → 位于 `design/ux/[kebab-name].md` 的界面/流程规范。无参数时询问"今天设计什么？"而非报错。

阶段 2 在提问前先读取上下文：游戏概念（或游戏简报）、玩家旅程、GDD UI Requirements、现有 UX 规范、模式库目录、美术圣经、无障碍需求，以及 `project.yaml` `platform` 块中的输入方式。若输出文件已存在，技能进入改造模式，仅填充 Empty 或 Placeholder 章节。否则询问"我可以在 [path] 创建骨架文件吗？"并写入该模式的骨架。

随后每节执行 上下文 → 提问 → 选项 → 决策 → 草稿 → 批准 → 写入："这准确描述了 [章节名] 吗？"，然后"我可以将 [章节名] 章节写入 `[filepath]` 吗？"，再用 Edit 替换占位符。各模式的指导文件中仅加载当前模式的，模板指南逐节读取。交叉引用检查后，交接说明规范必须通过 `/ux-review`，并以结论 `COMPLETE` 结束。无导演门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含结论关键词：`COMPLETE`
- [ ] 每节包含"May I write"措辞，骨架包含"May I create"
- [ ] 具有下一步交接（`/ux-review` 验证完成的规范）
- [ ] 记录四种模式及其输出路径，包括 `accessibility` 的 `design/accessibility-requirements.md`

---

## 导演门禁检查

无。`/ux-design` 仅解析 `automation`、`workflow` 和 `docs.density`（不含 `review_mode`），不生成导演门禁。`/ux-review` 是独立的验证步骤。可咨询专家（`art-director`、`ui-programmer`、`game-designer`、`narrative-director`）处理子主题；其输出呈现给用户，本会话负责所有文件写入。

---

## 夹具默认值

除非各自夹具另有说明，每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——新建 HUD 设计

**夹具：**
- 无 `design/ux/hud.md`
- `design/gdd/combat.md` 和 `design/gdd/stamina.md` 有 `## UI Requirements` 章节；`design/gdd/save-system.md` 没有
- `project.yaml` 有 `platform` 块（`targets: [PC]`、`gamepad_support: Full`）
- `design/gdd/game-concept.md` 和 `design/player-journey.md` 存在

**输入：** `/ux-design hud`

**预期行为：**
1. 模式为 HUD 设计；输出文件 `design/ux/hud.md`
2. 通过一次 Grep 在 `design/gdd/*.md` 中收集 UI Requirements；技能统计三个系统 GDD——`game-concept.md` 位于 `design/gdd/` 但不属于系统 GDD——将 `save-system.md` 列为未匹配，并与用户确认其为无头后排除
3. 输入方式从 `platform` 块推导（键盘/鼠标和手柄）——不询问用户
4. 显示上下文摘要，然后问"开始前还有什么需要我读的，还是继续？"
5. 无现有文件，故问："我可以在 `design/ux/hud.md` 创建骨架文件吗？"——同意后写入 HUD 骨架并更新 `production/session-state/active.md`。骨架包含 `/ux-review` 的 HUD 检查清单所需的每个章节，使完成的规范能通过该审查
6. 阶段 4 读取 `.claude/skills/ux-design/references/sections-hud.md`，绝不读取其他两个模式文件（`sections-ux-spec.md`、`sections-patterns.md`）；若某节需要深入，HUD 设计指南逐节读取，绝不整篇读取
7. 每节：草稿 → "这准确描述了 [章节名] 吗？" → "我可以将 [章节名] 章节写入 `design/ux/hud.md` 吗？" → Edit 替换该节占位符
8. 阶段 5 呈现交叉引用检查结果
9. 阶段 6 说明规范应在实现前用 `/ux-review` 验证，然后提供四个选项（立即运行 `/ux-review` / 设计另一界面 / 更新模式库 / 停止）；结论 `COMPLETE`

**断言：**
- [ ] 无 UI Requirements 章节的 GDD 被列出并与用户确认，而非静默排除
- [ ] `game-concept.md` 不计为系统 GDD，故不列为未匹配
- [ ] 输入方式来自 `project.yaml` `platform` 块，不询问用户
- [ ] 骨架仅在"我可以在 `design/ux/hud.md` 创建骨架文件吗？"批准后写入
- [ ] HUD 骨架包含 `/ux-review` 阶段 3B 检查的章节：HUD 理念、信息架构、布局区域、HUD 元素规格、按游戏情境的 HUD 状态、信息层级、视觉预算、反馈与通知系统、平台适配、调优旋钮、验收标准
- [ ] 三个模式指导文件中仅加载 `sections-hud.md`，且无指南被整篇读取
- [ ] 每节先批准，再获得各自的"我可以将 [章节名] 章节……"询问后才写入
- [ ] 交接指明实现前必须 `/ux-review`，结论为 `COMPLETE`

---

### 用例 2：现有文档——仅改造不完整章节

**夹具：**
- 场景 (a)：`design/ux/main-menu.md` 存在，基于 UX 规范骨架构建；Purpose & Player Need、Player Context on Arrival、Navigation Position 和 Entry & Exit Points 有实际内容；其他章节均为 `[To be designed]`
- 场景 (b)：`design/accessibility-requirements.md` 存在，基于无障碍骨架构建；Accessibility Tier Definition 承诺 Standard 层级；其他章节均为 `[To be designed]`

**输入：** (a) `/ux-design main-menu` (b) `/ux-design accessibility`

**预期行为：**
1. 在任何骨架步骤前，技能 Glob 输出路径——(b) 中为 `design/accessibility-requirements.md`，而非 `design/ux/` 下的路径——并找到文件
2. 读取文件，将每节分类为 Complete / Empty / Placeholder，显示状态表，结尾为"我将只处理 [N] 个不完整章节——现有内容不会被覆盖。"
3. 跳过骨架创建
4. 阶段 4 仅撰写 Empty/Placeholder 章节，每节在其"May I write"批准后用 Edit 就地填充占位符
5. (a) 四个完整章节不变；(b) 已承诺的层级不变

**断言：**
- [ ] 检测到现有文件，不写入骨架
- [ ] 撰写前显示状态表和"仅不完整章节"消息
- [ ] 仅撰写 Empty 或 Placeholder 章节；完整章节不被覆盖
- [ ] 每个填充章节在其"May I write"批准后用 Edit 写入
- [ ] (a) 状态表行使用 UX 规范骨架的章节名（如 `Purpose & Player Need`、`Layout Specification`、`States & Variants`、`Input Method Completeness Checklist`）
- [ ] (b) 状态表行使用无障碍骨架的章节名（如 `Accessibility Tier Definition`、`Visual Accessibility`、`Motor Accessibility`），文件保持在 `design/accessibility-requirements.md`

---

### 用例 3：缺少上下文——无游戏概念且无玩家旅程

**夹具：**
- 无 `design/gdd/game-concept.md` 且无 `design/game-brief.md`
- 无 `design/player-journey.md`
- 无 `design/ux/inventory.md`

**输入：** `/ux-design inventory`

**预期行为：**
1. 技能警告："未找到游戏概念。请先运行 `/brainstorm` 建立游戏基础，再设计 UX。"仅在用户要求时继续
2. 指出缺少旅程："未在 `design/player-journey.md` 找到玩家旅程图。缺少它意味着我们将对玩家情境做假设……"
3. 上下文摘要显示"旅程阶段：未知——无旅程图"
4. 规范的 Open Questions 章节记录："玩家旅程图尚未创建。请从 `.claude/docs/templates/player-journey.md` 的模板撰写……"
5. 技能绝不告诉用户运行 `/ux-design` 阶段 2b 来创建旅程图

**断言：**
- [ ] 缺少概念产生 `/brainstorm` 警告，技能仅在用户同意后继续
- [ ] 缺少玩家旅程在对话中指出，并在 Open Questions 中记录模板路径
- [ ] 补救指向模板，绝不指回 `/ux-design`
- [ ] 上下文摘要将旅程阶段标记为未知

---

### 用例 4：未提供参数——询问而非报错

**夹具：**
- 无参数
- `design/ux/main-menu.md` 和 `design/accessibility-requirements.md` 均不存在，故两个场景都撰写新文件
- 场景 (a)：用户输入"Main Menu"
- 场景 (b)：用户选择"项目级无障碍需求"

**输入：** `/ux-design`

**预期行为：**
1. 技能不报错；询问"今天设计什么？"并提供："特定界面或流程（我来命名）"、"游戏 HUD"、"交互模式库"、"项目级无障碍需求"、"不确定——帮我确定"
2. (a) 名称规范化为 kebab-case；输出文件为 `design/ux/main-menu.md`
3. (b) 输出文件为 `design/accessibility-requirements.md`，骨架询问指明该路径，而非 `design/ux/` 下的路径
4. (b) 首先撰写 Accessibility Tier Definition 章节
5. 骨架批准前不创建任何文件

**断言：**
- [ ] 无用法错误；询问"今天设计什么？"并提供五个选项
- [ ] (a) 输入的界面名变为 `design/ux/` 下的 kebab-case 文件名
- [ ] (b) 无障碍模式写入 `design/accessibility-requirements.md`
- [ ] (b) 层级定义先于其他无障碍章节撰写
- [ ] 骨架批准前不写入任何文件

---

### 用例 5：无导演门禁——`/ux-review` 是独立审查

**夹具：**
- `project.yaml` 有 `modes.review_mode: full` 和 `platform` 块（`targets: [PC, Console]`、`gamepad_support: Full`）
- 无 `design/ux/settings-menu.md`

**输入：** `/ux-design settings-menu`

**预期行为：**
1. 技能照常撰写界面规范；不解析审查模式，也不据此分支
2. UX 规范骨架的头部带有 `> **Platform Target**:`，由阶段 2h 填充——PC 和 Console、键盘/鼠标和手柄——即 `/ux-review` 检查的行
3. 不生成导演门禁代理，不出现门禁 ID 或跳过说明
4. 若咨询专家（如 `ui-programmer` 关于可行性），其输出呈现给用户，由会话——而非代理——写入文件
5. 运行以 `/ux-review` 声明和结论 `COMPLETE` 结束

**断言：**
- [ ] 骨架头部的 Platform Target 行指明 `platform` 块中的目标和输入方式
- [ ] 不调用导演门禁，不出现门禁跳过消息
- [ ] `modes.review_mode: full` 不改变技能行为
- [ ] 专家输出呈现给用户；代理绝不写入文件
- [ ] 结论为 `COMPLETE`，并指明 `/ux-review` 为验证步骤

---

### 用例 6：交接——指明 Production 门禁及审查要求所依赖的内容

**夹具：**
- `modes.rigor: standard`（固定：交接说明 Production 门禁如何按工作流层级对待 `/ux-review`，`standard` 是推荐层级）
- 界面规范 `design/ux/settings-menu.md`，其章节均已批准并写入；交叉引用检查已运行
- 变体：用户选择"先设计另一界面，然后一起审查所有规范"

**输入：** `/ux-design settings-menu`（继续至交接）

**预期行为：**
1. 阶段 6 更新 `production/session-state/active.md`
2. 提供选项前说明："本规范应在进入实现流水线前用 `/ux-review` 验证。Production 门禁（`/gate-check production`）要求所有关键界面规范在 `workflow: full` 时已通过 `/ux-review`（`standard` 时推荐）。"
3. 然后问"现在运行 `/ux-review [filename]`，还是先做别的？"并提供四个选项（立即运行 / 设计另一界面 / 更新模式库 / 停止）
4. 变体：添加"提醒：在运行 `/gate-check production` 前对所有完成的规范运行 `/ux-review`。"
5. 结尾下一步包括在所有关键界面拥有已批准的 UX 规范后运行 `/gate-check production`
6. 结论：`COMPLETE`

**断言：**
- [ ] 交接指明 `/gate-check production` 为依赖 `/ux-review` 的门禁，而不仅是 `/ux-review` 本身
- [ ] 说明要求为 `workflow: full`，`standard` 时仅为推荐，故 `standard` 项目不会被告知审查是强制的
- [ ] 该声明在选项问题之前
- [ ] 变体——选择"先设计另一界面"时添加指明 `/gate-check production` 的提醒
- [ ] 下一步在界面规范批准后列出 `/gate-check production`
- [ ] 结论为 `COMPLETE`

---

## 协议合规

- [ ] 模式确定后——无参数时模式问题优先（用例 4）——在询问用户其他任何内容前读取上下文
- [ ] 创建骨架前检查现有输出文件；改造仅填充 Empty 或 Placeholder 章节
- [ ] 骨架仅在"我可以在 [path] 创建骨架文件吗？"后创建
- [ ] 章节逐一草拟；每节批准后，在写入前询问"我可以将 [章节名] 章节写入 `[filepath]` 吗？"
- [ ] 加载当前模式的指导文件，绝不加载其他两个；模板指南逐节读取，绝不整篇读取
- [ ] 以 `/ux-review` 交接和结论 `COMPLETE` 结束

---

## 覆盖说明

- 模式库模式（`patterns`）未做夹具测试。
- 当 `project.yaml` 和 `technical-preferences.md` 均未配置平台时的输入方式问题未测试。
- 阶段 5 对模式库中尚不存在的模式的提示未单独测试。
- `guided` 和 `autonomous` 自动化模式、`docs.density` 深度，以及中断会话后的恢复未测试。