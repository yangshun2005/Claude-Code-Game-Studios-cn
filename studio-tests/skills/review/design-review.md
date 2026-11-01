# 技能测试规范：/design-review

## 技能摘要

`/design-review` 依据项目的设计标准（概述、玩家幻想、详细规则、公式、边界情况、依赖、调参旋钮、验收标准）审查一份游戏设计文档（GDD）。哪些章节是必需（REQUIRED）取决于该系统解析出的工作流层级：`full` 层级要求全部 8 个；`standard` 层级要求 5 个（当系统定义了数值规则时再加公式）。它通过 `.claude/scripts/gdd-structure-check.sh` 收集章节存在情况，校验声明的依赖图，检查内部一致性、可实现性和跨系统契合度，并给出 APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED 或 NOT ASSESSED 的结论。

审查模式决定审查深度：`full` 增加并行的对抗性专家审查，外加 `creative-director` 的资深综合（阶段 3b）；`lean` 不启用 agent 运行所有阶段；`solo` 只运行阶段 1–4，不提供下一步提示。阶段 4 不写入任何内容。在阶段 5 中，它可以更新 `design/gdd/systems-index.md`，并向 `design/gdd/reviews/[doc-name]-review-log.md` 追加内容，但每次都必须先经用户通过 `AskUserQuestion` 批准。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 至少有 2 个阶段标题或编号步骤
- [ ] 包含结论关键词：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED、NOT ASSESSED
- [ ] `allowed-tools` 包含 Write/Edit，因此存在"May I"写入批准措辞（systems-index 更新、review-log 追加）
- [ ] 记录了输出格式（技能正文中展示了审查模板）

---

## Fixture 默认值

除非某个用例自身的 Fixture 另有说明，否则每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——完整 GDD，8 个章节齐全

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/light-manipulation.md` 存在，8 个章节均有实质内容（详细规则章节标题为 `## Detailed Design`）
- 公式章节至少定义一个带命名变量的公式
- 验收标准章节至少包含 3 条可测试的标准
- 其依赖章节中提到的每个系统在 `design/gdd/` 中都有 GDD
- 该文档没有先前的审查日志

**输入：** `/design-review design/gdd/light-manipulation.md --review lean`

**预期行为：**
1. 新鲜度检查（`review-receipts.sh check …`）未找到回执 → 完整审查
2. 技能完整读取目标文档并读取 CLAUDE.md
3. 技能运行 `bash .claude/scripts/gdd-structure-check.sh design/gdd/light-manipulation.md`
4. 技能对每个声明的依赖 Glob `design/gdd/`
5. 技能检查内部一致性、可实现性和跨系统契合度
6. 不生成任何 agent（lean）
7. 技能输出阶段 4 审查，以结论结尾
8. 若为 APPROVED，一个多选 `AskUserQuestion` 提供两项跟踪更新；只写入被选中的项；随后是最终的收尾组件

**断言：**
- [ ] 技能在产生任何输出前读取目标文件
- [ ] 章节存在情况来自 `gdd-structure-check.sh`，且 `## Detailed Design` 被接受为详细规则章节（不标记为缺失）
- [ ] 输出包含"Completeness: 8/8"（N 为 8，因为层级是 `full`）
- [ ] 输出包含"Dependency Graph"章节，列出每个声明的依赖及其 GDD 是否存在
- [ ] 输出包含"Required Before Implementation"、"Recommended Revisions"和"Scope Signal"章节，以及"Re-review: No — first review"
- [ ] 当所有必需章节齐全且未发现阻塞问题时，输出以 `### Verdict:` APPROVED 结尾
- [ ] 在跟踪记录组件之前不写入任何文件，且只写入用户选择的选项
- [ ] 若选择了 review-log 选项，追加的条目包含 `Review mode:` 行，以及该文档、技能文件及其读取的相关 GDD 的 `review-receipts.sh hash` 输出

---

