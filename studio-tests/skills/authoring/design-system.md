# 技能测试规范：/design-system

## 技能摘要

`/design-system <system-name>` 为一个系统编写游戏设计文档，路径为 `design/gdd/[system-name].md`。所需章节取决于解析出的 `workflow` 层级（该系统的 `system_overrides` 行优先）：`full` 层级需要全部 8 个章节（概述、玩家幻想、详细设计、公式、边界情况、依赖、调参旋钮、验收标准）；`standard` 层级需要 5 个章节加条件性公式；自愿选择的 `minimal` 层级需要 5 个章节。`## Summary` 在每个层级都会编写，位于其他章节之后。

五个阶段各对应一个 `references/N-*.md` 文件，技能在执行该阶段前读取，若在该阶段期间对话被压缩则再次读取。

技能先读取上下文（游戏概念、系统索引、实体注册表、依赖 GDD 章节、引擎参考），展示上下文摘要和技术可行性简报，然后询问"我可以在 `design/gdd/[system-name].md` 创建骨架文件吗？"。随后每个章节运行循环：上下文 → 提问 → 选项 → 决策 → 草稿 → 批准 → 写入：在 `collaborative` 模式下，草稿和"批准[章节名称]章节？"组件出现在同一响应中，批准的章节立即通过 Edit 写入。每个章节会咨询专家 agent，受各章节审查模式检查约束。

所有章节写入后，CD-GDD-ALIGN 门仅在 `full` 模式下运行一次。`retrofit <path>` 仅填充现有 GDD 中缺失或占位的章节。技能从不内联提供 `/design-review`——它引导用户到新会话。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含裁决关键词：`APPROVED`、`CONCERNS`、`BLOCKED`
- [ ] 包含"May I write" / "May I create"协作协议语言（骨架和逐章节批准）
- [ ] 末尾有下一步交接
- [ ] 记录骨架优先方法（批准后先创建带标题的文件，再填充内容）
- [ ] 记录 CD-GDD-ALIGN 门：仅在 full 模式运行；在 lean 和 solo 模式下跳过并附具名说明
- [ ] 记录针对现有 GDD 文件的 retrofit 模式

---

## 总监门检查

审查模式来自解析出的 `review_mode`（`project.yaml` 中的 `modes.review_mode`）；`--review full|lean|solo` 可覆盖单次运行。

**CD-GDD-ALIGN**（`creative-director`）在步骤 5a-bis 运行一次，位于所有章节和 Summary 写入后、自检从文件读回 GDD 之后。传入 GDD 路径、游戏支柱、MDA 美学目标和 GDD 的玩家幻想章节（或声明该层级未编写此章节）。裁决记录在 GDD Status 头部：`> **Creative Director Review (CD-GDD-ALIGN)**: APPROVED [date] / CONCERNS (accepted) [date] / REVISED [date] / NOT ASSESSED [date] — [missing input]`。

- `full` → 启动。
- `lean` → 跳过（非 PHASE-GATE）。说明："CD-GDD-ALIGN skipped — Lean mode."
- `solo` → 跳过。说明："CD-GDD-ALIGN skipped — Solo mode."

逐章节专家启动（非总监门）有各自的检查：`solo` 全部跳过；`lean` 跳过它们，但 D 章节（公式）和 H 章节（验收标准）、当 D 章节定义了旋钮交互的公式时的 G 章节（调参旋钮）、以及当系统的视觉反馈对其核心时（玩家通过其事件进行游玩的 `Gameplay` 或 `UI` 系统）的视觉/音频除外；`full` 启动它们。每次跳过的启动都会打印该章节的说明，指明 agent 和模式，例如"`creative-director` not consulted — Lean mode. Review manually before production."

---

## Fixture 默认值

除非其自身 Fixture 另有说明，每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——新战斗 GDD，full 模式，full 工作流

