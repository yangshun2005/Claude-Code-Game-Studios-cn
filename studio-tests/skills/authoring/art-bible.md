# 技能测试规范：/art-bible

## 技能摘要

`/art-bible` 在 `design/art/art-bible.md` 编写美术圣经，共九个部分：1 视觉识别声明、2 氛围与基调、3 形状语言、4 色彩系统、5 角色设计方向、6 环境设计语言、7 UI/HUD 视觉方向、8 资产标准、9 参考方向。

它首先读取 `design/gdd/game-concept.md`（或在 `rigor: minimal` 下读取 `design/game-brief.md`），两者都不存在则停止。一个双标签页的 `AskUserQuestion` 框定本次会话：**范围**（将解析出的 `workflow` 层级所要求的选项标记为推荐）和**参考**（自由文本）。编写是渐进式的：每个部分的草稿都来自生成的专家（`art-director`；第 7 部分另加 `ux-designer`，第 8 部分另加 `technical-artist`），呈现给用户、获得批准，并在下一部分开始前立即写入文件。第 2–4 部分共用一次 `art-director` 调用，第 5–6 部分同理，但它们的批准和写入仍逐部分进行。首次写入会询问"我可以根据美术圣经模板创建 `design/art/art-bible.md` 吗？"；每个部分只有在批准选项明确指明写入（`[A] 锁定并写入 design/art/art-bible.md`）之后才会写入。只编写所选范围内的部分，结束时说明遗漏了哪些部分及原因。

如果文件已存在，技能通过两次 Grep（标题和占位符标记）构建部分状态表，而不读取文档，并且只编写空或占位符部分。

在范围内部分写入后，阶段 5 仅在 `full` 审查模式下运行 AD-ART-BIBLE 签核，并将结论记录在美术圣经的状态头中。阶段 6 以按项目状态过滤的下一步 `AskUserQuestion` 结束。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含记录在状态头中的门禁结论关键词：`APPROVED`、`CONCERNS`
- [ ] 包含写入前询问的措辞：文件存在前询问"我可以根据美术圣经模板创建 `design/art/art-bible.md` 吗？"，以及每个部分指明写入的批准选项（`锁定并写入 design/art/art-bible.md`）
- [ ] 记录 AD-ART-BIBLE 主管门禁及其模式行为
- [ ] 具有下一步交接（例如 `/map-systems`、`/create-architecture`）

---

## 主管门禁检查

| 门禁 ID      | 触发条件                                        | 模式守卫 |
|--------------|----------------------------------------------------------|------------|
| AD-ART-BIBLE | 阶段 5——阶段 1 范围内的所有部分写入之后 | 仅 full；lean 注明"AD-ART-BIBLE 已跳过——Lean 模式。"，solo 注明"AD-ART-BIBLE 已跳过——Solo 模式。" |

门禁以其定义所指定的代理生成（`.claude/docs/director-gates/ad-art-bible.md`：`art-director`），并传入该定义列出的四项：美术圣经路径、游戏支柱与核心幻想、平台与性能约束（`platform.*` 和 `performance.*`），以及视觉识别锚点。从一页简报中，它传入简报的推介语和"他们的感受"行作为支柱与幻想，传入其"美术与音频方向"行作为锚点。

向 `art-director`（以及 `ux-designer` / `technical-artist`）委托部分草稿不是门禁：它是强制性的，在每种审查模式下都会运行。

---

## 夹具默认值

除非自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——Full 模式、full 工作流、全部九个部分、AD-ART-BIBLE 批准

**夹具：**
- 无 `design/art/art-bible.md`
- `design/gdd/game-concept.md` 存在，包含支柱和视觉识别锚点部分
- `project.yaml` 具有 `modes.review_mode: full` 和 `modes.workflow: full`
- AD-ART-BIBLE 返回 APPROVE

**输入：** `/art-bible`