### 用例 2：失败路径——不完整 GDD（`full` 层级下 4/8 章节）

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/light-manipulation.md` 仅有概述、玩家幻想、详细规则和依赖；公式、边界情况、调参旋钮和验收标准缺失

**输入：** `/design-review design/gdd/light-manipulation.md --review lean`

**预期行为：**
1. `gdd-structure-check.sh` 在其 `ABSENT:` 列表中报告这 4 个章节
2. 这 4 个在 `full` 层级都是必需的，因此每个都阻塞批准
3. 技能输出"Completeness: 4/8"并列出每个缺失章节
4. 结论为修订类结论，绝不可能是 APPROVED
5. 阶段 5 提供 `[A] Revise the GDD now` / `[B] Stop here`；"Accept as-is"仅在所有项都是建议性时才提供，而此处并非如此

**断言：**
- [ ] 输出在完整性章节显示"4/8"（不是更高的数字）
- [ ] 输出明确将每个缺失章节（公式、边界情况、调参旋钮、验收标准）列为必需
- [ ] 结论为 NEEDS REVISION 或 MAJOR REVISION NEEDED——在必需章节缺失时绝不可能是 APPROVED，也不是 NOT ASSESSED（文档已被读取）
- [ ] 输出不暗示该文档已可实施
- [ ] 不提供"Accept as-is and move on"，因为缺失章节是阻塞性的
- [ ] 除明确的阶段 5 选择外不写入任何文件：`[A] Revise the GDD now`、"May I update `design/gdd/systems-index.md`…"或"May I append this review summary…"
- [ ] systems-index 提示提供 `Needs Revision` 或 `In Review`——在此修订结论下绝不提供 `Approved`——且 review-log 条目记录修订结论，而非批准

---

### 用例 3：部分路径——`standard` 层级，有数值规则但无公式章节

**Fixture：**
- `project.yaml` 设置 `modes.workflow: standard`
- GDD 有概述、玩家幻想、`## Detailed Design`、边界情况、依赖、调参旋钮和验收标准；无公式章节
- 详细设计陈述了数值规则（"每秒消耗 2 点氧气"、"补充花费 50 信用点"）但没有公式定义它们
- 验收标准含糊（"手感好"而非可度量）

**输入：** `/design-review design/gdd/oxygen.md --review lean`

**预期行为：**
1. 技能将公式标记为 `standard` 层级的必需项，因为系统定义了数值规则（仅凭系统的 `Category` 永远不能免除它）
2. N = 6（5 个标准章节 + 公式）；完整性显示 5/6，而非 /8 计数
3. 技能将含糊的验收标准标记为实现性问题
4. 结论为修订类结论

**断言：**
- [ ] 输出将缺失的公式章节标识为必需（非建议性），因为存在数值规则
- [ ] 完整性按 `standard` 计数报告（"5/6"），而非"7/8"
- [ ] 输出将含糊的验收标准标记为实现性缺口，并逐条引用——主审查自行完成此工作，因为 `lean` 下不运行 `qa-lead`
- [ ] 结论为 NEEDS REVISION 或 MAJOR REVISION NEEDED，绝不可能是 APPROVED

---

### 用例 4：边界情况——文件未找到

**Fixture：**
- 提供的路径在项目中不存在

**输入：** `/design-review design/gdd/nonexistent.md --review lean`

（`lean`，因此阶段 5 运行。在 `solo` 下——当 `project.yaml` 既未设置 `modes.review_mode` 也未设置 `modes.rigor` 时审查模式解析为 `solo`——阶段 5 永不运行，下面的最后一条断言将无从测试。）

**预期行为：**
1. 新鲜度检查报告无回执；技能尝试读取文件
2. 文件未找到
3. 技能输出 `### Verdict: NOT ASSESSED`，指出缺失的文档及生成它的技能（`/design-system`）
4. 技能不产生质量结论，也不进行任何跟踪写入
5. 阶段 5 跳过跟踪组件，直接进入最终收尾组件，以 `/design-system` 开头

**断言：**
- [ ] 结论为 NOT ASSESSED——不是 APPROVED、NEEDS REVISION 或 MAJOR REVISION NEEDED
- [ ] 输出指出缺失的文件及由哪个技能生成
- [ ] 阶段 5 运行，且既不提供 `systems-index.md` 状态更新，也不提供 review-log 条目——尤其绝不提供将系统标记为 Approved

---

### 用例 5：审查模式——`full` 下委派专家，`lean` / `solo` 下不委派

**Fixture（所有子用例）：**
- `project.yaml` 设置 `modes.workflow: full`
- `design/gdd/light-manipulation.md` 存在，8 个章节齐全，包括公式和战斗数值