**Fixture：**
- `design/gdd/game-concept.md` 和 `design/gdd/systems-index.md` 存在；`combat` 列出，`Category: Gameplay`
- 无 `design/gdd/combat.md`
- `project.yaml` 有 `engine.name: Godot`、`modes.review_mode: full`、`modes.workflow: full`、`modes.automation: collaborative`；`docs/engine-reference/godot/` 有 `VERSION.md` 和 `modules/physics.md`（combat 映射到 Physics 域）
- CD-GDD-ALIGN 返回 APPROVE

**输入：** `/design-system combat`

**预期行为：**
1. 阶段 2 在提问前读取概念、系统索引、实体注册表（如存在）和依赖 GDD 章节，然后展示"Designing: Combat"上下文摘要和技术可行性简报及其 `AskUserQuestion`
2. 询问"Ready to start designing combat?"
3. 询问"May I create the skeleton file at `design/gdd/combat.md`?"——确认后，骨架写入全部 8 个必需章节及模板的其他章节，每个章节含 `[To be designed]`；`production/session-state/active.md` 更新
4. 按顺序遍历 A–H 章节；每份草稿在同一响应中后接"Approve the [Section Name] section?"，含 `[A] Approve — write it to file` / `[B] Make changes — describe what to fix` / `[C] Start over`
5. B 章节咨询 `creative-director`；C 章节并行启动 Combat 行的 `game-designer` 及支持 agent；D 章节启动 `systems-designer`；E 章节 `systems-designer`；H 章节 `qa-lead`
6. 每个批准的章节立即通过 Edit 写入，其 `old_string` 包含章节标题；每次后更新 `active.md`
7. 视觉/音频和游戏手感视为必需（Gameplay 类别）——不作为可跳过项提供
8. `## Summary` 在其他章节后编写；步骤 5a 从文件读回 GDD
9. CD-GDD-ALIGN 启动一次并返回 APPROVE；Status 头部记录 `APPROVED [date]`
10. 完成摘要引导用户在新会话中运行 `/design-review design/gdd/combat.md`

**断言：**
- [ ] 骨架仅在"May I create the skeleton file"批准后写入，且包含全部 8 个必需章节标题
- [ ] 每份章节草稿在同一响应中后接"Approve the [Section Name] section?"组件
- [ ] 每个章节在其批准后立即单独写入，通过锚定其标题的 Edit
- [ ] C 章节的专家来自路由表的 Combat 行，并在起草前并行启动
- [ ] CD-GDD-ALIGN 启动一次，位于所有章节和 Summary 写入后——而非逐章节
- [ ] 裁决记录为 `> **Creative Director Review (CD-GDD-ALIGN)**: APPROVED [date]`
- [ ] `/design-review` 引导至新会话，从不内联提供

---

### 用例 2：Retrofit 模式——在 standard 层级仅填充不完整章节

**Fixture：**
- `project.yaml` 有 `engine.name: Godot`、`modes.workflow: standard`、`modes.review_mode: lean`
- `design/gdd/game-concept.md` 和 `design/gdd/systems-index.md` 存在；`inventory` 列出，`Category: Economy`（引擎域 Scripting）
- `docs/engine-reference/godot/` 有 `VERSION.md` 和 `modules/`，但无 `modules/scripting.md`
- `design/gdd/inventory.md` 存在，概述、详细设计和公式已完整编写；其 `## Summary` 仅含 `[To be designed]`
- `## Edge Cases` 和 `## Acceptance Criteria` 仅含 `[To be designed]`；`## Dependencies` 正文为空
- 文件无玩家幻想或调参旋钮章节

**输入：** `/design-system retrofit design/gdd/inventory.md`

**预期行为：**
1. 技能读取现有 GDD，识别哪些章节存在、仅占位或为空
2. 展示"Retrofit: Inventory"块：已编写章节（不会触碰）和缺失或不完整章节（将编写）
3. 玩家幻想和调参旋钮列为可添加项，而非缺口，因为 `standard` 不要求它们
4. 询问："Shall I fill the 3 missing sections? I will not modify any existing content."
5. 确认后，阶段 2 正常运行；§2e 指明缺失模块——"no engine reference for `Scripting` under `docs/engine-reference/godot/modules/`; feasibility not checked against the pinned engine"——并带入 §5-pre
6. 不创建骨架；仅边界情况、依赖和验收标准运行章节循环，然后 §5-pre 替换 Summary 占位
7. 每次写入仅替换占位或空正文；概述、详细设计和公式不被修改