**预期行为：**
1. 阶段 0 读取游戏概念并提取标题、推介语、支柱、视觉识别锚点和平台
2. 阶段 1 询问一个 `AskUserQuestion`，包含范围标签页（完整圣经标记为推荐，因为 `full` 要求全部 9 个部分）和自由文本参考标签页，并注明它找到的锚点
3. 第 1 部分：`art-director` 起草；用户从 `[A] 锁定并写入 design/art/art-bible.md` / `[B] 修改一句话陈述` / `[C] 修改一条支撑原则` / `[D] 描述我自己的方向` 中选择；在"我可以根据美术圣经模板创建 `design/art/art-bible.md` 吗？"之后，从 `.claude/docs/templates/art-bible.md` 创建文件，批准的部分替换第 1 部分的 `[To be designed]` 行
4. 第 2–4 部分：一次 `art-director` 调用返回三个带标签的块；第 2 部分被呈现、以指明写入的选项批准并写入，然后才呈现第 3 部分，依此类推
5. 第 5–6 部分：一次 `art-director` 调用；逐部分呈现、批准并写入
6. 第 7 部分：`art-director` 和 `ux-designer` 并行生成；第 8 部分：`art-director` 和 `technical-artist` 并行生成；第 9 部分：`art-director`
7. 阶段 5：AD-ART-BIBLE 在第九部分写入后生成，传入美术圣经路径、支柱与核心幻想、平台与性能约束，以及视觉识别锚点
8. 状态头记录 `> **Art Director Sign-Off (AD-ART-BIBLE)**: APPROVED [date]`
9. 阶段 6：下一步的 `AskUserQuestion` 始终包含 `/create-architecture` 和"在此停止"

**断言：**
- [ ] 范围标签页将"完整圣经——全部 9 个部分"标记为推荐并说明原因
- [ ] 每个部分的内容都来自生成的专家，绝不来自编排器自身的起草
- [ ] 第 2–4 部分来自单次 `art-director` 调用，但逐部分呈现、批准并写入
- [ ] 在任何指明写入的批准（`锁定并写入 design/art/art-bible.md`）之前不写入任何部分，且每个部分在批准后立即写入
- [ ] 文件仅在"我可以根据美术圣经模板创建 `design/art/art-bible.md` 吗？"——一个指明文件的询问——之后从 `.claude/docs/templates/art-bible.md` 创建，因此其九个 `## N. Name` 标题正是改造模式（用例 4）所读取的
- [ ] AD-ART-BIBLE 仅在全部九个部分写入后生成，作为 `art-director`（其门禁定义所指定的代理）
- [ ] AD-ART-BIBLE 被传入全部四项上下文：美术圣经路径、支柱与核心幻想、平台与性能约束，以及视觉识别锚点
- [ ] APPROVE 结论记录在美术圣经的状态头行中
- [ ] 结束选项包含 `/create-architecture` 和"在此停止"

---

### 用例 2：AD-ART-BIBLE 返回 CONCERNS 或 NOT ASSESSED

**夹具：**
- 全部九个部分已写入 `design/art/art-bible.md`
- `project.yaml` 具有 `modes.review_mode: full` 和 `modes.workflow: full`
- 场景 (a)：AD-ART-BIBLE 返回 CONCERNS："色彩系统与战斗的氛围与基调目标不匹配"
- 场景 (b)：AD-ART-BIBLE 返回 NOT ASSESSED——没有可用的平台或性能约束来对照检查资产标准

**输入：** `/art-bible`（到达阶段 5）

**预期行为：**
1. (a) CONCERNS 通过 `AskUserQuestion` 呈现给用户，使用 `director-gates.md` 中的标准选项：`修改标记项` / `接受并继续` / `进一步讨论`
2. (a) 如果选择"接受并继续"：状态头记录 `CONCERNS (accepted) [date]`
3. (a) 如果选择"修改标记项"：`art-director`——色彩系统自身的专家——重新起草第 4 部分；呈现给用户并以指明写入的选项批准后才写入；状态头随后记录 `REVISED [date]`
4. (b) 指明缺失的输入，状态头记录 `NOT ASSESSED [date] — [missing input]`，绝不记录 `APPROVED`
5. 阶段 6 仅在结论记录后运行