**用例 5a——full：** `/design-review design/gdd/light-manipulation.md --review full`

**预期行为：**
1. 生成前，技能打印"Full review: spawning specialist agents in parallel…"
2. 技能通过真实的 `Agent` 调用并行生成相关专家——此处至少包括 `game-designer`（任何玩法系统）和 `systems-designer`（公式）
3. 专家返回后，技能生成 `creative-director` 作为资深审查者；其综合成为最终结论
4. 发现项带有来源标记；分歧被列出，而非静默解决

**断言（5a）：**
- [ ] 在任何 agent 生成前打印该通知
- [ ] 专家通过 `Agent` 并行生成（非会话内模拟）
- [ ] `creative-director` 仅在专家响应后生成，且输出包含"Senior Verdict [creative-director]"章节
- [ ] "Specialists consulted:"列出实际生成的 agent，且每个发现项都带有来源标记，如 `[game-designer]`
- [ ] 专家分歧出现在"Specialist Disagreements"下
- [ ] `### Verdict:` 行恰好携带本技能词汇之一——APPROVED、NEEDS REVISION 或 MAJOR REVISION NEEDED——取自 creative-director 的综合；绝不使用 READY、CONCERNS、NOT READY、APPROVE 或 REJECT 等 director-gate 词汇
- [ ] 不出现 director-gate ID（"Gate: CD-…"等）——资深审查是直接生成，而非来自 `director-gates.md` 的 gate

**用例 5b——通过旧标志使用 lean：** `/design-review design/gdd/light-manipulation.md --depth lean`

**断言（5b）：**
- [ ] `--depth lean` 被视为 `--review lean`，且技能说明一次该标志已重命名
- [ ] 不进行 `Agent` 调用；审查在会话内运行
- [ ] 阶段 5 的下一步组件仍运行

**用例 5c——solo：** `/design-review design/gdd/light-manipulation.md --review solo`

**断言（5c）：**
- [ ] 不进行 `Agent` 调用
- [ ] 只运行阶段 1–4：无阶段 5 下一步提示，因此不提供 systems-index 或 review-log 写入

---

### 用例 6：审查模式来自解析后的配置——无标志

**Fixture：**
- `project.yaml` 设置 `modes.workflow: full` 和 `modes.review_mode: lean`
- `design/gdd/light-manipulation.md` 存在，8 个章节齐全

**输入：** `/design-review design/gdd/light-manipulation.md`

**预期行为：**
1. 无 `--review` 标志，因此技能从技能顶部的解析配置块取 `review_mode: lean`
2. 审查在会话内运行，无 `Agent` 调用
3. 阶段 5 的组件运行——而 `solo`（`rigor: minimal` 默认值）本会跳过它们

**断言：**
- [ ] 未给标志时，审查模式来自解析配置块（`review_mode: lean (project.yaml)`）
- [ ] 不进行 `Agent` 调用，且阶段 5 的下一步组件运行——因此本次运行为 `lean`，而非 `solo` 默认值
- [ ] 变体——相同 fixture 但输入带 `--review solo`：标志优先于配置，且不运行任何阶段 5 组件

---

## 协议合规

- [ ] 审查本身（阶段 4）期间不写入任何内容
- [ ] 每次写入（systems-index 状态、review-log 追加）都仅在 `AskUserQuestion` 批准后进行
- [ ] 在结论行之前呈现完整发现项
- [ ] 除 `solo` 外，以 `AskUserQuestion` 下一步组件收尾，而非纯文本（collaborative/guided 模式）

---

## 覆盖说明

- 跨多个 GDD 的跨系统一致性由 `/review-all-gdds` 规范覆盖；本规范只检查单个文档的依赖引用。
- 新鲜度回执路径（字节完全相同的重审提供先前结论；请求更深审查时提供较浅的已记录审查；仅跨文档变更）未在此做 fixture 测试。
- 实体注册表 grep 路径（注册表列出此系统）未测试；上述 fixture 使用读取依赖中命名的 GDD 的回退方案。
- `minimal` 层级（不期望 GDD；标准章节以建议性方式检查）未测试。
- 涉及超大 GDD 文件的性能和边界情况不在范围内。