**断言：**
- [ ] retrofit 块在任何更改前展示
- [ ] 该层级不要求的章节不报告为缺口
- [ ] 技能询问"Shall I fill the 3 missing sections? I will not modify any existing content."
- [ ] 缺失的 `scripting.md` 模块在输出中指明，而非静默跳过如同已检查可行性
- [ ] 不创建骨架，完整章节不被重写或覆盖
- [ ] 每个填充章节在写入前仍经过逐章节批准

---

### 用例 3：总监门——CD-GDD-ALIGN 返回 CONCERNS、REJECT 或 NOT ASSESSED

**Fixture：**
- `design/gdd/game-concept.md`（支柱含"Fearless Exploration"）和 `design/gdd/systems-index.md` 存在；`stamina` 列出，`Category: Gameplay`
- `project.yaml` 有 `modes.review_mode: full` 和 `modes.workflow: full`
- `design/gdd/stamina.md` 的全部八个章节（含玩家幻想）和 Summary 已编写
- 场景 (a)：CD-GDD-ALIGN 返回 CONCERNS："Detailed Design's stamina drain undercuts the 'Fearless Exploration' pillar"
- 场景 (b)：CD-GDD-ALIGN 返回 REJECT："stamina exhaustion strands the player mid-exploration, which the 'Fearless Exploration' pillar rules out"
- 场景 (c)：CD-GDD-ALIGN 返回 NOT ASSESSED——无 MDA 美学目标可用于对照检查 GDD

**输入：** `/design-system stamina`（到达步骤 5a-bis）

**预期行为：**
1. 启动 `creative-director`，门为 CD-GDD-ALIGN，传入 GDD 路径、游戏支柱、MDA 美学目标和 GDD 的玩家幻想章节
2. (a) CONCERNS 通过 `AskUserQuestion` 展示，含标准选项：`Revise flagged items` / `Accept and proceed` / `Discuss further`
3. (a) 若"Revise flagged items"：详细设计重新运行其章节循环——先咨询其专家——并通过"Approve the Detailed Design section?"批准后才写入；Status 头部记录 `REVISED [date]`
4. (a) 若"Accept and proceed"：Status 头部记录 `CONCERNS (accepted) [date]`
5. (b) 展示阻塞项；被标记章节以同样方式重新起草并重新批准后才写入其他内容，Status 头部记录 `REVISED [date]`，绝不记录 `APPROVED`
6. (c) 指明缺失输入；用户可提供并重新运行门，或 Status 头部记录 `NOT ASSESSED [date] — [missing input]`，绝不记录 `APPROVED`
7. 仅在裁决记录后，技能才继续到实体注册表步骤（5b）

**断言：**
- [ ] CD-GDD-ALIGN 接收完成的 GDD 路径、支柱、MDA 目标和玩家幻想章节
- [ ] (a) CONCERNS 向用户展示三个标准选项，不自动接受
- [ ] 修订章节重新运行其章节循环并在写入前重新批准
- [ ] Status 头部记录 `REVISED [date]` 或 `CONCERNS (accepted) [date]` 以匹配用户选择
- [ ] (b) REJECT 后，在被标记章节修订前不运行任何后续步骤（5b 注册表、5d 系统索引）
- [ ] (c) NOT ASSESSED 绝不记录或报告为批准

---

### 用例 4：Lean 和 Solo 模式——门跳过一次；专家启动遵循各自检查