**断言：**
- [ ] (a) CONCERNS 以三个标准选项呈现给用户，而非自动接受
- [ ] (a) "接受并继续"在状态头记录 `CONCERNS (accepted) [date]`
- [ ] (a) 标记的部分由其专家重新起草，而非编排器，并在写入前获得批准；头部记录 `REVISED [date]`
- [ ] (b) NOT ASSESSED 如实记录，指明缺失的输入——而非作为批准
- [ ] 在结论记录前不呈现阶段 6 的下一步

---

### 用例 3：Lean 模式、Standard 工作流——仅核心部分，门禁跳过

**夹具：**
- 无现有美术圣经
- `design/gdd/game-concept.md` 存在
- `project.yaml` 具有 `modes.review_mode: lean` 和 `modes.workflow: standard`
- 用户选择推荐范围

**输入：** `/art-bible`

**预期行为：**
1. 范围标签页将"视觉识别核心（仅第 1–4 部分）"标记为推荐并说明"第 1–4 部分是 `standard` 所要求的；如果需要，第 5–9 部分也可用。"
2. 第 1–4 部分由 `art-director` 起草，每个部分在下一个之前批准并写入
3. 阶段 3 和 4（第 5–9 部分）不运行，因为它们在所选范围之外
4. 阶段 5 不生成 AD-ART-BIBLE 并打印"AD-ART-BIBLE 已跳过——Lean 模式。"
5. 阶段 6 首先说明第 5–9 部分本次未编写（在所选范围之外），然后呈现下一步

**断言：**
- [ ] 范围推荐为第 1–4 部分，并附有 `standard` 说明
- [ ] 仅编写第 1–4 部分
- [ ] 结束时说明第 5–9 部分未编写及原因，使四部分的美术圣经不会被读作完整
- [ ] 在 lean 模式下不生成 AD-ART-BIBLE，跳过说明为"AD-ART-BIBLE 已跳过——Lean 模式。"
- [ ] 每次写入前仍需要逐部分用户批准

---

### 用例 4：现有美术圣经——仅改造不完整部分

**夹具：**
- `design/art/art-bible.md` 存在，第 1–4 部分已完整编写
- 第 5 部分的标题后跟 `[To be designed]`
- 第 6–9 部分有标题但之间无内容
- `design/gdd/game-concept.md` 存在；`project.yaml` 具有 `modes.workflow: full` 和 `modes.review_mode: full`
- 在范围标签页用户选择 `继续——填充缺失部分`
- AD-ART-BIBLE 返回 APPROVE

**输入：** `/art-bible`

**预期行为：**
1. 阶段 0 用 Glob 查找 `design/art/art-bible.md` 并找到它
2. 技能运行两次 Grep——`^## ` 标题和占位符标记（`[To be designed]`、`[TBD]`、`[To be written]`、`TODO`）——不读取整个文档；有歧义的部分仅按行范围读取
3. 它构建状态表（第 1–4 部分完整，第 5 部分占位符，第 6–9 部分空）并说明："在 `design/art/art-bible.md` 找到现有美术圣经。4 个部分完整，5 个需要内容。我将只处理不完整部分——现有内容不会被改动。"
4. 仅编写、批准并写入第 5–9 部分；第 1–4 部分不重新编写或重写
5. 第九部分写入后，阶段 5 生成 AD-ART-BIBLE——在 `full` 审查模式下改造仍以签核结束——状态头记录 `APPROVED [date]`
6. 阶段 6 说明第 1–4 部分本次未编写（已完整）