**Fixture：**
- `design/gdd/game-concept.md` 和 `design/gdd/systems-index.md` 存在；系统为 `stamina`（`Category: Gameplay`）；无 `design/gdd/stamina.md`
- `project.yaml` 有 `engine.name: Godot` 和 `modes.workflow: full`
- D 章节的 stamina 公式无交互旋钮；stamina 条是玩家游玩所读的反馈
- 场景 (a)：`modes.review_mode: lean`
- 场景 (b)：`modes.review_mode: solo`

**输入：** `/design-system stamina`

**预期行为：**
1. (a) B、C、E 和 G 章节无专家启动起草，每个打印其说明并含模式，例如"`creative-director` not consulted — Lean mode. Review manually before production."
2. (a) D 章节仍启动 `systems-designer`，H 章节 `qa-lead`，视觉/音频 `art-director`——stamina 的反馈对游玩核心
3. (a) 在步骤 5a-bis："CD-GDD-ALIGN skipped — Lean mode."——打印一次，非逐章节
4. (b) 不启动任何专家或总监 agent；每个跳过启动的章节携带其说明，例如"`creative-director` not consulted — Solo mode. Review manually before production."
5. (b) 在步骤 5a-bis："CD-GDD-ALIGN skipped — Solo mode."
6. 两个场景中每个章节在写入前仍需"Approve the [Section Name] section?"批准

**断言：**
- [ ] (a) lean 模式下 B、C、E 和 G 章节无专家起草，而 D、H 章节和视觉/音频仍启动其专家
- [ ] (a) 每个 lean 跳过的启动打印"not consulted — Lean mode"说明并指明其 agent——无一静默跳过
- [ ] (a) CD-GDD-ALIGN 跳过说明在步骤 5a-bis 出现一次
- [ ] (b) solo 模式下不启动任何 agent，每个跳过的专家留下"not consulted — Solo mode"说明
- [ ] (b) 跳过说明为"CD-GDD-ALIGN skipped — Solo mode."
- [ ] 两种模式均需逐章节用户批准

---

### 用例 5：缺失输入和拒绝骨架

**Fixture：**
- 每个场景中 `project.yaml` 有 `modes.workflow: standard`——该层级要求系统索引，因此指向 `/map-systems` 是正确的
- 场景 (a)：无参数；`design/gdd/systems-index.md` 将 `stamina` 列为最高优先级"Not Started"系统（MVP，Core 层）
- 场景 (b)：无参数；无系统索引
- 场景 (c)：`/design-system stamina`，概念和索引存在；用户拒绝骨架

**输入：** (a) `/design-system` (b) `/design-system` (c) `/design-system stamina`

**预期行为：**
1. (a) `AskUserQuestion`："The next system in your design order is **stamina** (MVP | Core). Start designing it?"，含 `[A] Yes — design stamina` / `[B] Pick a different system` / `[C] Stop here`
2. (b) 技能停止，显示用法消息（"Usage: `/design-system <system-name>` …"）、retrofit 示例和"No systems index found. Run `/map-systems` first to map your systems and get the design order."
3. (c) 技能停止并显示："Verdict: **BLOCKED** — skeleton creation declined. …"，不进入 A 章节
4. 三个场景中均不在批准前写入 GDD 文件

**断言：**
- [ ] (a) 从系统索引提议下一个系统，含三个选项
- [ ] (b) 打印用法消息和 `/map-systems` 指引；不写入任何内容
- [ ] (c) 拒绝骨架产生 BLOCKED 裁决并在 A 章节前停止
- [ ] 无批准不创建 GDD 文件

---

### 用例 6：Minimal 项目，一个系统由 `system_overrides` 提升

**Fixture：**
- `project.yaml` 有 `modes.workflow: minimal`、`modes.review_mode: solo` 和 `workflow_overrides.system_overrides.combat: standard`
- `design/game-brief.md` 存在；其 MVP 功能描述战斗（伤害规则和硬直阈值），构建顺序将战斗列为首位
- 无 `design/gdd/game-concept.md`，无 `design/gdd/systems-index.md`，无 `design/gdd/combat.md`

**输入：