**断言：**
- [ ] 部分状态来自两次 Grep，而非完整读取文件
- [ ] 改造消息说明完整和不完整的数量，以及现有内容不会被改动
- [ ] 仅编写空和占位符部分；完整部分保持不变
- [ ] 状态表行使用技能编写的部分名称（例如 `Mood & Atmosphere`、`Shape Language`、`Reference Direction`）
- [ ] AD-ART-BIBLE 在改造部分写入后运行，如同全新运行之后一样

---

### 用例 5：Solo 模式和缺失概念

**夹具：**
- 场景 (a)：无 `design/gdd/game-concept.md`；`design/game-brief.md` 存在；`project.yaml` 具有 `modes.review_mode: solo` 和 `modes.workflow: minimal`
- 场景 (b)：`design/gdd/game-concept.md` 和 `design/game-brief.md` 都不存在

**输入：** `/art-bible`

**预期行为：**
1. (a) 阶段 0 读取一页简报（工作标题、推介语、"他们的感受"行、"美术与音频方向"行）代替概念
2. (a) 在范围问题之前，技能说明在 `minimal` 下不需要美术圣经，并提供第 1–4 部分而非默认全部 9 个
3. (a) 仍生成 `art-director` 起草每个编写的部分——solo 审查模式跳过签核，而非编写委托
4. (a) 阶段 5 打印"AD-ART-BIBLE 已跳过——Solo 模式。"且不生成门禁代理；对"我可以将跳过的签核记录在 `design/art/art-bible.md` 的头部吗？"回答是时，状态头记录 `> **Art Director Sign-Off (AD-ART-BIBLE)**: SKIPPED [date] — solo mode`
5. (a) 阶段 6 在 `workflow: minimal` 下提供 `/create-stories`、`/dev-story [next story]` 和在此停止——而非 `/create-architecture` 或 GDD/`/map-systems` 池
6. (b) 技能停止并显示："未找到游戏概念。请先运行 `/brainstorm`——美术圣经在游戏概念批准后编写。"不创建文件，不生成代理

**断言：**
- [ ] (a) 游戏概念缺失时使用游戏简报
- [ ] (a) 技能在询问范围问题前说明在 `minimal` 下不需要美术圣经
- [ ] (a) `art-director` 在 solo 模式下仍起草各部分
- [ ] (a) 不生成 AD-ART-BIBLE；跳过说明为"AD-ART-BIBLE 已跳过——Solo 模式。"，头部记录 SKIPPED 签核行，含模式和日期
- [ ] (a) 阶段 6 选项池为 `workflow: minimal` 集合，而非 `standard`/`full` GDD 池
- [ ] (b) 技能以 `/brainstorm` 消息停止，不写入任何内容

---

## 协议合规

- [ ] 编写前读取游戏概念（或游戏简报）；两者都不存在时以 `/brainstorm` 消息停止
- [ ] 每个部分由专家代理起草；编排器绝不自行起草内容
- [ ] 每个部分逐部分呈现、批准并写入——批准和写入绝不批量进行
- [ ] 文件仅在"我可以创建 `design/art/art-bible.md`……"询问之后创建，每个部分仅在指明写入的批准选项之后写入
- [ ] 仅编写所选范围内的部分，结束时说明遗漏的每个部分及原因
- [ ] `art-director` 与 `ux-designer` 或 `technical-artist` 之间的冲突呈现给用户，绝不静默解决
- [ ] AD-ART-BIBLE 仅在 full 模式下运行，在范围内部分写入之后；lean 和 solo 打印指明门禁和模式的跳过说明
- [ ] 以按项目状态过滤的下一步 `AskUserQuestion` 结束——在 `standard`/`full` 下始终包含 `/create-architecture` 和"在此停止"；在 `workflow: minimal` 下，`/create-stories` 或 `/dev-story [next story]` 加在此停止

---

## 覆盖说明

- AD-ART-BIBLE 的 REJECT 结论未单独测试；根据标准结论规则，技能呈现阻塞项并在其解决前不推进